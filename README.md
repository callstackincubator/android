# Rock Android GitHub Action

This GitHub Action enables remote building of Android applications using Rock. It supports both debug and release builds, with automatic artifact caching and code signing capabilities.

## Features

- Build Android apps in debug or release mode
- Automatic artifact caching to speed up builds
- Code signing support for release builds
- Re-signing capability for PR builds
- Native fingerprint-based caching
- Configurable build parameters
- Gradle wrapper validation

## Usage

```yaml
name: Android Build
on:
  push:
    branches: [main]
  pull_request:
    branches: ['**']

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2

      - name: Build Android
        uses: callstackincubator/android@v3 # replace with latest commit hash
        with:
          variant: 'debug' # or else
          github-token: ${{ secrets.GITHUB_TOKEN }}
          # For release builds, add these:
          # sign: true
          # Option 1: Use keystore file directly
          # keystore-file: 'path/to/your-keystore.jks'
          # Option 2: Use base64 encoded keystore (alternative to keystore-file)
          # keystore-base64: ${{ secrets.KEYSTORE_BASE64 }}
          # keystore-store-file: 'your-keystore.jks'
          # keystore-store-password: ${{ secrets.KEYSTORE_STORE_PASSWORD }}
          # keystore-key-alias: 'your-key-alias'
          # keystore-key-password: ${{ secrets.KEYSTORE_KEY_PASSWORD }}
          # keystore-path: 'tools/buildtools/upload-key.keystore' # Optional: for custom keystore locations
```

## Inputs

| Input                     | Description                              | Required | Default            |
| ------------------------- | ---------------------------------------- | -------- | ------------------ |
| `github-token`            | GitHub Token                             | Yes      | -                  |
| `working-directory`       | Working directory for the build command  | No       | `.`                |
| `validate-gradle-wrapper` | Whether to validate the Gradle wrapper   | No       | `true`             |
| `setup-java`              | Whether to run actions/setup-java action | No       | `true`             |
| `variant`                 | Build variant (debug/release)            | No       | `debug`            |
| `sign`                    | Whether to sign the build with keystore  | No       | -                  |
| `re-sign`                 | Re-sign the APK with new JS bundle       | No       | `false`            |
| `keystore-file`           | Path to the keystore file                | No       | -                  |
| `keystore-base64`         | Base64 encoded keystore file             | No       | -                  |
| `keystore-store-file`     | Keystore store file name                 | No       | -                  |
| `keystore-store-password` | Keystore store password                  | No       | -                  |
| `keystore-key-alias`      | Keystore key alias                       | No       | -                  |
| `keystore-key-password`   | Keystore key password                    | No       | -                  |
| `keystore-path`           | where the keystore should be placed      | No       | `release.keystore` |
| `rock-build-extra-params` | Extra parameters for rock build:android  | No       | -                  |
| `comment-bot`             | Whether to comment PR with build link    | No       | `true`             |
| `custom-ref`              | Custom app reference for artifact naming | No       | -                  |

## Outputs

| Output         | Description               |
| -------------- | ------------------------- |
| `artifact-url` | URL of the build artifact |
| `artifact-id`  | ID of the build artifact  |

## Code Signing

When `sign: true` is enabled, this action configures Android code signing by setting Gradle properties. It supports **two property conventions** for maximum compatibility:

### Android injected properties

(this is an undocumented feature used by Fastlane and AGP)

The action automatically sets `android.injected.signing.*` properties which are natively recognized by the Android Gradle Plugin. These properties work with any standard `build.gradle` configuration without modifications:

```gradle
signingConfigs {
    release {
        // These hardcoded values will be automatically overridden
        storeFile file('path/to/keystore.jks')
        keyAlias 'placeholder'
        storePassword 'placeholder'
        keyPassword 'placeholder'
    }
}
```

### Custom ROCK Properties

For apps that explicitly read custom properties in their `build.gradle`, the action also sets `ROCK_UPLOAD_*` properties:

```gradle
signingConfigs {
    release {
        storeFile file('path/to/keystore.jks')
        keyAlias project.findProperty('ROCK_UPLOAD_KEY_ALIAS') ?: 'placeholder'
        storePassword project.findProperty('ROCK_UPLOAD_STORE_PASSWORD') ?: 'placeholder'
        keyPassword project.findProperty('ROCK_UPLOAD_KEY_PASSWORD') ?: 'placeholder'
    }
}
```

The following mappings are set:

- `ROCK_UPLOAD_KEY_ALIAS` ← `inputs.keystore-key-alias`
- `ROCK_UPLOAD_STORE_FILE` ← `inputs.keystore-store-file`
- `ROCK_UPLOAD_STORE_PASSWORD` ← `inputs.keystore-store-password`
- `ROCK_UPLOAD_KEY_PASSWORD` ← `inputs.keystore-key-password`

Both conventions are set simultaneously, so the action works with any existing build configuration.

## Prerequisites

- Ubuntu runner
- Rock CLI installed in your project
- For release builds:
  - Valid Android keystore file
  - Proper code signing setup

## License

MIT
