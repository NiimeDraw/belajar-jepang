# Belajar Jepang — Android App (Offline)

Aplikasi Android offline untuk belajar menulis Hiragana, Katakana, dan Kanji dasar.
Dibangun sebagai WebView wrapper di atas satu file HTML mandiri (tanpa internet).

## Isi
- `www/index.html` — web app mandiri (offline, tanpa dependensi eksternal)
- `app/` — proyek Android minimal (satu Activity + WebView)
- `NOTES.md` — catatan build

## Build
Lihat `NOTES.md`. Ringkasnya (tanpa Gradle — build manual via aapt2/d8/apksigner):

```bash
export JAVA_HOME=<jdk17> PATH=$JAVA_HOME/bin:<build-tools-34.0.0>:$PATH
AJAR=<sdk>/platforms/android-34/android.jar
aapt2 link -o out/app-unsigned.apk -I $AJAR --manifest app/src/main/AndroidManifest.xml \
  -A app/src/main/assets --min-sdk-version 24 --target-sdk-version 34
javac -cp $AJAR -d build/classes app/src/main/java/com/muse/nihongo/MainActivity.java
d8 --lib $AJAR --min-api 24 --output build/dex build/classes/com/muse/nihongo/MainActivity.class
(cd out && cp ../build/dex/classes.dex . && zip -q app-unsigned.apk classes.dex && rm classes.dex)
zipalign -f 4 out/app-unsigned.apk out/app-aligned.apk
apksigner sign --ks debug.keystore --ks-pass pass:android --key-pass pass:android \
  --out out/belajar-jepang.apk out/app-aligned.apk
```

## Instal
Unduh `dist/belajar-jepang.apk` dari repo ini, izinkan "instal dari sumber tidak dikenal".
