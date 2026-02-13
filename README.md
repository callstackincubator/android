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
          # For store AAB add this:
          # aab: true
```

## Inputs

| Input                     | Description                              | Required | Default            |
| ------------------------- | ---------------------------------------- | -------- | ------------------ |
| `github-token`            | GitHub Token                             | Yes      | -                  |
| `working-directory`       | Working directory for the build command  | No       | `.`                |
| `validate-gradle-wrapper` | Whether to validate the Gradle wrapper   | No       | `true`             |
| `setup-java`              | Whether to run actions/setup-java action | No       | `true`             |
| `variant`                 | Build variant (debug/release)            | No       | `debug`            |
| `aab`                     | Build Android App Bundle instead of APK  | No       | `false`            |
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
| `custom-identifier`       | Custom identifier used in artifact naming for re-sign and ad-hoc flows to distinguish builds with the same native fingerprint     | No       | -           |

## Artifact Naming

The action uses two distinct naming strategies for uploads:

### ZIP Artifacts (native build caching)

ZIP artifacts store the native build for reuse. The naming depends on the flow:

- **Ad-hoc flow** (`ad-hoc: true`): ZIP name uses **fingerprint only** — `rock-android-{variant}-{fingerprint}`. One ZIP per fingerprint, shared across all builds with the same native code. Skipped if already uploaded.
- **Non-ad-hoc re-sign flow** (e.g. `pull_request` with `re-sign: true`): ZIP name includes an **identifier** — `rock-android-{variant}-{identifier}-{fingerprint}`. Used as the distribution mechanism without adhoc builds.
- **Regular builds** (no `re-sign`): ZIP name uses **fingerprint only** `rock-android-{variant}-{fingerprint}`

### Ad-Hoc Artifacts (distribution to testers)

When `ad-hoc: true`, distribution files (APK + `index.html`) are uploaded under a name that **always includes an identifier**: `rock-android-{variant}-{identifier}-{fingerprint}`. This ensures every uploaded adhoc build can point to unique distribution URL based on `{identifier}`, even when multiple builds share the same native fingerprint.

### Identifier Priority

The identifier distinguishes builds that share the same native fingerprint (e.g., concurrent builds from different branches).
It is resolved in this order:
1. `custom-identifier` input — explicit value provided by the caller (e.g., commit SHA of the head of the PR branch)
2. PR number — automatically extracted from `pull_request` events
3. Short commit SHA — 7-character fallback for push events and dispatches

> **Note:** The identifier becomes part of artifact names and S3 paths. Allowed characters: `a-z`, `A-Z`, `0-9`, `-`, `.`, `_`. Commas are used internally as trait delimiters and converted to hyphens (e.g., `debug,42` → `debug-42`), so they must not appear in the identifier. Spaces, slashes, and shell metacharacters are also not allowed.

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
