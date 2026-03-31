# Building MuxPod APK

## Prerequisites

- Flutter 3.24+ (stable channel)
- Android SDK 36+ with build-tools
- Java 17+

## Local Build (x86_64 Linux / macOS)

```bash
# Install dependencies
flutter pub get

# Build release APK (debug-signed, no keystore needed)
flutter build apk --release --no-tree-shake-icons

# Output: build/app/outputs/flutter-apk/app-release.apk
```

## Local Build with Release Signing

1. Generate a keystore (one-time):

```bash
keytool -genkey -v \
  -keystore android/app/upload-keystore.jks \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias upload
```

2. Create `android/key.properties`:

```properties
storePassword=<your-store-password>
keyPassword=<your-key-password>
keyAlias=upload
storeFile=upload-keystore.jks
```

3. Build:

```bash
flutter build apk --release
```

## CI Build (GitHub Actions)

The repo includes a workflow at `.github/workflows/build-apk.yml` that builds on Ubuntu x86_64 runners.

### Trigger manually:

```bash
gh workflow run build-apk.yml
```

### Download the artifact:

```bash
# Find the latest run
gh run list -w build-apk.yml -L 1

# Download APK
gh run download <RUN_ID> -n muxpod-release-apk
```

### Why CI?

Building Android APKs requires x86_64 Linux or macOS. The Android SDK tools (AAPT2, gen_snapshot) are not available for aarch64/ARM64 Linux hosts.

## Notes

- `android/key.properties` and `android/app/upload-keystore.jks` are gitignored
- Without a keystore, the release build falls back to debug signing (sideload-only, not Play Store ready)
- APK size: ~75MB (fat APK with arm, arm64, x64 architectures)
