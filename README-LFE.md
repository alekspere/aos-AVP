# NOVA 6.6.5 with optional LFE boost

This branch builds the existing NOVA 6.6.5 release with an Android TV checkbox that increases the LFE contribution in FFmpeg stereo downmix from 0.5 to 1/sqrt(2), approximately 0.70710678 (+3.01 dB). It defaults off, requires forced stereo downmix with PCM output, and preserves the saved preference across upgrades. Passthrough and spatialization disable the effective boost.

## Git checkout over SSH

The Git submodules are pinned to the same exact component commits as the build manifest:

```sh
git clone --recurse-submodules --branch lfe-stereo-downmix-v6.6.5 git@github.com:alekspere/aos-AVP.git
cd aos-AVP
git -C Video switch lfe-stereo-downmix-v6.6.5
git -C MediaLib switch lfe-stereo-downmix-v6.6.5
git -C native/avos switch lfe-stereo-downmix-v6.6.5-release
```

The parent and the three modified components use the personal forks; dependency repositories use explicit upstream SSH URLs. The native feature branch is also available for preparing the upstream PR. After updating the parent checkout, use `git submodule update --init --recursive` to restore its pinned component revisions.

## Android repo checkout

The Android repo tool is also supported:

```sh
mkdir nova-lfe
cd nova-lfe
repo init -u https://github.com/alekspere/aos-AVP -b lfe-stereo-downmix-v6.6.5
repo sync -j4 -d
repo manifest -r -o manifest-pinned.xml
```

The branch's `default.xml` points to `lfe-v6.6.5.xml`. It pins Video, MediaLib and native/avos to exact commits in the personal forks. The other 35 dependency projects use the exact commits from the published 6.6.5 release manifest. Two additional libdovi projects are pinned to the revisions used by the validated 6.6.5 build, because the release manifest omitted them despite the native build requiring them. The AVP build project follows this manifest branch; `repo manifest -r` records its resolved commit too. Keep the pinned revisions when syncing.

## Build

Set up the Android SDK/NDK and JDK as described in README.md. From the repo workspace:

```sh
cd Video
BUILD=RELEASE NDK_DEBUG=0 ./gradlew -Poneapk assembleNoamazonRelease
```

Use your own local keystore configuration to sign release APKs. The Xiaomi Box S used for validation requires the armeabi-v7a APK. Build outputs are under `Video/build/outputs/apk/noamazon/release`.

## Source branches and future upstream review

- `aos-Video/lfe-stereo-downmix-v6.6.5`: preference UI and effective playback setting.
- `aos-MediaLib/lfe-stereo-downmix-v6.6.5`: LibAvos Java API.
- `aos-avos/lfe-stereo-downmix-v6.6.5`: coefficient precision correction followed by the optional LFE feature and production conversion regression test.
- `aos-avos/lfe-stereo-downmix-v6.6.5-release`: the same feature plus the independent release-build stream buffer accessor fix. The build manifest selects this commit so release APKs include that prerequisite.

The native feature branch keeps the release-build fix out of the future LFE PR. The coefficient correction is a separate commit for independent review. The related native, Java bridge and UI changes span three upstream repositories.

No limiter or normalization is added. Legacy downmix.c/downmix.h, MediaCodec and the release's dependency versions are retained.

## Validation

The same source combination produced the custom 6.6.5 release APK tested on Xiaomi Box S. R8 optimization and resource shrinking were enabled; lint passed with zero errors and 18 warnings. The native production FFmpeg conversion test passed 9,410 checks on the Box, including toggling a cached matrix, 5.1/7.1 layouts, eight PCM formats, channel routing, saturation and unchanged multichannel output.

Run `native/avos/test/run-ffmpeg-downmix.sh` from the repo workspace with ANDROID_NDK_HOME set and adb on PATH. Set ANDROID_SERIAL if multiple devices are connected. The +3 dB gain of the prior implementation was also confirmed by the user with REW and a calibrated microphone.
