# RNEF Android GitHub Action

This GitHub Action enables remote building of Android applications using RNEF (React Native Enterprise Framework). It supports both debug and release builds, with automatic artifact caching and code signing capabilities.

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
        uses: callstackincubator/android@v2 # replace with latest commit hash
        with:
          variant: 'debug' # or else
          github-token: ${{ secrets.GITHUB_TOKEN }}
          # For release builds, add these:
          # sign: true
          # keystore-base64: ${{ secrets.KEYSTORE_BASE64 }}
          # keystore-store-file: 'your-keystore.jks'
          # keystore-store-password: ${{ secrets.KEYSTORE_STORE_PASSWORD }}
          # keystore-key-alias: 'your-key-alias'
          # keystore-key-password: ${{ secrets.KEYSTORE_KEY_PASSWORD }}
```

## Inputs

| Input                     | Description                              | Required | Default |
| ------------------------- | ---------------------------------------- | -------- | ------- |
| `github-token`            | GitHub Token                             | Yes      | -       |
| `working-directory`       | Working directory for the build command  | No       | `.`     |
| `validate-gradle-wrapper` | Whether to validate the Gradle wrapper   | No       | `true`  |
| `setup-java`              | Whether to run actions/setup-java action | No       | `true`  |
| `variant`                 | Build variant (debug/release)            | No       | `debug` |
| `sign`                    | Whether to sign the build with keystore  | No       | -       |
| `re-sign`                 | Re-sign the APK with new JS bundle       | No       | `false` |
| `keystore-base64`         | Base64 encoded keystore file             | No       | -       |
| `keystore-store-file`     | Keystore store file name                 | No       | -       |
| `keystore-store-password` | Keystore store password                  | No       | -       |
| `keystore-key-alias`      | Keystore key alias                       | No       | -       |
| `keystore-key-password`   | Keystore key password                    | No       | -       |
| `rnef-build-extra-params` | Extra parameters for rnef build:android  | No       | -       |
| `comment-bot`             | Whether to comment PR with build link    | No       | `true`  |

## Outputs

| Output         | Description               |
| -------------- | ------------------------- |
| `artifact-url` | URL of the build artifact |
| `artifact-id`  | ID of the build artifact  |

## Prerequisites

- Ubuntu runner
- RNEF CLI installed in your project
- For release builds:
  - Valid Android keystore file
  - Proper code signing setup

## License

MIT
