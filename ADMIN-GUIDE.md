*A copy of `docs/admin-guide.md` of the DARCQ repository at release 0.4.0.*

# DARCQ Administrator Guide

Status: Guide · Scope: deploying DARCQ in an organisation and managing it through the operating system's own policy channel (DR-142) · Normative sources: platform-v1.md §4.5, cli-spec.md §8 rule 11 and §10.6, gui-spec.md D83, errors.md 1124

## 1. What an organisation controls

DARCQ has no server, console or account. An organisation manages it the way it manages a browser: through Group Policy or Intune on Windows, a configuration profile on macOS, and a file under `/etc` on Linux. Two policies exist:

| Policy | What it does |
|---|---|
| `RecoveryRecipients` | One to four recipient strings (`darcq1…`) of keys the organisation holds. Every file DARCQ encrypts on a managed computer is also encrypted to each of them, so the organisation can open the file after an employee leaves or loses their recovery kit. |
| `DisableUpdateCheck` | The DARCQ app makes no update check, at launch or on request. New versions come from your software distribution. |

The recovery recipients are ordinary post-quantum hybrid recipients (ML-KEM-768 with X25519). The policy changes no algorithm and no file format, and it never copies a person's key: the organisation recovers files, not keys.

Users see the policy wherever recipients appear: an "Organization recovery" chip on Home, a note in the command line, and `darcq doctor` or Health. DARCQ refuses to remove a recovery recipient from a file.

What the policy does not do:

- It is not a data-loss-prevention control. It prevents mistakes, not a user who sets out to avoid it: such a user can encrypt with another tool, with a DARCQ they built themselves, or in a private mount namespace without `/etc/darcq`.
- Files written before the policy, on another computer or by someone outside the organisation have no recovery stanza until they are re-encrypted (§7).
- Whoever holds a recovery key's 24 words can read every file written under the policy. Treat that key like a master key, and tell your users that the organisation can open their DARCQ files.

## 2. Before you deploy

- **Pre-release.** DARCQ 0.4 is a pre-release. Files it writes will not be readable by DARCQ 1.0 (DR-29), and the independent audit that gates 1.0 has not taken place. Pilot DARCQ with test data first.
- **Signing.** Until the project has code-signing certificates, the Windows installers are unsigned and the macOS app is signed ad hoc. App-control rules that allow by publisher (WDAC, AppLocker, Santa) and Gatekeeper treat it accordingly; allow it by file hash, or wait for signed releases.
- **FIPS.** DARCQ is not FIPS 140-3 validated, and it will not be (brief §1.4).

## 3. Create the recovery key

Create each recovery key on a computer that is offline and stays offline, such as a laptop set aside for this purpose. Do not create it on a computer you use every day.

1. Install the DARCQ command-line tool on the offline computer.
2. Create the key and verify its kit:

   ```
   darcq keys init --label "Example Corp recovery"
   ```

   Write the 24 words and the fingerprint on the paper kit as DARCQ shows them, then complete the three-word check.
3. Print the recipient string and the fingerprint:

   ```
   darcq keys show
   ```

   Copy the recipient string (`darcq1…`, about 1,970 characters) to removable media. It is public; the policy needs it.
4. Keep the kit in a safe under two-person control. A second recovery key, kept by other custodians, protects against the loss of the first; the policy takes up to four.
5. Keep the identity file only on the offline computer, or delete it: the 24 words recreate it.

## 4. Deploy DARCQ

| OS | Package | Notes |
|---|---|---|
| Windows | `DARCQ-<ver>-<x64\|arm64>-setup.exe` (per-user NSIS installer), or the `-offline-setup.exe` variant with the WebView2 runtime | Install silently with `/S`. As an Intune Win32 app, set the install behaviour to *User*. A per-machine installer is planned once the installers are signed (DR-142 (h)). The CLI is `darcq-cli-<ver>-<arch>-pc-windows-msvc.zip`; copy `darcq.exe` to a folder on the `PATH`. |
| macOS | `DARCQ-<ver>-universal.dmg` with the app; the universal CLI binary | Copy `DARCQ.app` to `/Applications` with your MDM. A signed installer package is planned (DR-142 (h)). |
| Linux | `darcq-gui` and `darcq` deb or rpm packages; the GUI AppImage | Install the packages from your repository or with configuration management. |

