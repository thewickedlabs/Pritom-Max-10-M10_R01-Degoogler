# M10_R01 Degoogler

*Wicked Labs - cyberspace7.org, October 2026. For the Pritom Max 10 (M10_R01) with KernelSU Next.*

This module lets you choose how much Google to keep on the tablet. It is **systemless**: it hides
Google's preinstalled apps with the module and never writes to `/system` or `/product`. Disable the
module and reboot, and the hidden apps come back.

Unlike the Android Go phones, the M10_R01 ships the **full** Google stack (full GMS, the Play Store,
and the Velvet Google app with Search/Assistant), so this module targets those.

## Choosing during install

When you install the zip in the KSU Next app, a menu appears:

| Key | Action |
|---|---|
| **Volume Down** | move to the next choice |
| **Volume Up** | select the shown choice |
| no key for 30 s | **Cancel** (nothing is installed) |

The choices are **A**, **B**, **C** and **Cancel**. Choice B asks you to confirm with Volume Up
a second time. Your choice only takes effect at the **next reboot**. If you change your mind before
then, remove the module in KSU Next and nothing will have changed. (The one exception is B's backup,
which is made during install. See below.)

> **Note for this unit:** the tablet's physical volume keys are dead. Drive the menu over adb with
> `sendevent` on the `adc-keys` input device (`KEY_VOLUMEUP` = 115 to select, `KEY_VOLUMEDOWN` = 114
> to move), or repair the keys, before installing. On a tablet with working keys, just press them.

Every choice (A, B and C) removes three Google telemetry stubs:

- **Google Location History** (the Timeline stub)
- **Google Feedback** (crash and feedback uploads)
- **Mainline telemetry** (a telemetry marker)

These have no user-facing function. Nothing else on the tablet depends on them.

---

## A. Keep Google, trim telemetry, Morphe with root mount

**What it does:** removes the three telemetry stubs. Google Play services (GMS), the Play Store and
every Google app stay.

**Impact:**
- Everything keeps working: Google sign-in, Play Store, push notifications, Play Integrity (as far
  as an unlocked tablet gets it), Find My Device, Google backup.
- The RAM saving is small. Full GMS and the Play Store are still running.

**What you should do afterwards:**
1. Turn off Google's own tracking in Settings → Google:
   - **Ads** → *Delete advertising ID*.
   - **Usage & diagnostics** → off.
   - **Location** → *Google Location History / Timeline* → off.
