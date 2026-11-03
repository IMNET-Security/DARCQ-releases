# Verifying and opening unsigned DARCQ builds

*A copy of `docs/unsigned-builds.md` of the DARCQ repository at release 0.4.0; the public keys it names are in [`release-keys/`](release-keys/) here.*

Status: Draft 1.0 · Date: 2026-09-24 · Normative source: design-brief.en.md §4.9, §8 (signing row), §13 item 4; roadmap.md §14

## 1. Purpose and scope

This page is for people who download DARCQ. It explains how to check that a download is genuine, and how to open a build that macOS or Windows does not recognise because it carries no Apple or Microsoft signature.

- **Verification (§3) applies to every download**, signed or not, on every platform.
- **Opening an unsigned build (§4, §5)** applies to macOS and Windows releases without a platform signature: 0.1–0.4 unless signing is already in place, and any later release on a platform where the jurisdiction plan has switched to the unsigned path (roadmap.md §14). The release notes say, for each platform, whether its files carry a platform signature.
- **Linux (§6)** files never carry a platform signature.

Updates installed from inside the DARCQ app (from 0.4, on Windows, macOS and the AppImage) need no manual check: the app verifies each update's signature and the version it was signed for before installing it (platform-v1.md §18). The `.sig` files and `latest.json` on the release page are for the app; you do not need them. `SHA512SUMS` lists the `.sig` files but not `latest.json`, which each download site makes with its own addresses; the app checks every update's signature whatever `latest.json` says.

The commands and dialog paths on this page are checked from a quarantined download at the M2 rehearsal (the CLI on macOS 14, macOS 15 or later, Windows 10 and Windows 11) and at the 0.3 beta (the app), and corrected then (roadmap.md §14.4).

## 2. Why a build may be unsigned

Apple's and Microsoft's code-signing programmes are not available to every developer in every country. An unsigned build is not less secure because of that: what proves that a file comes from the DARCQ project is the project's own signature over the `SHA512SUMS` file, which does not depend on Apple, Microsoft or the download site. Verify it before you open anything. A warning from your operating system is expected for an unsigned build; a failed check in §3 is not, and means you MUST NOT open the file.

## 3. Verify a download (all platforms)

You need three files from the release page, or from the mirror, which holds identical files: the file you want (for example `DARCQ-X.Y.Z-x64-setup.exe`), `SHA512SUMS` and `SHA512SUMS.minisig`. Put them in one folder and open a terminal there. The steps below are also scripted in `ci/release/verify.sh` in the DARCQ repository, which runs them in this order and stops at the first failure.

**Step 1. Get the DARCQ public key once, and keep it.** Take `darcq-minisign.pub` from the `release-keys/` folder of the DARCQ repository. Its second line starts with `RW`. Compare that line with the key printed in `SECURITY.md` and in the release announcement; all three MUST be identical. Reuse the saved key for later releases. A new key is announced in a release signed with the old one (development.md §8.8); if the key has changed without such an announcement, stop.

**Step 2. Install minisign.** macOS: `brew install minisign`. Windows: `scoop install minisign` or `choco install minisign`. Linux: your distribution's `minisign` package.

**Step 3. Check the signature of `SHA512SUMS`.**

```sh
minisign -Vm SHA512SUMS -p darcq-minisign.pub
```

It MUST print `Signature and comment signature verified`. Any other output means stop.

**Step 4. Check the file against `SHA512SUMS`.**

```sh
sha512sum -c --ignore-missing SHA512SUMS               # Linux
shasum -a 512 -c --ignore-missing SHA512SUMS           # macOS
```

Your file MUST be listed, followed by `OK`. On Windows, in PowerShell, set `$f` to the file name and run:

```powershell
$f = 'DARCQ-X.Y.Z-x64-setup.exe'
$want = Select-String -Path SHA512SUMS -Pattern ('^([0-9a-f]{128}) [ *]' + [regex]::Escape($f) + '$') | ForEach-Object { $_.Matches[0].Groups[1].Value }
$got = (Get-FileHash -Algorithm SHA512 -LiteralPath $f).Hash
if ($want -and $got -eq $want) { 'OK' } else { 'MISMATCH: do not open this file' }
```

**Optional further checks.** The SLH-DSA post-quantum signature over `SHA512SUMS` (OpenSSL 3.5 or later; the key is `release-keys/darcq-slhdsa.pub.pem`) and the build provenance of a program inside the download (GitHub CLI):