When you set `DisableUpdateCheck`, update DARCQ the same way you installed it.

## 5. Set the policy

### 5.1 Windows

DARCQ reads `HKLM\SOFTWARE\Policies\DARCQ` only, in the 64-bit view.

- **Group Policy.** Copy `packaging/policy/windows/DARCQ.admx` and `en-US/DARCQ.adml` into `%SystemRoot%\PolicyDefinitions`, or into the domain's Central Store (`\\<domain>\SYSVOL\<domain>\Policies\PolicyDefinitions`). The two policies appear under *Computer Configuration → Administrative Templates → DARCQ*. For "Organization recovery recipients", choose *Show…* and enter one recipient string per row.
- **Intune.** Import `DARCQ.admx` and `DARCQ.adml` under *Devices → Configuration → Import ADMX*, then create a configuration profile from the imported template and assign it to devices.
- **Registry**, for a test or a script run as an administrator:

  ```
  reg add HKLM\SOFTWARE\Policies\DARCQ\RecoveryRecipients /v 1 /t REG_SZ /d darcq1…
  reg add HKLM\SOFTWARE\Policies\DARCQ /v DisableUpdateCheck /t REG_DWORD /d 1
  ```

  Further recipients are the values `2`, `3` and `4`. The subkey must hold exactly the values `1` to `n`: a gap or any other value stops encryption until it is fixed (§6).

### 5.2 macOS

DARCQ reads the device-scope managed preferences of the domain `org.darcq.DARCQ`: the file `/Library/Managed Preferences/org.darcq.DARCQ.plist`, which macOS writes from a configuration profile of device scope. A user-scope profile and a value a user writes with `defaults` are ignored, because a user may be able to install them.

Start from `packaging/policy/macos/org.darcq.DARCQ.mobileconfig`. Replace the recipient string, remove `DisableUpdateCheck` if users may check for updates, and give both `PayloadUUID` keys new values (`uuidgen`) and both `PayloadIdentifier` keys your organisation's prefix. Keep `PayloadScope` `System`. Deploy it with your MDM to the device channel, for example as a custom settings payload for the preference domain `org.darcq.DARCQ`. The keys are `RecoveryRecipients` (an array of strings) and `DisableUpdateCheck` (a boolean). After it is installed, `/Library/Managed Preferences/org.darcq.DARCQ.plist` holds them.

### 5.3 Linux

DARCQ reads `/etc/darcq/policy.toml`. It must be a regular file, not a symbolic link, owned by root and writable by no one else, in a directory `/etc/darcq` of the same kind; otherwise DARCQ treats the policy as broken (§6). Start from `packaging/policy/linux/policy.toml`:

```
install -d -m 0755 -o root -g root /etc/darcq
install -m 0644 -o root -g root policy.toml /etc/darcq/.policy.toml.new
mv /etc/darcq/.policy.toml.new /etc/darcq/policy.toml
```

Replace the file by a rename, as the last two lines do, whenever you change it: a run that finds it missing or half written would encrypt without the policy, or not at all.

On NixOS, give the `environment.etc` entry a `mode` such as `"0644"`, so that the file is copied rather than linked into the store.

## 6. Check the policy

On a managed computer, `darcq doctor` reports a `managed_policy` line, and Health in the app shows the same:

```
info  managed_policy  managed by your organization through /etc/darcq/policy.toml: recovery recipient dqfp1…; update checks off
```

Encrypt a test file and run `darcq inspect` on it: the header lists a KEM stanza for each recovery recipient (hidden key ids are zero).

A policy that DARCQ cannot apply exactly (a damaged recipient string, more than four, a value of the wrong type, a Windows list with a gap, a Linux or macOS file with the wrong owner or mode) stops every encryption with `E_ORG_POLICY` (1124) until it is fixed, so that no file is written that the organisation could not recover. Decrypting is never affected. `darcq doctor` names the key at fault; the log has the same line.

## 7. Recover a file

1. On the offline computer, restore the recovery key from its kit, if its identity file is not there:

   ```
   darcq keys import-words
   ```

   and compare every group of the fingerprint with the one on the kit.