2. For ad-free YouTube / YouTube Music, install **[Morphe Manager](https://github.com/MorpheApp/morphe-manager)**.
   When it asks, pick **Root Mount install**. The patched app is mounted over the real one, keeps its
   real package name and signs in with the real Google account. You don't need microG.
   Patches: [morphe-patches](https://github.com/MorpheApp/morphe-patches).

**Caveats:**
- Morphe patches target **specific app versions**. In the Play Store, open each patched app → ⋮ →
  untick **Enable auto update**. If you don't, a Play update replaces the app and the patch goes away
  until you patch again.
- This is the least de-Googled choice. Google still sees the device through GMS.

---

## B. Full microG: remove Google Play services

**What it does:**
- Removes the three telemetry stubs.
- Hides **Google Play services**, **Google Services Framework**, the **Play Store**, **Partner
  Setup**, **Device Health Services (Turbo)**, **ConfigUpdater**, **One Time Initializer**, **Google
  Restore** and the **Google app (Velvet, Search & Assistant)**. These depend on Play services and
  would only crash without it.
- **Makes a backup during install** (this is the only step that happens right away): the APKs and data
  of Play services, the Play Store, Services Framework, Partner Setup, Restore and the Google app,
  saved to `/data/adb/wickedlabs/degoogler/backup-<date>/`. On a full-GMS build this is large (often
  close to 1 GB); the exact path and result are printed at install. It stays there even if you remove
  the module. Delete it yourself once you're happy.
- On the **first boot** after install, it uninstalls the Play-updated copies of those apps from
  `/data/app` once. Otherwise they would linger as ordinary apps. The result is logged in
  `/data/adb/wickedlabs/degoogler/firstboot.log`.

**Impact (what you lose):**
- **Google sign-in, the Play Store, Google backup and restore, Find My Device, Assistant, Google
  Search and the home-screen search bar's Google results.** Your Google account is removed from the
  tablet.
- **Play Integrity.** With microG, banking, payment and Wallet apps that check integrity will
  refuse to run. Integrity modules (Play Integrity Fix and the like) only matter with real Play
  services.
- **Device Health Services (Turbo):** Adaptive Battery predictions.
- Apps that *require* Play services and aren't supported by microG may fail. microG covers most push
  notifications, Google sign-in, Maps API and location. It does not cover everything.

**What you gain:** several hundred MB of RAM back on this 3 GB tablet, and background wakeups and
network traffic to Google drop sharply.

**What you must do after the reboot (in this order):**
1. **Signature spoofing.** Stock Android 13 doesn't allow microG to pose as Google, so add it with
   an Xposed framework (a Zygisk module such as LSPosed, which needs Zygisk — this device already
   runs **ReZygisk**): install **[FakeGApps](https://github.com/whew-inc/FakeGApps)**, enable it in
   the framework manager, scope it to **System Framework**, then reboot. Background:
   [microG wiki: Signature Spoofing](https://github.com/microg/GmsCore/wiki/Signature-Spoofing).
2. **Install microG.** Get **[microG Services (GmsCore)](https://github.com/microg/GmsCore/releases)**
   and **microG Companion** (the Play Store stand-in) from the
   [microG releases](https://github.com/microg/GmsCore/releases). You can also get them through
   F-Droid by adding the repo `https://microg.org/fdroid/repo`. Project page:
   [github.com/microg/GmsCore](https://github.com/microg/GmsCore), docs in the
   [wiki](https://github.com/microg/GmsCore/wiki).
3. Open **microG Settings → Self-Check** and fix everything it flags. Grant every permission it asks
   for, and confirm that *signature spoofing* is ticked.
4. In microG Settings, turn on **Google device registration** and **Cloud Messaging** if you want
   push notifications. Sign in to a Google account there only if you want one.
5. **Location:** without Google, network location needs a provider. In microG's location settings,
   pick a network location service such as **[BeaconDB](https://beacondb.net)**.
6. **Apps:** install **[Aurora Store](https://auroraoss.com)**
   ([source](https://gitlab.com/AuroraOSS/AuroraStore)) for Play Store apps (anonymous login works)
   and **[F-Droid](https://f-droid.org)** for open-source apps.
7. **Patched YouTube / Music:** **[Morphe Manager](https://github.com/MorpheApp/morphe-manager)**
   works with real microG. Use a standard (non-root) install, or a root mount if you prefer.

**Caveats:**
- **Install microG only after the first reboot.** The first-boot cleanup uninstalls whatever is
  installed as `com.google.android.gms` at that moment.
- microG uses the same package name as Google Play services but a different signature. You can never
  have both installed at once.
- Apps you installed from the Play Store stay installed, but nothing updates them until you use
  Aurora Store.
- Google's own apps that stay (YouTube, Gmail, Maps, Drive and so on) run through microG, and some of
  their features may break.

**To go back to Google:**
1. Uninstall microG Services and microG Companion first (same package names as Google's, different
   signatures).
2. Disable or remove this module in KSU Next, then reboot. The *factory* versions of Play services
   and the Play Store come back, with empty data.
3. Open the Play Store, sign in again and let it update Google's apps. Your account data syncs back
   from Google. Local Play-services data does not.
4. The backup in `/data/adb/wickedlabs/degoogler/backup-<date>/` has the newer APKs. As root, you can
   reinstall a package from it with
   `pm install-multiple -r /data/adb/wickedlabs/degoogler/backup-<date>/<package>/*.apk`.
   The `data.tgz` / `user_de.tgz` archives are a last resort, for manual recovery only. Restoring app
   data by hand needs the right owner and SELinux labels.

---

## C. Keep Google, trim telemetry, Morphe without root (MicroG-RE)

**What it does on the tablet:** exactly the same as **A**. It removes the three telemetry stubs, and
Google stays. The difference is how you run patched apps.

**Impact:** the same as A. On top of that, Morphe apps run on their **own** copy of microG next to
real Play services. Two Play-services stacks on 3 GB of RAM is the worst choice for performance. It's
here because it doesn't depend on root mounts or on blocking Play updates.

**What you should do afterwards:**
1. The same Google privacy switches as A (delete the ad ID, usage & diagnostics off, Location
   History off).
2. Install **[MicroG-RE](https://github.com/MorpheApp/MicroG-RE)**. It's Morphe's microG fork, with
   its own package name, so it runs next to Google Play services without signature spoofing.
3. Install **[Morphe Manager](https://github.com/MorpheApp/morphe-manager)** and pick a **Standard
   install**. The patched apps get new package names and sign in through MicroG-RE.

**Caveats:**
- Patched apps are separate from the stock YouTube / Music apps. Disable the stock ones if you don't
  want both.
- This doesn't de-Google anything. It only makes the patched apps independent of root and of Play
  updates.

---

## Which one?

- **Do you need banking, payment or Wallet apps, or anything that checks Play Integrity?** Pick
  **A**.
- **Don't need any of that, and want the tablet fast and Google-free?** Pick **B**.
- **C** only if you want patched apps that survive without root.

## Undo, any choice

In KSU Next, toggle the module **off** (or remove it) and reboot. Everything it hid comes back. For
choice B, follow "To go back to Google" above.

## Links

- microG: <https://github.com/microg/GmsCore> · [releases](https://github.com/microg/GmsCore/releases)
  · [wiki](https://github.com/microg/GmsCore/wiki) · F-Droid repo `https://microg.org/fdroid/repo`
- FakeGApps (signature spoofing via an Xposed framework): <https://github.com/whew-inc/FakeGApps>
- Morphe: [Manager](https://github.com/MorpheApp/morphe-manager) ·
  [Patches](https://github.com/MorpheApp/morphe-patches) ·
  [MicroG-RE](https://github.com/MorpheApp/MicroG-RE)
- Aurora Store: <https://auroraoss.com> · [source](https://gitlab.com/AuroraOSS/AuroraStore)
- F-Droid: <https://f-droid.org>
- BeaconDB (network location for microG): <https://beacondb.net>

---

## Install (from the release)

1. Download [`Pritom-Max-10-M10_R01-Degoogler.zip`](https://github.com/thewickedlabs/Pritom-Max-10-M10_R01-Degoogler/releases/latest/download/Pritom-Max-10-M10_R01-Degoogler.zip) (latest release).
2. KernelSU Next → Modules → Install from storage → select the zip.
3. Make your A/B/C choice with the volume keys, then reboot.

## License

[WTFPL](LICENSE). Use at your own risk; no warranty and no liability for anything this does to your device.

Wicked Labs · https://www.cyberspace7.org
