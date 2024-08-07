# My keyboard was misbehaving so I had to exploit my NAS


I recently received my [ZimaCube](https://www.zimaspace.com/products/cube-personal-cloud): a NAS from [IceWhale](https://github.com/IceWhaleTech), the same company behind the [ZimaBlade](https://www.zimaspace.com/products/blade-personal-nas), [ZimaBoard](https://www.zimaspace.com/products/single-board-server) and most notably [CasaOS](https://casaos.io/), a UI to manage docker applications.

The ZimaCube ships with the default OS called `ZimaOS`.

{{< admonition type=note title="Note" open=true >}}
This part is a bit tricky but stay with me because I need to clarify this point, otherwise the rest of the article could be hard to understand:

IceWhale decided to put the word `OS` into `CasaOS` even if it's not an operating system and then release their NAS, `ZimaCube`, with something called `ZimaOS` that _is_ a Linux distro for their NAS that also contains a custom version of `CasaOS`.

After all, naming things is [one of the hardest things](https://imgs.xkcd.com/comics/permanence.png).
{{< /admonition>}}

Upon booting it up connected to an external monitor I was welcomed with the following TTY message:

{{< figure src="press_alt_f2.jpeg" title="Press alt+F2 to proceed" >}}

Cool, I need to press al `alt+F2`, there is just one small, insignificant problem...

## The problem
All my keyboards are <= 60% so the F row is activated by the `Fn` special key, the problem is that for some reasons I'm not aware of connecting it to the NAS will not work so no luck.

Let's analyze the possible alternatives:

![mmm](mmm.png)

- Let's exclude `B` because is something that no one will do, [right](https://www.forbes.com/sites/daveywinder/2022/02/05/one-american-hacker-suddenly-takes-down-north-koreas-internet-all-of-it/)? [RIGHT](https://reddit.com/r/IAmA/comments/1divlp3/im_the_hacker_that_brought_down_north_koreas/)?

- `A` seems the easiest one but it will not get delivered until Monday and it's [way too hot outside these days](https://www.msn.com/en-gb/weather/topstories/heatwave-inferno-record-temperatures-sweep-across-italy-and-the-balkans/ar-BB1oDko8).
And I also don't want to change a working keyboard.

- `D` is kinda reasonable too, but the OS is on an M2 Drive and I have no adapters.

So I don't see any other choice of getting my hands dirty and find some good, old unintended solutions.

{{< admonition type=note title="Note" open=false >}}
There were at least 10 different solutions that were faster and cleaner than this one (like patching the OS image file or getting an ESP32 to inject the keystrokes) but it was my first free Saturday in a while and I really love to overcomplicate stuff.
{{< /admonition >}}

## Overcomplicating the issue for fun (and boredom)
In the same screen where I'm asked to press `alt+F2` there is a reference to the _ZimaOS WebUI_ so I decided to start investigating it.

{{< figure src="homepage.png" title="ZimaOS Homepage" >}}


After loading the homepage I saw something similar to CasaOS interface, but in some way different (as I said, they ship a custom version). 

While ZimaOS was in beta I found a code injection in  the Storage Management API that is still unpatched but unfortunately I need an additional mounted drive to exploit it.

CasaOS is opensource but any component in ZimaOS isn't, so I have no idea on the exact differences between the two interfaces.	

{{< admonition type=note title="TL;DR" open=false >}}
Long story short: the `unmount` API is vulnerable to code injection but if no mount point is detected the entrypoint will not be reached.

More on [the advisory](https://github.com/IceWhaleTech/CasaOS/security/advisories/GHSA-cjq9-c4gf-jfr2)
{{< /admonition >}}

The [code injection](https://github.com/IceWhaleTech/CasaOS-LocalStorage/blob/ca6d672f152920dca851c24a941136747ccc2ec2/pkg/utils/command/command_helper.go#L62-L69) exists because they are concatenating arbitrary user input into a `cmd.Exec` function:

```go
func ExecLSBLKByPath(path string) []byte {
    // the path parameter is sent by the user
	output, err := exec.Command("lsblk", path, "-O", "-J", "-b").Output() 
	...
}
```


Before merging the disk (and so calling `lsblk`), the [existence for the mountpoint is checked](https://github.com/IceWhaleTech/CasaOS-LocalStorage/blob/ca6d672f152920dca851c24a941136747ccc2ec2/service/v2/merge.go#L209-L216):

```go
func excludeVolumesWithWrongMountPointAndUUID(volumes []*model2.Volume) []*model2.Volume {
	return filterVolumes(volumes, func(v *model2.Volume) bool {
        // This function checks if the mountpoint exists
        // If it returns err the vulnerable call is not reached
		path, err := partition.GetDevicePath(v.UUID)
		if err != nil {
			logger.Error("failed to corresponding device path by volume UUID", zap.Error(err), zap.String("uuid", v.UUID))
			return false
		}
    // Call the vulerable function
    par := command.ExecLSBLKByPath(path)
    ...
```

Unfortunately I have no empty disks at the moment so I have to find another bug.


### Arbitrary file read
This UI has 3 main features:

- The `App Store` that basically let's you download apps that run in Docker via `docker-compose` files (more on this later).

- The `Files` app that allows you to navigate the filesystem (only partially)

- The `ZVM` app that is a wrapper on `libvirt` and allows virtualization.

I started with the `Files` app since having a built-in function to read/write would be a nice advantage.

{{< figure src="files_app.png" title="The Files app" >}}

We are forced to read and write the content of `/DATA` and `/Media` but looking at the file download function there is a pretty straightforward parameter that will allow to read arbitray files.

Sending a `GET` request to `/v3/file` with the paramether `files` to any path outside the webroot will do the trick. 

{{< figure src="arbitrary_file_read.png" title="Details of the arbitray file read request" >}}


### Looking at CasaOS source code

The API for the `Files` app is under `/v3/file`, but sometimes also `/v2_1/files/file`, we have no source for it but answers and the requests parameters looks very similar to [CasaOS files app](https://github.com/IceWhaleTech/CasaOS/blob/main/service/file.go) so i supposeed that finding any vulnerability in that app may also work on ZimaOS WebUI. 

Looks like [there was a code injection](https://github.com/IceWhaleTech/CasaOS/blob/b331c484f593cfd155a16b03d3f175cecea4c732/service/connections.go#L59-L61) in the `MountSmaba` (yes, there is a typo) function since `host` is appended directly to the command string and [the only validation mechanism](https://github.com/IceWhaleTech/CasaOS/blob/b331c484f593cfd155a16b03d3f175cecea4c732/route/v1/samba.go#L150-L153) checks [the number of `:` in the string](https://github.com/IceWhaleTech/CasaOS/blob/b331c484f593cfd155a16b03d3f175cecea4c732/pkg/utils/ip_helper/ip.go#L10-L15) but (un)fortunately was patched last year with [this pull request](https://github.com/IceWhaleTech/CasaOS/pull/1021/files).

#### Bonus #1: Executing arbitrary commands during the update process

IceWhale decided to handle system update with [this function](https://github.com/IceWhaleTech/CasaOS/blob/8f7c99779fe31026d2b0d0fe3cb15cb25c0ebb82/service/system.go#L358-L375):

```go
func (s *systemService) UpdateSystemVersion(version string) {
	keyName := "casa_version"
	Cache.Delete(keyName)
	if file.Exists(config.AppInfo.LogPath + "/upgrade.log") {
		os.Remove(config.AppInfo.LogPath + "/upgrade.log")
	}
	file.CreateFile(config.AppInfo.LogPath + "/upgrade.log")

    if len(config.ServerInfo.UpdateUrl) > 0 {
		go command2.OnlyExec("curl -fsSL " + config.ServerInfo.UpdateUrl + " | bash")
	} else {
		osRelease, _ := file.ReadOSRelease()
		go command2.OnlyExec("curl -fsSL https://get.casaos.io/update?t=" + osRelease["MANUFACTURER"] + " | bash")
	}

}
```

the TL;DR is: a script get downloaded from `https://get.casaos.io/update` and piped to bash. 

It's honestly a questionable design but I'm not here to judge (not today atleast), but since they already got native code runnig the update script could be a native function instead.
And will for sure cut out issues like the domain being unreachable or, even worse, the update script compromised.

{{< admonition type=note title="Note" open=false >}}
The source of the update script is available [here](https://github.com/IceWhaleTech/get).
{{< /admonition >}}


During the update process, there are two paths that could be used during a race condition to turn an arbitrary file write into a code execution since every script in this directory get executed:

One is: `/tmp/casaos-installer/${randomstring}/build/scripts/migration/script.d`:

```bash
DownloadAndInstallCasaOS() {

    if [ -z "${BUILD_DIR}" ]; then
        ...
        # TMP_ROOT is TMP_ROOT=/tmp/casaos-installer
        TMP_DIR=$(${sudo_cmd} mktemp -d -p ${TMP_ROOT} || Show 1 "Failed to create temporary directory")
        ...
        BUILD_DIR=$(realpath -e "${TMP_DIR}"/build || Show 1 "Failed to find build directory")
        ...
    fi

    
    MIGRATION_SCRIPT_DIR=$(realpath -e "${BUILD_DIR}"/scripts/migration/script.d || Show 1 "Failed to find migration script directory")

    # MIGRATION_SCRIPT_DIR is /tmp/casaos-installer/${randomstring}/build/scripts/migration/script.d
    for MIGRATION_SCRIPT in "${MIGRATION_SCRIPT_DIR}"/*.sh; do
        ...
        # execute every script in that directory as root
        ${sudo_cmd} bash "${MIGRATION_SCRIPT}" || Show 1 "Failed to run migration script"
    done
 ...
}

```

and the other is `/tmp/casaos-installer/${randomstring}/build/scripts/setup/script.d`, few lines later in the same function:

```bash
  SETUP_SCRIPT_DIR=$(realpath -e "${BUILD_DIR}"/scripts/setup/script.d || Show 1 "Failed to find setup script directory")
  for SETUP_SCRIPT in "${SETUP_SCRIPT_DIR}"/*.sh; do
    # again, executing everything in that directory
    ${sudo_cmd} bash "${SETUP_SCRIPT}" || Show 1 "Failed to run setup script"
  done
```

#### Directory listing

Using `mktemp` would be a good enough method to prevent path guessing, if only there wasn't a directory listing in `/v2_1/files/file`.

Indeed, sending a `GET` request to `/v2_1/files/file` with the parameter `path` pointing to an arbitrary path will result in a directory listing, even if we shouldn't be able to interact with anthing outside `/Data` and `/Media`.

{{< figure src="directory_listing.png" title="Directory Listing" >}}

This way we can detect if any directory has been created in `/tmp/casaos-installer/` and find the name of the temporary directory.


#### Path traversal in upload
So we have an arbitrary file read, a directory listing and two entrypoints (even if we need to perform a system update), we just need an arbitrary file write to complete the chain.

Luckily (only for us tho), the same endpoint we used to read arbitrary files [has an upload function with a path traversal vulnerability ](https://github.com/IceWhaleTech/CasaOS/blob/8f7c99779fe31026d2b0d0fe3cb15cb25c0ebb82/service/file_upload.go#L60-L167)

```go
func (s *FileUploadService) UploadFile(
  ...
) error {
	...

    // path and relativePath are sent by the user
	file, err := os.OpenFile(path+"/"+relativePath+".tmp", os.O_WRONLY|os.O_CREATE, 0644)
	...
    src, err := bin.Open()
	if err != nil {
		return err
	}
	defer src.Close()

	_, err = io.Copy(file, src)

	...
}
```

{{< figure src="arbitrary_file_upload.png" title="Arbitrary file upload" >}}

#### A valid entrypoint
Digging deeper I found [another interesting entrypoint to turn arbitrary file write into code execution](https://github.com/IceWhaleTech/CasaOS/blob/8f7c99779fe31026d2b0d0fe3cb15cb25c0ebb82/pkg/utils/command/command_helper.go#L111):
This time the only requirement is a reboot (the webUI allows to reboot from web interface).

In brief, every file placed in `/etc/casaos/start.d` will be executed [at boot](https://github.com/IceWhaleTech/CasaOS/blob/8f7c99779fe31026d2b0d0fe3cb15cb25c0ebb82/main.go#L203):


```go
func ExecuteScripts(scriptDirectory string) {
    ...
    // scriptDirectory is /etc/casaos/start.d
	files, err := os.ReadDir(scriptDirectory)
    ..
    // for each file in /etc/casaos/start.d
	for _, file := range files {
        ...
		scriptFilepath := filepath.Join(scriptDirectory, file.Name())

        ...
        // execute the script
		cmd := exec.Command(interpreter, scriptFilepath)
        ...
}
```

{{< figure src="ExecuteScripts_Logic.png" title="ExecuteScripts() logic flow" >}}

#### Abusing the move and copy funtionality
Let's pretend for a second we don't have any of the previous arbitrary read/write vulnerabilities, what else can we use?

Another intereting functionality CasaOS has is the ability to [move and copy files](https://github.com/IceWhaleTech/CasaOS/blob/8f7c99779fe31026d2b0d0fe3cb15cb25c0ebb82/service/file.go#L119-L126) (supposedly inside the allowed directory).

Since there is no mechanism in place to check _what_ and _where_ is being moved/copied, we can abuse this functionality to get read/write in the whole filesystem.

```go
if temp.Type == "move" {
			lastPath := v.From[strings.LastIndex(v.From, "/")+1:]
			if !file.CheckNotExist(temp.To + "/" + lastPath) {
				if temp.Style == "skip" {
					temp.Item[i].Finished = true
					continue
				} else {
					os.RemoveAll(temp.To + "/" + lastPath)
				}
			}
			err := file.CopyDir(v.From, temp.To, temp.Style)
			if err == nil {
				err = os.RemoveAll(v.From)
				if err != nil {
					logger.Error("file move error", zap.Any("err", err))
					err = file.MoveFile(v.From, temp.To+"/"+lastPath)
					if err != nil {
						logger.Error("MoveFile error", zap.Any("err", err))
						continue
					}

				}
			}
```

Indeed, sending a `POST` request to `/v2_1/files/task` with the following body:

```json
{
  "to":"/etc/casaos/start.d",
  "type":"move",
  "item":[
	{
		"from": "/media/ZimaOS-HD/Downloads/payload"
	 }
  ]
}
```
allows us to move files around in places we are not supposed to, easy as that.

A standard WebUI user could just move files in the `/Media` directory, that is allowed to read, or upload files to `/Media` and then move them around arbitrarily.
So this functionality can be effectively abused to get full code execution on the machine since we eariler found that everything in `/etc/casaos/start.d` grants execution at boot.


### Finalizing the exploit

So I finally have enough to resolve the issue. 

I wrote few lines of code to do the following:

- Write a file containing a reverse shell to `/media/ZimaOS-HD/Downloads/` (intended functionality)
- Move the file to `/etc/casaos/start.d`
- Use the directory listing to check if the file has been succesfully moved
- Reboot the machine

{{< admonition type=note title="Note" open=false >}}
Since the script is executed at boot the network interface could still be in the process of going up, so remember to add a small timeout before executing the payload.
{{< /admonition>}}

The PoC is available at https://github.com/himazawa/zimaos-rce-poc

{{< figure src="rev_shell.png" title="Problem Solved" >}}


{{< admonition type=note title="Note" open=true >}}
Due to the number of vulnerabilities found there are plenty of other ways to do the same thing. I decided to use the move functionality just because it looked more interesting to me.
Feel free to experiment and write your own chain.
{{< /admonition>}}



### Bonus #3: DOM based XSS
I have no interest in making the chain remote, but there is a DOM based xss at `/v2/app_management/web/sync/appstore` in the `url` parameter and the payload is `:<img src=x onerror=alert(1) >`.
Note the `:` because they are needed in order to make this [function fail](https://github.com/IceWhaleTech/CasaOS-AppManagement/blob/8d0082ad64c2fc4209031284abcf7c9d252e0d72/service/appstore_management.go#L99)

{{< figure src="XSS.png" title="XSS" >}}

## Conclusion
In the end it was a fun Saturday that demostrated that good old web vulnerabilities are still a thing :P
CasaOS and ZimaOS are being reworked right now to fix the issues and the advisories will be published soon.

## Timeline
- 24/06/2024 - Vulnerability Reported
- 03/08/2024 - A fix for the code injection was released
- 07/08/2024 - Asked IceWhale permission to release this post
