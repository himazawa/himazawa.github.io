# Debloating the Onyx Boox Go 10.3


I was looking for an eink tablet to r
ead books and take notes while I'm away from home.

After adventuring in the eInk rabbit hole I decided to go for the [Onyx Boox Go 10.3](https://amzn.to/3SC2W6Z): a Black and White eInk Android Tablet with 300ppi that's also good for taking notes, weighting only 365g!

I was a bit concerned about [this report from Mozilla](https://foundation.mozilla.org/en/privacynotincluded/onyx-boox/) so I decided to take a look at the device.

{{< admonition type=danger title="" open=true >}}
Boox is using a custom version of the kernel [and it's not sharing the source](https://news.ycombinator.com/item?id=23735962) so act accordingly.
{{< /admonition>}}

## Enabling ADB
The Onyx default Launcher is hiding a lot of the Android inner features but reading a bit of documentation I figured out how to enable ADB.

__TL;DR__: Press the `Apps` Icon on the left -> Top right icon -> `App Management` -> Enable "USB Debug Mode".

Connecting the device via usb and running `adb shell` will open a shell on the device.


{{< figure src="onyx_settings.jpg" title="Enable ADB via Onyx Launcher" >}}


## Removing unnecessary apps
Navigating the UI I figured out the tablet is full of useless (for me) apps, so I decided to remove them.
After [installing adb](https://www.xda-developers.com/install-adb-windows-macos-linux/#how-to-set-up-adb-on-your-computer) I removed the unwanted apps with the following commands:

```adb
pm uninstall com.android.bips
pm uninstall com.android.bluetoothmidiservice
pm uninstall com.android.cameraextensions
pm uninstall com.android.cts.ctsshim
pm uninstall com.android.dreams.phototable
pm uninstall com.android.emergency
pm uninstall com.android.internal.display.cutout.emulation.corner
pm uninstall com.android.internal.display.cutout.emulation.double
pm uninstall com.android.internal.display.cutout.emulation.hole
pm uninstall com.android.internal.display.cutout.emulation.tall
pm uninstall com.android.internal.display.cutout.emulation.waterfall
pm uninstall com.android.internal.systemui.navbar.gestural
pm uninstall com.android.internal.systemui.navbar.gestural_extra_wide_back
pm uninstall com.android.internal.systemui.navbar.gestural_narrow_back
pm uninstall com.android.internal.systemui.navbar.gestural_wide_back
pm uninstall com.android.internal.systemui.navbar.threebutton
pm uninstall com.android.printservice.recommendation
pm uninstall com.android.providers.blockednumber
pm uninstall com.android.providers.contacts
pm uninstall com.android.quicksearchbox
pm uninstall com.android.smspush
pm uninstall com.android.theme.font.notoserifsource
pm uninstall com.android.vending
pm uninstall com.google.android.apps.restore
pm uninstall com.google.android.gms.location.history
pm uninstall com.google.android.overlay.gmsconfig.common
pm uninstall com.google.android.syncadapters.calendar
pm uninstall com.google.android.syncadapters.contacts
pm uninstall com.onyx.aiassistant #AI Assistant app
pm uninstall com.onyx.android.production.test #Test feature
pm uninstall com.onyx.appmarket # Appmarket app
pm uninstall com.onyx.calculator # Calculator app
pm uninstall com.onyx.clock # Clock app
pm uninstall com.onyx.dict # Dict app
pm uninstall com.onyx.easytransfer # Easytransfer app
pm uninstall com.onyx.gallery # Gallery app
pm uninstall com.onyx.kime # Kime app
pm uninstall com.onyx.latinime # Keyboard app, removing it will leave you with the Google Speech-to-text keyboard 
pm uninstall com.onyx.mail # Mail app
pm uninstall com.onyx.musicplayer # Music player
pm uninstall com.onyx.voicerecorder # Voice Recorder
pm uninstall com.qualcomm.embms
pm uninstall com.qualcomm.qti.seccamservice
pm uninstall com.qualcomm.qti.server.qtiwifi
pm uninstall com.qualcomm.qti.services.systemhelper
pm uninstall com.qualcomm.qti.uim
pm uninstall com.qualcomm.qti.uimGbaApp
pm uninstall com.qualcomm.qti.xrcb
pm uninstall com.qualcomm.qti.xrvd.service
pm uninstall com.qualcomm.qtil.aptxui
pm uninstall org.chromium.chrome # Chrome browser
```

After installing Nova Launcher and selecting it as the default launcher I was able to also remove this three apps.

{{< admonition type=warning title="Warning" open=true >}}
Removing this apps without changing launcher will result in Onyx Launcher crashing.
{{< /admonition>}}

```adb
pm uninstall com.onyx.igetshop # iGetShop app
pm uninstall com.onyx.android.ksync # Ksync app,this app is doing a lot of calls to Onyx servers
pm uninstall com.onyx.kreader # Reader and Note taking app
```


### For less tech-savvy people
If you don't want to play around with the shell and manual adb commands, I added the Onyx apps to the [Android Universal Debloater](https://github.com/Universal-Debloater-Alliance/universal-android-debloater-next-generation) list.

{{< admonition type=note title="Note" open=true >}}
With UAD you can create a backup, it's highly recommended if you are unsure on what you are doing
{{< /admonition>}}

Just run it and select the packages I listed in the previous sections.

{{< figure src="uad.png" title="53 packages uninstalled" style="border: solid 1px red" >}}


## Installing an alternative store
Since I removed the default Onyx Store, I decided to install Obtanium to get releases directly from the source.

Go [here](https://obtainium.imranr.dev/), download the APK and install it via adb with:

```adb
adb install app-release.apk
```

Why not F-Droid you ask? Because F-Droid doesn't get updates directly from the source and sometimes are slow with releases alignment.


### Minor changes to avoid phoning home
Onyx has a lot of calls to their domains, I just replaced them with different endpoints.

## Changing NTP Server
Onyx uses its own NTP server, I replaced it with `ntp.org`

```adb
abd shell settings put global ntp_server pool.ntp.org
adb shell settings put global ntp_server_2 en.pool.ntp.org # replace "en" with your local time
```

## Change captive portal endpoints
Onyx uses their endpoints for the captive portal pings so I switched them with the ones from Android:

```adb
adb shell settings put global captive_portal_http_url "http://connectivitycheck.android.com/generate_204"
adb shell settings put global captive_portal_https_url "https://connectivitycheck.android.com/generate_204"
adb shell settings put global captive_portal_fallback_url "http://connectivitycheck.gstatic.com/generate_204"
```

## Bonus: Blocking onyx domains
Just in case I added the following domains to my blocklist:

```
send2boox.com # used by KSync
push.boox.com # used by KSync
en-rom.boox.com
onyx-international.cn #Used for NTP Server
```

## Conclusion
In the end the Onyx Boox Go 10.3 is a very nice device (hardware-wise) that suits my needs.
If you want to buy it consider everything written in this post and make an informed purchease.

P.S. also consider that the device is new at the time of writing and the default cover is terrible.

## Useful Resources
- [Manifacturer website](https://www.boox.com/)
- [Mobile Read thread on Onyx Boox Debloat](https://www.mobileread.com/forums/showthread.php?t=349930)
- [Reddit Onyx Boox Community](https://www.reddit.com/r/Onyx_Boox/)
- [Reddit eink Community](https://www.reddit.com/r/eink)

