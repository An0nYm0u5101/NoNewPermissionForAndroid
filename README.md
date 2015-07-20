# NoNewPermissionForAndroid
Permission matters!  That's what users care about the most.  They hate every permission from the bottom of heart, especially the **newly added permission(s)**.  So it's better to check if you have added any new permission to your Android app by any means (e.g. the external library could bring some new permissions).

This script could help warn developers of any permission change by setting up a CI job.

## Precondition
* Ruby
* rake gem (can install by running `gem install rake`)

## Available tasks
```
$ rake -D

rake examine[android_build_tools_path,apk_file]
    Examine the apk and the snapshot to compare the permission change

rake take_snapshot[android_build_tools_path,apk_file]
    Generate the reference snapshot file, before the first run
```

## Jenkins Configure
* **Build** -> **Execute shell** -> **Command**
```
#!/usr/bin/env bash
cd <DIRECTORY_OF_THIS_SCRIPT>
rake examine[<ANDROID_SDK_PATH>/build-tools/<BUILD_TOOLS_REV>,<ANDROID_APK_PATH>]
```

## Results
There are 4 possible cases (++, --, ++&--, ==), and the results could look like the example output below:
* permission++
* => Failure (exit code 1)
```
======================================================================
4 new permissions added:
    android.permission.CAMERA
    android.permission.FLASHLIGHT
    android.permission.SEND_SMS
    com.me.app.myapp.permission.DEADLY_ACTIVITY
======================================================================
```

* permission++  &  permission--
* => Failure (exit code 1)
```
======================================================================
4 new permissions added:
    android.permission.WRITE_EXTERNAL_STORAGE
    com.sonyericsson.home.permission.BROADCAST_BADGE
    com.sec.android.provider.badge.permission.READ
    com.sec.android.provider.badge.permission.WRITE

2 old permissions removed:
    android.permission.CAMERA
    android.permission.FLASHLIGHT
======================================================================
```

* permission--
* => Success (exit code 0)  & update snapshot automatically
```
======================================================================
Brilliant!  You got 3 permissions removed:
    android.permission.CAMERA
    android.permission.FLASHLIGHT
    com.me.app.myapp.permission.DEADLY_ACTIVITY

Snapshot file has been updated.
======================================================================
```

* permission==
* => Success (exit code 0)
```
======================================================================
No permission is changed.
======================================================================
```

## Note
* Before running the regular CI job, you must generate a snapshot file first:
```
rake take_snapshot[<ANDROID_SDK_PATH>/build-tools/<BUILD_TOOLS_REV>,<ANDROID_APK_PATH>]
```
* If you use zsh, you may encounter `no matches found`, then please try to add `\` before bracket, like below:
```
rake examine\[<ANDROID_SDK_PATH>/build-tools/<BUILD_TOOLS_REV>,<ANDROID_APK_PATH>\]
```
