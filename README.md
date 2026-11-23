# YouTube & YouTube Music Morphe (Root, no MicroG)

This repo holds a single GitHub Actions workflow that builds YouTube and YouTube Music with [Morphe](https://morphe.software) patches. The difference from a regular Morphe build: the `GmsCore support` patch is turned off, so the package name stays the same as the original and MicroG isn't needed at all. The APK gets installed straight over the stock app.

I got tired of re-patching on my phone every time Morphe pushed an update. Now I just let the workflow run, grab the APK from Releases and install it.

The app icon and name are left untouched too (`Custom branding` is disabled), so it looks like plain YouTube in the launcher.

## Requirements

- Root (KernelSU / Magisk / APatch)
- Signature and digest verification disabled, e.g. with CorePatch (LSPosed) or a similar module

No root? Then this isn't for you. Use the regular Morphe Manager with MicroG instead.

## How it works

Workflow file: `.github/workflows/morphe-root.yml`

1. Grabs the latest Morphe CLI (`MorpheApp/morphe-desktop`) and patches (`MorpheApp/morphe-patches`). Defaults to the `dev` channel.
2. Looks up the newest YouTube / YouTube Music version those patches support (`list-versions`).
3. Downloads the stock APK from APKPure using [apkeep](https://github.com/EFForg/apkeep). If the newest version isn't there yet, it falls back to the previous one.
4. Renames the Morphe entry in the app settings to **Morphe (Patch By Yudharn)**, then patches without `GmsCore support` and without `Custom branding`.
5. Puts the original Google signing block from the stock APK back onto the patched APK.
6. Publishes YouTube and YouTube Music together in one release.

Both apps build in parallel. If one of them fails or hasn't changed, the release carries over that app's APK from the previous release, so every release always has both.

It runs on its own every day at 19:17 UTC. If neither app nor the patches changed since the last release, nothing gets published, so Releases doesn't fill up with duplicates.

### About the signature

Because the APK still carries Google's certificate, Android and Play Services see it as the official app. Your Google account stays signed in across updates, the same way it does with other root builds that do this. The catch is that the signature no longer matches the content, which is why digest verification has to be off on the device. Without that the install just fails.

A note on "latest": it picks the newest version the **patches support**, not the newest one on the Play Store. If the Play Store already has something newer, you'll have to wait for Morphe to catch up.

## Setup

The build runs in one repo, and the APKs are published to [yudharn/YouTube-Morphe](https://github.com/yudharn/YouTube-Morphe). That way the release repo stays clean: just releases, no workflow noise.

**1. Release repo** (`yudharn/YouTube-Morphe`)

It needs at least one commit (a README is enough), because a release tag has to point at something. Nothing else goes in there.

**2. Token**

Create a fine-grained token at **GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens**:

- Repository access: *Only select repositories* → `YouTube-Morphe`
- Permissions: **Contents → Read and write** (Metadata read-only gets added automatically)
- Expiration: whatever you like, just remember to renew it before it runs out

A classic token with the `public_repo` scope (or `repo` if the release repo is private) works too.

**3. Builder repo** (the one holding `.github/workflows/morphe-root.yml`)

- **Settings → Secrets and variables → Actions → New repository secret**: name it `RELEASE_TOKEN`, paste the token
- Open the **Actions** tab, enable workflows if asked, then start a run with **Run workflow**

The first step checks that the token can reach the release repo and stops with a clear error if it can't. To publish somewhere else, change `RELEASE_REPO` at the top of the workflow.

## Manual run inputs

| Input | Default | Notes |
|---|---|---|
| `apps` | `all` | `all`, `youtube`, or `music` |
| `patches_channel` | `dev` | `dev` follows Morphe pre-releases, `stable` only stable releases |
| `youtube_version` | empty | Force a specific version, e.g. `21.46.522` |
| `music_version` | empty | Force a specific version, e.g. `9.46.51` |
| `youtube_apk_url` | empty | Direct link to a stock APK, for when APKPure acts up |
| `music_apk_url` | empty | Same, for YouTube Music (use the arm64-v8a APK) |
| `include_experimental` | `true` | Include versions the patches mark as experimental |
| `striplibs` | `arm64-v8a` | Native lib architectures to keep. Leave empty to keep all |
| `signature` | `original` | `original` keeps Google's signature, `custom` signs with your own keystore |
| `force_build` | `false` | Rebuild even if a release with the same versions already exists |

If you use one of the `*_apk_url` inputs, fill in the matching version too, so the file name and release tag come out right.

## Keystore (only for `signature: custom`)

You don't need this with the default `original` mode. With `custom` and no keystore, every build gets a random key, and your Google account will most likely disappear after each update.

If you want to use `custom`, set up a fixed key:

```bash
keytool -genkeypair -keystore morphe.keystore -storetype PKCS12 \
  -alias morphe -keyalg RSA -keysize 2048 -validity 36500 \
  -storepass YOURPASSWORD -dname "CN=Morphe"

base64 -w0 morphe.keystore
```

Then add these under **Settings → Secrets and variables → Actions**:

| Secret | Value |
|---|---|
| `KEYSTORE_BASE64` | the `base64` output above |
| `KEYSTORE_PASSWORD` | keystore password |
| `KEYSTORE_ALIAS` | `morphe` |
| `KEYSTORE_ENTRY_PASSWORD` | can be left out if it's the same as the keystore password |

## Installing & updating

Download the APK from Releases and install it over the existing app. Your Morphe settings carry over.

Before a big update, back up your settings from Morphe's import/export option in the app settings, just in case you end up having to uninstall.

## Google account missing

If you open the app and get an "Add account" screen with an empty account list, it's almost certainly a signature issue. Play Services only hands Google accounts to apps registered with Google's signature. This happens if the app on your phone was installed from a build signed with some other key, for example an older `custom` build.

Check it first:

```bash
su -c "dumpsys package com.google.android.youtube | grep -m1 signatures"
su -c "dumpsys package com.google.android.apps.youtube.music | grep -m1 signatures"
```

If the broken app shows a different signature than the working one, back up your Morphe settings, uninstall the app and install the `original` build from Releases. After that, updates keep your account.

## Release format

One release per update, holding both apps. Tag: `morphe-<patch version>-yt-<version>-ytm-<version>`, e.g. `morphe-v2.4.0-dev.9-yt-21.46.522-ytm-9.46.51`

Files: `YouTube-21.46.522-Morphe-2.4.0-dev.9-ROOT.apk` and `YouTube-Music-9.46.51-Morphe-2.4.0-dev.9-ROOT.apk`, each with a `.sha256` file for checking the hash.

If any patch fails to apply, it's listed in the release notes.

## Forking

Go ahead, but please keep the "Patch By Yudharn" credit in the settings name (`SETTINGS_TITLE` at the top of the workflow). If you change the patches or the build in a meaningful way, add your own name next to it instead of replacing it.

## License & credits

- Patches: [MorpheApp/morphe-patches](https://github.com/MorpheApp/morphe-patches), GPLv3
- CLI: [MorpheApp/morphe-desktop](https://github.com/MorpheApp/morphe-desktop)
- Downloader: [EFForg/apkeep](https://github.com/EFForg/apkeep)

This repo has nothing to do with the Morphe team or Google. If a patch is broken, report it to the Morphe repo, not here. The "Morphe" name and logo belong to the Morphe project.

Stock APKs come from APKPure, which is a third party. If you want to be safer, download the APK yourself from a source you trust, upload it to a release in your own repo and pass the link via `youtube_apk_url` / `music_apk_url`. Use at your own risk.
