# YouTube & YouTube Music Morphe (Root, no MicroG)

This repo holds a single GitHub Actions workflow that builds YouTube and YouTube Music with [Morphe](https://morphe.software) patches. The difference from a regular Morphe build: the `GmsCore support` patch is turned off, so the package name stays the same as the original and MicroG isn't needed at all. The APK gets installed straight over the stock app.

I got tired of re-patching on my phone every time Morphe pushed an update. Now I just let the workflow run, grab the APK from Releases and install it.

The app icon and name are left untouched too (`Custom branding` is disabled), so it looks like plain YouTube in the launcher.

## Requirements

- Root (KernelSU / Magisk / APatch)
- Signature verification disabled, e.g. with CorePatch (LSPosed) or a similar module
- The **keep previous signature** option enabled (in CorePatch it's called "Use previous signatures" / UsePreSig). This one matters, see [Google account missing](#google-account-missing)
- The stock app from the Play Store already installed and signed in before you install the patched one

No root? Then this isn't for you. Use the regular Morphe Manager with MicroG instead.

## How it works

Workflow file: `.github/workflows/morphe-root.yml`

1. Grabs the latest Morphe CLI (`MorpheApp/morphe-desktop`) and patches (`MorpheApp/morphe-patches`). Defaults to the `dev` channel.
2. Looks up the newest YouTube / YouTube Music version those patches support (`list-versions`).
3. Downloads the stock APK from APKPure using [apkeep](https://github.com/EFForg/apkeep). If the newest version isn't there yet, it falls back to the previous one.
4. Patches it without `GmsCore support` and without `Custom branding`.
5. Uploads the result to Releases.

YouTube and YouTube Music build in parallel. If one fails, the other keeps going.

It runs on its own every day at 19:17 UTC. If the app version and patch version are the same as the last release, the build is skipped, so Releases doesn't fill up with duplicates.

A note on "latest": it picks the newest version the **patches support**, not the newest one on the Play Store. If the Play Store already has something newer, you'll have to wait for Morphe to catch up.

## Setup

1. Fork or clone this repo.
2. Go to **Settings → Actions → General** and set *Workflow permissions* to **Read and write permissions**.
3. Open the **Actions** tab and enable workflows if GitHub asks.
4. Kick off a manual run with **Run workflow** to test it.

That's it. No secrets required.

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
| `force_build` | `false` | Rebuild even if a release with the same versions already exists |

If you use one of the `*_apk_url` inputs, fill in the matching version too, so the file name and release tag come out right.

## Keystore (optional)

Without a keystore, every build is signed with a random key. As long as signature verification is disabled on your phone, that's fine.

If you'd rather have a fixed key:

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

If you open the app and get an "Add account" screen with an empty account list, it's almost certainly a signature issue.

Play Services only hands Google accounts to apps registered with Google's signature. Disabling signature verification only lets the APK *install*. What keeps your account working is the keep-previous-signature option, because the system keeps the original Google signature on record.

Check it first:

```bash
su -c "dumpsys package com.google.android.youtube | grep -m1 signatures"
su -c "dumpsys package com.google.android.apps.youtube.music | grep -m1 signatures"
```

If the broken app shows a different signature than the working one, fix it like this:

1. Back up your Morphe settings, then uninstall the patched app.
2. Install the stock app from the Play Store and sign in.
3. Make sure UsePreSig (or whatever your module calls it) is enabled.
4. Install the patched APK on top.

## Release format

Tags: `youtube-<version>-morphe-<patch version>-root` and `music-<version>-morphe-<patch version>-root`

Files: e.g. `YouTube-21.46.522-Morphe-2.4.0-dev.9-ROOT.apk`, plus a `.sha256` file for checking the hash.

If any patch fails to apply, it's listed in the release notes.

## License & credits

- Patches: [MorpheApp/morphe-patches](https://github.com/MorpheApp/morphe-patches), GPLv3
- CLI: [MorpheApp/morphe-desktop](https://github.com/MorpheApp/morphe-desktop)
- Downloader: [EFForg/apkeep](https://github.com/EFForg/apkeep)

This repo has nothing to do with the Morphe team or Google. If a patch is broken, report it to the Morphe repo, not here. The "Morphe" name and logo belong to the Morphe project.

Stock APKs come from APKPure, which is a third party. If you want to be safer, download the APK yourself from a source you trust, upload it to a release in your own repo and pass the link via `youtube_apk_url` / `music_apk_url`. Use at your own risk.
