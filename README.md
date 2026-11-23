# YouTube & YouTube Music Morphe (Root, no MicroG)

Patched YouTube and YouTube Music builds made with [Morphe](https://morphe.software) patches, for rooted phones. There's no code in this repo, just the [Releases](https://github.com/yudharn/YouTube-Morphe/releases).

How these builds differ from what Morphe Manager gives you by default:

- **No MicroG.** `GmsCore support` is off, so the package name stays the same as the stock app and it signs in through regular Google Play Services.
- **Original Google signature.** The signing block from the stock APK is put back on the patched one, so Android and Play Services see the official app. Your account stays signed in across updates, no re-adding it every time.
- **Original icon and name.** `Custom branding` is off. In the launcher it just looks like YouTube.
- **arm64-v8a only.** Other native libs are stripped to keep the APK smaller.

The Morphe settings show up as **Morphe (Patch By Yudharn)** in the app settings, so you can tell this build apart from an official one.

New builds come out on their own whenever Morphe releases new patches or a newer supported app version shows up. Each release has both apps.

## Requirements

- An arm64 phone with root (KernelSU, Magisk or APatch)
- Signature **and** digest verification disabled, e.g. with CorePatch (LSPosed) or a similar module

The digest check part matters. The APK carries Google's signature but the content is modified, so with digest verification still on, the install just fails.

No root? Then these builds aren't for you. Use Morphe Manager with MicroG instead.

## Installing

1. Grab the APK you want from the [latest release](https://github.com/yudharn/YouTube-Morphe/releases/latest).
2. Optionally, check the hash against the `.sha256` file next to it.
3. Install it over the stock app from the Play Store. Your existing data and account stay where they are.

Turn off auto-updates for YouTube and YouTube Music in the Play Store, or it will replace the patched app with the stock one sooner or later.

## Updating

Just install the newer APK over the old one. Your Morphe settings carry over.

Before a big jump in versions, it's still worth backing up your Morphe settings with the import/export option in the Morphe settings.

## Files in a release

| File | What it is |
|---|---|
| `YouTube-<version>-Morphe-<patches>-ROOT.apk` | YouTube |
| `YouTube-Music-<version>-Morphe-<patches>-ROOT.apk` | YouTube Music |
| `*.apk.sha256` | SHA-256 hash of each APK |

The release notes list the Morphe patch version used, and any patch that failed to apply.

## Troubleshooting

**"App not installed" / install fails**
Digest or signature verification is still on. Check your CorePatch (or similar) settings, then reboot.

**"Add account" screen with an empty account list**
Your current install came from a build signed with a different key, for example a MicroG build or one patched on the phone. Back up your Morphe settings, uninstall, then install the build from here. After that, updates keep your account.

**The app closes right after opening**
Grab a crash log and open an issue with it:

```bash
su -c "logcat -b crash -d" > /sdcard/crash.txt
```

**Why isn't this on the newest version from the Play Store?**
A build only uses the newest version the Morphe patches support. Sometimes there's also a short delay before a proper arm64 APK of a new version is available to build from.

## Credits

- [Morphe patches](https://github.com/MorpheApp/morphe-patches) and [Morphe CLI](https://github.com/MorpheApp/morphe-desktop) by the Morphe team
- Morphe itself builds on the earlier work of [ReVanced](https://github.com/ReVanced)

Please report bugs in the patches themselves to the Morphe repo. Problems specific to these builds (install issues, crashes that don't happen on official Morphe builds) can go in this repo's issues.

## License

The Morphe patches inside these APKs are licensed under the GNU GPLv3, see [LICENSE](LICENSE). [NOTICE](NOTICE) carries the extra terms Morphe asks every derivative build to keep. The source code for the patches is at [MorpheApp/morphe-patches](https://github.com/MorpheApp/morphe-patches).

YouTube and YouTube Music are Google's apps and are not covered by that license. This project has no connection to Google or the Morphe team. The "Morphe" name and logo belong to the Morphe project. Use at your own risk.