```sh
openssl pkeyutl -verify -rawin -pubin -inkey darcq-slhdsa.pub.pem \
        -in SHA512SUMS -sigfile SHA512SUMS.slhdsa.sig
gh attestation verify ./darcq --repo <owner>/<repo>
```

OpenSSL MUST print `Signature Verified Successfully`.

**If any check fails:** delete the download, do not open it, download again from the other origin (GitHub or the mirror), and if it fails again, report it to the address in `SECURITY.md`.

## 4. macOS

**The app (DMG).** Verify the DMG first (§3). Open it and drag DARCQ to Applications. Open DARCQ once. macOS 15 and later say that Apple could not verify the app and offer no Open button: click **Done** (not **Move to Trash**). Then open **System Settings → Privacy & Security**, scroll to **Security**, find the line saying that DARCQ was blocked, and click **Open Anyway**. Confirm with your login password, and choose **Open Anyway** once more when DARCQ starts. The button appears only for a while after the blocked attempt; if it is missing, open DARCQ again first. On macOS 14 and earlier, Control-click DARCQ in Applications, choose **Open**, then **Open** again. You do this once for each version you download.

**The command-line tool.** If macOS blocks `darcq` when you first run it, use the same **Privacy & Security → Open Anyway** path, then run it again.

**The keychain.** If you let DARCQ unlock your key through the macOS keychain, an unsigned build asks for keychain permission again after every update: "Always Allow" does not carry over to a new version. This is expected (platform-v1.md §5.4). Enter your login password only in a macOS dialog that names `darcq` or DARCQ.

**Do not** turn Gatekeeper off for the whole Mac, and do not run commands found online that remove the quarantine mark from folders. If macOS says DARCQ "is damaged and can't be opened", do not work around it: download again, verify (§3), and report it if the message persists.

## 5. Windows

**Which installer.** From 0.4 there are two per architecture (`x64` for Intel and AMD processors, `arm64` for Arm): `DARCQ-X.Y.Z-x64-setup.exe` is small and downloads Microsoft's WebView2 runtime during the installation if your computer lacks it (Windows 11 and up-to-date Windows 10 have it); `DARCQ-X.Y.Z-x64-offline-setup.exe` carries that runtime inside (about 210 MB more) for a computer without internet access. Verify and open either one as below; both install the same app, which later updates itself with the small installer.

**The browser.** Your browser may say the file is not commonly downloaded. Keep it only in order to verify it (§3), and delete it if verification fails.

A test installer from the project's `test-installer.yml` run (DR-143) is not a release and has no `SHA512SUMS` entry: compare it with the run's own `SHA256SUMS.txt`, and use it only on a test machine.

**SmartScreen.** After verification, run the installer. If Windows shows "Windows protected your PC", click **More info**, check that the app name is exactly the file you verified and that the publisher is "Unknown publisher" (a signed build shows the name given in `SECURITY.md`), then click **Run anyway**. The same applies to `darcq.exe` from the command-line zip.

**Smart App Control (Windows 11).** When it is on, it blocks unsigned programs and offers no exception for one program; the unsigned build cannot run on that PC while it is on. Turning Smart App Control off lowers protection for every program, and on some Windows 11 versions it cannot be turned back on without reinstalling Windows. DARCQ does not ask you to turn it off; use a signed release when one exists.

## 6. Linux

Linux files carry no platform signature, and Linux shows no warning for them, so §3 is the only check: do it before you install. Then make the AppImage executable (`chmod +x DARCQ-X.Y.Z-x86_64.AppImage`), or install the verified package with your package manager: from 0.4 the app is `darcq-gui_X.Y.Z_amd64.deb` (`sudo apt install ./darcq-gui_X.Y.Z_amd64.deb`, which also installs WebKitGTK if it is missing) or `darcq-gui-X.Y.Z-1.x86_64.rpm` (`sudo dnf install ./darcq-gui-X.Y.Z-1.x86_64.rpm`), and the command-line tool is `darcq_X.Y.Z_amd64.deb` or `darcq-X.Y.Z-1.x86_64.rpm` (`arm64` and `aarch64` on Arm); the two packages are independent. The app's package also adds DARCQ to the menu and the file managers. The app never updates a deb or rpm install, and the AppImage updates itself only from inside the app: verify and install each new package the same way.
