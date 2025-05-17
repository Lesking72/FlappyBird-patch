## Flappy Bird universal architecture & API-level patch
### Why
Flappy Bird, being a product of 2014, was not produced for 64-bit devices. Google stopped including 32-bit ABIs starting with the Pixel 7 and many phones released since then are also 64-bit only. Additionally, Android 14+ blocks on-device installation of apps targeting Android versions below 6.0. Android 15 raised this requirement to 7.0.

### How
Download [the APK in this repository](https://github.com/Lesking72/FlappyBird-patch/raw/refs/heads/master/com.dotgears.flappybird-anyarch+api_patched.apk) or follow these directions to patch it yourself.

You need [Java](https://adoptium.net/), [apktool](https://apktool.org/), and [uber-apk-signer](https://github.com/patrickfav/uber-apk-signer)

1. Unpack the original Flappy Bird APK: `java -jar apktool_2.11.1.jar d <apk_file>.apk -o apk_out`
    - This unpacks the APK into a folder called "apk_out"
1. Delete the `apk_out/lib` folder.
    - Flappy Bird only uses one native library: `libandengine.so`. According to [this](https://stackoverflow.com/questions/56642331/looking-for-andengine-64bit-version) Stack Overflow post, this library works around bugs in Android 2.x and 3.x. Above Android 4.0, these libraries aren't needed. The rest of the game is Dalvik code that runs on any architecture.
1. Open `apk_out/AndroidManifest.xml` and delete these lines:
    ```
        <uses-permission android:name="android.permission.INTERNET"/>
        <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
        <activity android:configChanges="keyboard|keyboardHidden|orientation|screenLayout|screenSize|smallestScreenSize|uiMode" android:name="com.google.ads.AdActivity"/> 
    ```
1. Add `android:exported="true"` to the first activity tag
1. Add these parameters to both activity tags: `android:resizeableActivity="false" android:maxAspectRatio="1.78"`
1. Open `apk_out/smali/com/dotgears/b.smali` and change line 32 from `const/4 v1, 0x0` to `const/4 v1, 0x4`
    - This stops the ad library from complaining that it has no Internet permission.
1. Open `apk_out/apktool.yml`, change minSdkVersion to 14 and targetSdkVersion to 34
    - Changing the minimum API is not required, but we _did_ remove those workarounds.
1. Repack the APK: `java -jar apktool_2.11.1.jar b apk_out`
    - You now have the repacked APK at `apk_out/dist/<apk_file>.apk`, but you need to sign it before you can install it.
1. Sign the APK: `java -jar uber-apk-signer-1.3.0.jar -a apk_out/dist/<apk_file>.apk`
    - Note: The APK is signed with a public debug key. This poses a slight security risk since anyone can use this key. For this to happen, an attacker needs access to your unlocked phone and to know you have com.dotgears.flappybird signed with a debug key, or you need to be dumb enough to install an update to an app that's been dead for eleven years when you know the app you're updating is signed with a debug key. Do your friends a favor and don't send the apk to them without telling them this, or generate your own keypair to sign the app. The APK I've provided is signed with a private key.

The signed APK will be at `apk_out/dist/<apk_file>-aligned-debugSigned.apk`.

### Bypassing the Minimum SDK requirement
This is not required if you changed the Target API.
Enable USB debugging on your phone and download [Android SDK Platform Tools](https://developer.android.com/tools/releases/platform-tools). Then `adb install --bypass-low-target-sdk-block <apk_file>`

### Effects of changing API Levels
- At API 24, the app is no longer letterboxed to a 16:9 aspect ratio. This is alleviated by adding `android:resizeableActivity="false"`
- At API 26, the app stops being letterboxed again. This is alleviated by adding `android:maxAspectRatio="1.78"`
- At API 31, the apk won't install without setting `android:exported` on activities with intent filters.
- At API 35, the window is always drawn behind the display cutout.
