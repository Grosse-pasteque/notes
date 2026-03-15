# Apk decompiling
**You must have a plugged device to extract from**
### List plugged devices
```bash
adb devices
```
### Find the app you want to extract
```bash
adb shell pm list packages | grep example
```
### Find the path to this specific app
```bash
adb shell pm path com.example
```
### Extract it
```bash
adb pull /data/app/com.example-XXXX/base.apk
```
### Unzip it
*NOTE: apks are basically zip bundles*
```bash
unzip base.apk -d example
```
