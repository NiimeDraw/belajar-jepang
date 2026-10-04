# Build notes — Belajar Jepang APK

Web artifact: `belajar-menulis-hiragana-katakana-kanji` (web_static, slug sama).
APK: WebView wrapper di atas file HTML tunggal offline.

## Alur update
1. Ubah web artifact dulu via `artifact.edit` (slug: belajar-menulis-hiragana-katakana-kanji).
2. Export ulang: `artifact.export` slug tersebut → `~/workspace/your_files/belajar-menulis-hiragana-katakana-kanji/*.html`
3. Buat versi offline: hapus tag `<link>` Google Fonts (Android pakai font sistem).
   → `~/workspace/apk_build/www/index.html`
4. Salin ke `app/src/main/assets/www/index.html`
5. Naikkan `versionCode`/`versionName` di `app/src/main/AndroidManifest.xml`.
6. Build manual (Gradle daemon DIBLOKIR oleh sandbox: "other_tcp = Deny"
   mengintersep semua koneksi TCP mentah Java, termasuk localhost):

```bash
cd ~/workspace/apk_build
export JAVA_HOME=$PWD/jdk-17.0.20.1+1
export PATH=$JAVA_HOME/bin:$PWD/sdk/build-tools/34.0.0:$PATH
AJAR=$PWD/sdk/platforms/android-34/android.jar
aapt2 link -o out/app-unsigned.apk -I $AJAR \
  --manifest app/src/main/AndroidManifest.xml -A app/src/main/assets \
  --min-sdk-version 24 --target-sdk-version 34
javac -nowarn -cp $AJAR -d build/classes app/src/main/java/com/muse/nihongo/MainActivity.java
d8 --lib $AJAR --min-api 24 --output build/dex build/classes/com/muse/nihongo/MainActivity.class
cp build/dex/classes.dex out/ && (cd out && zip -q app-unsigned.apk classes.dex && rm classes.dex)
zipalign -f 4 out/app-unsigned.apk out/app-aligned.apk
apksigner sign --ks debug.keystore --ks-pass pass:android --key-pass pass:android \
  --out out/belajar-jepang.apk out/app-aligned.apk
apksigner verify out/belajar-jepang.apk
```

7. Salin hasil ke `~/workspace/your_files/belajar-jepang-apk/belajar-jepang.apk` lalu lampirkan.

## Catatan
- Keystore: `debug.keystore` (alias androiddebugkey, pass android). Self-signed,
  user harus izinkan "instal dari sumber tidak dikenal".
- Package: `com.muse.nihongo`, minSdk 24, targetSdk 34.
- SDK manual di `sdk/` (platforms/android-34, build-tools/34.0.0). Jangan pakai
  sdkmanager/Gradle: butuh TCP langsung yang diblokir sandbox.