2. Decrypt the file:

   ```
   darcq decrypt report.pdf.darcq
   ```

3. Hand the plaintext over by your organisation's secure means, or re-encrypt it there for its new owner with `darcq encrypt -r <their recipient>`.

Never import a recovery key on a computer that is online.

## 8. Rotate, add or retire a recovery key

- **Add a key.** Add its recipient string to the policy. New files get it; existing files keep their stanzas. To give existing files the new key, rewrap them (`darcq rewrap --add <recipient> FILE`) or re-encrypt them (`darcq rekey FILE`).
- **Replace a key.** Put the new recipient string in the policy in place of the old one. Keep the old kit for as long as files written under it exist; only re-encrypted files no longer need it.
- **Bring older files under the policy.** `darcq rekey` writes a file with a new file key for the recipients you choose plus the recovery recipients. `darcq rewrap --add` adds a recipient without re-encrypting.

## 9. Troubleshooting

| Symptom | Cause | Action |
|---|---|---|
| `E_ORG_POLICY` (1124) when encrypting | The policy cannot be applied | Run `darcq doctor`; fix the key it names |
| `E_ORG_POLICY` with "must stay a recipient" | `rewrap --remove` of a recovery recipient | None: the policy forbids it |
| `managed_policy` says "not managed" | No policy reached this computer, or it was written to a per-user place (`HKCU`, a user-scope profile, the user's defaults) | Deploy the policy at machine level (§5) |
| The Linux file is reported as unreadable | Owner, mode or type of `/etc/darcq` or the file | `ls -ld /etc/darcq /etc/darcq/policy.toml`; both owned by root, mode 0755 / 0644 |

## 10. Uninstall

An uninstall removes DARCQ and its traces; your users' keys, contacts and settings stay unless you or they ask for their removal (DR-144, platform-v1.md §16.1). Before you remove them, make sure that every person keeps their 24-word kits, or that your recovery recipients cover their files: a file encrypted only to a deleted key cannot be opened again without its kit. Encrypted files are never deleted, and your policy stays until you remove it with Group Policy, Intune, your MDM or your configuration management.

### 10.1 Windows

- **By hand:** Settings → Apps → DARCQ → Uninstall. The confirm page's "Delete the application data" checkbox also removes the person's DARCQ keys, contacts and settings, and DARCQ's device keys in Credential Manager, after a question whose default is No.
- **Silently, keeping the data:** `"%LOCALAPPDATA%\DARCQ\uninstall.exe" /S`. This is the uninstall command for an Intune Win32 app (install behaviour *User*).
- **Silently, removing the data too:** `"%LOCALAPPDATA%\DARCQ\uninstall.exe" /S /REMOVEDATA`.

The uninstaller always removes the program, its shortcuts and Explorer entries, the WebView2 folder `%LOCALAPPDATA%\org.darcq.DARCQ` and its own registry records. When some data cannot be removed, for example because a DARCQ window is still open, it says so, unless it runs silently.

### 10.2 macOS

1. To remove the data, as the user, before removing the app:

   ```
   /Applications/DARCQ.app/Contents/MacOS/darcq-gui --remove-data
   ```

   It removes the keys, contacts and settings in `~/Library/Application Support/DARCQ`, the DARCQ device keys in the login keychain and the Quick Action, and exits 0 when everything went.
2. Quit DARCQ and move `DARCQ.app` to the Trash.
3. Optionally remove the app's own preferences: `defaults delete org.darcq.DARCQ`.

### 10.3 Linux

1. To remove the data, as the user: `darcq-gui --remove-data` (with the AppImage: `DARCQ-<ver>-<arch>.AppImage --remove-data`). It removes `~/.local/share/darcq` (or `$XDG_DATA_HOME/darcq`), the DARCQ items of the Secret Service and the per-user desktop entries.
2. Remove the packages, `sudo apt remove darcq-gui darcq` or `sudo dnf remove darcq-gui darcq`, or delete the AppImage.

With only the command-line tool installed, remove each identity's keychain unlock with `darcq keys protectors remove os-keychain`, then delete the data directory.

