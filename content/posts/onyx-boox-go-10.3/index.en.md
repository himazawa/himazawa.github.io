---
title: "You should get an eInk Android Tablet"
date: 2024-08-02T12:00:00+01:00
draft: true
tags: [android, eink, Onyx Boox]
categories: [reviews]
featuredImage: boox_go_cover.jpg
---
I decided to buy 

## Enabling ADB


## Removing unnecessary apps
```bash
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
pm uninstall com.onyx.latinime # Keaboard app, removing it will leave you with the Google Speech-to-text keaboard 
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

After installing Nova Launcher and selecting it as the default launcher I was able to also remove this two apps.

{{< admonition type=warning title="Warning" open=true >}}
Removing this apps without changing launcher will result in Onyx Launcher crashing.
{{< /admonition>}}

```bash
pm uninstall com.onyx.igetshop #iGetShop app
pm uninstall com.onyx.android.ksync # Ksync app,this app is doing a lot of calls to Onyx servers
pm uninstall com.onyx.kreader # Reader and Note taking app
```

## Changing NTP Server
For some reasons Onyx has their NTP server, i replaced it with `ntp.org`

```bash
abd shell settings put global ntp_server pool.ntp.org
adb shell settings put global ntp_server_2 en.pool.ntp.org # replace "en" with your local time
```

## Change captive portal endpoints

```bash
adb shell settings put global captive_portal_http_url "http://connectivitycheck.android.com/generate_204"
adb shell settings put global captive_portal_https_url "https://connectivitycheck.android.com/generate_204"
adb shell settings put global captive_portal_fallback_url "http://connectivitycheck.gstatic.com/generate_204"
adb shell settings put global network_avoid_bad_wifi 0
```


