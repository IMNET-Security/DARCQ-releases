# Changelog

All notable changes to DARCQ are recorded in this file. The format follows
[Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/), and versions follow
[Semantic Versioning](https://semver.org/spec/v2.0.0.html) as described in
docs/development.md §8.1.

Every 0.x release note opens with: "files written by this version will not be readable by
DARCQ 1.0; keep this build until you have re-encrypted them" (DR-29).

## 0.4.0 (unreleased)

The platform release of the desktop app (milestone M5). Its release notes, which open with the DR-29
sentence above, are docs/release-notes/0.4.0.md. Decisions DR-126 to
DR-147; DR-125 is closed on macOS.

### Added

- A complete uninstall (DR-144). The Windows uninstaller removes the program with its
  shortcuts, Explorer entries, the WebView2 folder and its registry records, and keeps your keys,
  contacts and settings unless you tick "Delete the application data" and confirm; an organisation
  can pass `/S /REMOVEDATA`. `darcq-gui --remove-data` removes the same data on every system,
  device keys in the OS secret store included. Encrypted files are never touched.
- Managed deployment (DR-142): an organisation sets DARCQ's policy through each operating
  system's own channel, which only an administrator can write: Group Policy or Intune
  (`HKLM\SOFTWARE\Policies\DARCQ`, with an ADMX template), a device-scope configuration profile
  for `org.darcq.DARCQ` on macOS, or a root-owned `/etc/darcq/policy.toml` on Linux. Its recovery
  recipients get a stanza in every file DARCQ writes there and cannot be removed from one; its
  `DisableUpdateCheck` turns the app's update check off. A policy that cannot be applied stops
  every encryption (`E_ORG_POLICY`, 1124) until it is fixed, and decrypting is never affected.
  Home shows an "Organization recovery" chip, the CLI a note and `kind: "recovery"` in `--json`,
  and `doctor` and Health report the policy. docs/admin-guide.md covers
  deployment, the recovery key's offline custody and recovery.
- Shell integration under the `org.darcq` identifiers (DR-64): "Encrypt with DARCQ" on a file, a
  multi-selection and a folder in Windows Explorer, as a Quick Action in the macOS Finder, and in
  Nautilus and Dolphin on Linux; `.darcq` files open in DARCQ. The Encrypt verbs stage their files and
  never decrypt a `.darcq` among them. The NSIS installer writes and removes the Windows entries, the
  deb and rpm install theirs system-wide, and the app writes the macOS and Linux per-user entries.
  `doctor` reports each entry present, absent or stale on both surfaces, and Health offers Install,
  Repair and Remove (DR-127 (d)).
- Updates in the app, only when the user asks or turns on the check at start, through
  `tauri-plugin-updater` 2.12 driven from Rust: an update installs only after its minisign signature
  has been verified against the pinned key, with the signed version equal to the announced, newer
  one and the signed file name this build's artefact; HTTPS only, a 30 s read timeout and a 512 MiB
  cap; nothing installs while a job runs, and "Restart now" never passes a launch file again. Only the
  release job's builds update themselves; every other build answers that it is updated from where it
  was installed (DR-127 (c), (i)). The rollback test runs vectors (a) to (i) over a local HTTPS server
  with a TEST-ONLY key and never sets a `dangerous*` option (DR-127 (b)).
- GUI deb and rpm packages (`darcq-gui`), and offline Windows installers that carry the WebView2
  runtime (DR-127 (e), (f)).
- The updater's release files: the macOS `.app.tar.gz`, the `.sig` files and one `latest.json` per
  origin, with `verify.sh -l` to check them, and a nightly run of the whole signing ceremony with
  TEST-ONLY keys (DR-127 (h), (j)).

### Changed

- Releases are published in the public `IMNET-Security/DARCQ-releases` repository, which holds the
  installers, signatures and `latest.json` and no code; the app's updater reads `latest.json` from
  there, and from a second route on `raw.githubusercontent.com` until a mirror exists (DR-147).
- The workspace is version 0.4.0: files are written with `version_minor = 4`, and the version-0 KAT
  set is regenerated.
- The desktop app's look (gui-spec.md §5.2): a sidebar with the product mark, the navigation in
  three groups with icons, and the session card with "Lock now"; notices stack behind a "more"
  button that opens them; every screen is a title row over cards on one token set (15 px base,
  one accent, soft status tints, light and dark). Home names its mode, counts and sizes the staged
  files, draws recipients as pills with the unverified contact's fingerprint in a callout under
  them, keeps the mixing warning with the chips it concerns, and ends in an action bar that says
  what the batch does and why its button is disabled. Jobs shows a running, done and failed
  summary and status pills instead of coloured rows; Keys & Recovery is task cards; Contacts has
  status pills and switches; Settings is grouped cards; the wizard has a stepper and explains the
  recovery kit before it opens; the passphrase sheet has an eyebrow, a heading and a four-segment
  meter; the recovery kit numbers its words down three columns and offers its actions as tiles.
  Headings no longer draw a focus ring when a screen moves the focus to them.
- The GUI executable is reproducible: `build.rs` generates the Tauri context with fixed isolation
  UUIDs and sorted asset lists, and the release's executable comparison now blocks like the `dist/`
  comparison (DR-126).
- On macOS, "Print" in the recovery-kit window opens the print panel without a page preview of the
  kit (DR-125, DR-127 (g)); a print that fails to start shows an error.
- On macOS DARCQ no longer uses the single-instance plugin, whose socket sat at a fixed path in the
  shared `/tmp`; Launch Services keeps the app single-instance there (gui-spec.md D82).
- The persistent log keeps only DARCQ's own records.
- Supply chain: the updater's graph is vetted (671 packages), every exemption note was recomputed per
  release target, `cargo audit bin` fails on an executable without cargo-auditable data, and Node.js
  comes from SHA-256-pinned archives (DR-128).

### Fixed

- In the app, an error whose detail names a key, such as a retired identity chosen as "Me",
  showed the name and fingerprint empty (" () is retired"); the detail now gets the error's own
  parameters.
- The recovery-kit window keeps "Close and verify", "Verify" and "Import" at its bottom edge
  while the kit scrolls; the 24 words with the two actions were taller than the window, so the
  button to go on was out of view (gui-spec.md D85).
- On macOS, files opened through the Finder or the Quick Action while DARCQ was closed were dropped;
  on Windows, a cold start with many selected files could drop some of them.
- On Windows, a folder on a drive without a Cloud Files sync root (a computer without OneDrive, a
  second disk, a USB drive) was taken for a cloud folder, so every decryption there asked first
  (DR-129).
- On Windows, an output redirected with `>>` onto the input file itself was not refused (DR-99,
  DR-129).
- In the app, cancelling a job that waits for the passphrase window could, rarely, close the next
  job's window (DR-129).
- A damaged or crafted `config.toml` with thousands of nested keys could overflow the stack at
  start; such a file now counts as unreadable, and every setting takes its default (DR-129).
- A passphrase typed with a no-break space or another non-ASCII space (Option+Space on several macOS
  layouts, a paste from a password manager) now opens like the same passphrase typed with plain
  spaces: the RFC 8265 OpaqueString mapping the specification cites is applied before NFC
  (DR-130 (c)).
- A foreign identity file whose first passphrase record is out of policy now unlocks through the
  next one, and reports the policy error only when no record opens (identity-v1.md §6.6 rule 4).
- A rename or link that a network file system reports as failed although it took place now counts
  as the placement it was: the job no longer discards, and so empties, an output already in place,
  which for an in-place rewrap or rekey was the user's file, nor leaves a second name for it (DR-132).
- `inspect` no longer reports header room for a header whose length is not a multiple of 4,096,
  which a rewrap re-pads, moving the payload; no DARCQ writer produces one (DR-131 (b)).

### Security

- A passphrase of one or two words from the EFF list (the generator's list) is now refused as
  weak; zxcvbn alone had rated such passphrases strong enough (DR-134, audit AA-1).
- Retiring a key now asks an operating-system confirmation with its fingerprint first, as rotating
  and "Use for 'Me'" do (DR-134, audit AA-2).
- A crafted QR image can no longer hold DARCQ busy: images are scaled down before decoding, and a
  decode is abandoned after 10 seconds (DR-134, audit AA-3).
- ML-KEM now runs libcrux's AVX2 code only where the operating system has enabled it, and its
  portable code everywhere else, whatever a stale build directory holds (DR-135, audit AA-4, AA-5).
- A file whose name has an extension of about 250 bytes no longer fails to decrypt next to a file
  of the same name (DR-135, audit AA-6).
- `rewrap --hide-recipients` and `--remove` now also act on a recipient stanza of a suite this
  version cannot use, whose key id is in the same place, and `inspect` shows that key id
  (DR-136, audit AA-7, AA-14).
- A new passphrase or label may no longer contain a character that Unicode 17.0 leaves
  unassigned: a later Unicode version could change how it is normalised, and a file protected by
  that passphrase could then no longer be opened (DR-137, audit AA-8).
- The crypto path has no `unsafe` code: the ML-KEM key wipe goes through libcrux's own byte
  access (DR-140, audit AA-11).
- On macOS, Quit (⌘Q) no longer bypasses the update's exit hold or the "Quit before your kit is
  verified?" question (DR-140, audit AA-18).
- An update manifest without per-platform entries is refused (DR-140, audit AA-17).
- A passphrase of EFF words joined without spaces is scored by its words (DR-140, review R-10),
  and the passphrase window's idle close counts only real input (DR-140, review R-12).
- A damaged identity-file label is reported as damaged, not as needing a newer DARCQ (DR-140,
  audit AA-12).
- Fixes from the adversarial review of 2026-10-06 (docs/reviews/adversarial-review-2026-10-06.md):
  a repeated EFF word no longer counts as a new word in a passphrase; a crashed QR decode
  no longer blocks every later QR import, an abandoned one no longer delays them, and the
  decoded image is bounded before decoding; the release preflight requires every workflow's
  run on the tagged commit.
- The release keys exist: minisign key ID `AF5879C2A469F94D` and an SLH-DSA-SHAKE-256s key with SHA-256
  `d9623e2d7de30fd8…`, published in `SECURITY.md` and `release-keys/`; the app's updater checks
  updates against the minisign key. They are interim 0.x keys and are replaced before 1.0
  (DR-133).
- An internal security review of the audit scope found no Critical, High or Medium finding
  (docs/reviews/internal-security-review-2026-10.md);
  it feeds the external audit and does not replace it (DR-130).
- `keys repair` derives at most eight passphrase records: a crafted identity file could make it run
  hundreds of key derivations at the cap of 1 GiB and 8 passes (DR-130 (a)).
- `keys change-passphrase` also replaces a passphrase record of a key-derivation algorithm this build
  cannot use, so none stays sealed under the old passphrase (DR-130 (b)).
- On macOS an update installs only when this user owns and can write the app bundle itself: a
  bundle an administrator or another user placed, also in a sticky folder such as `/Users/Shared`,
  could otherwise bring up an administrator prompt and end up owned by this user (DR-130 (e)).
- In the app, only a folder chosen in the folder picker becomes the default decrypt folder, as
  gui-spec.md §6.9 requires; a dropped folder or one named at launch no longer can.
- On Linux the core-dump limit is 1 byte instead of 0: on Ubuntu, apport ignores a limit of 0 and
  stored the memory of DARCQ's crashed webview process in `/var/crash`, where a passphrase or the
  recovery words could have been; at 1 the kernel aborts the dump (DR-131 (a)).
- The follow-up verification of the AI-agent audit checked its 18 unverified candidates: 15 were
  confirmed (3 Low, 12 Informational) and 3 refuted
  (docs/reviews/agent-audit-2026-10.md, DR-145, DR-146):
  - Changing who can open a file, or re-encrypting it, now says in the operating-system
    confirmation what changes: the recipients added, removed or set, a file passphrase added or
    removed, the recipients hidden or shown, and where the result is saved (AA-20).
  - A screen lock or "Lock now" while a key passphrase is being processed cancels the operation:
    the recovery words are no longer shown afterwards, and nothing derived is kept (AA-21); "Use
    for 'Me'" and retire are cancelled the same way when the lock comes while their confirmation
    is open (AA-25).
  - A contact whose key has the same 32-bit key id as another contact's or as one of your
    identities is refused (`E_USAGE`, `same_key_id`), so removing one recipient from a file can no
    longer remove the other with it (AA-22).
  - A folder chosen as the default decrypt folder is no longer accepted as a job's input (AA-23).
  - Hints for a file that may have been tampered with now say so (AA-27), and the prompts and
    hints that ask you to compare a fingerprint ask for every group of it (AA-33).
  - The command-line tool no longer prints a panic's message: the log gets its location, and
    standard error a fixed line (AA-34).
  - The documents now state the residue of the primitive crates, the constant-time gate's
    coverage, the worst Argon2 cost of an identity file, and the isolation allowlist's role as
    they are (AA-24, AA-26, AA-28 to AA-32).

## 0.3.0 (unreleased)

The first beta of the desktop app (milestone M4). Its release notes, which open with the DR-29
sentence above, are docs/release-notes/0.3.0.md. Decisions DR-119 to
DR-125.

### Added

- The DARCQ desktop app (`darcq-gui`, Tauri 2.11): the ten screens, the first-launch choice, the setup
  wizard with the two kit actions and hide-before-verify, the passphrase sheet with the identity
  chooser, the recovery-kit window (show, verify, import), contacts with QR import and quarantine,
  file actions (rewrap, rekey), Settings and Health, in English and Persian (right to left). The
  webview is untrusted: paths arrive only as grants from drops and native pickers, recipients other
  than self are contacts only, secrets cross only the four enumerated ways (raw bodies over Tauri's IPC
  protocol, DR-122), every window is incognito and navigation-locked, the passphrase and kit windows
  are capture-excluded where the OS allows, and every trust-changing step is confirmed in an
  OS-rendered dialog (DR-65, DR-124).
- `darcq-app`, the application layer the CLI and the GUI share: one implementation of the store, the
  identity orders, the silent keychain unlock, the sync-root and folder rules, the per-input jobs and
  the doctor report (DR-119).
- Lock and suspend purging in `darcq-platform` (`notifications`, GUI only): logind on Linux with a
  sleep-delay inhibitor, NSWorkspace and the screen-lock notification on macOS, session and suspend
  notifications on Windows.
- A Persian user-input list for the passphrase strength check on both surfaces (DR-121), and a guide
  for copying the recovery words by hand, in Persian and English (`docs/user/kit-transliteration.md`).
- GUI installers: a per-user NSIS installer for Windows, a universal DMG for macOS and AppImages for
  Linux, with pinned and verified bundling tools (DR-123, DR-124).

### Changed

- The workspace is version 0.3.0: files are written with `version_minor = 3` (DR-124).
- The hints of `E_NO_PASSPHRASE_STANZA` (1103, skipped) and `W_PASSPHRASE_WEAKEST_LINK` (9004) name the file
  passphrase.
- Supply chain: the GUI graph is vetted by publisher trust and annotated exemptions; the
  `serde_json/unbounded_depth` ban is checked on the runtime graph (DR-123).

## 0.2.0 (unreleased)

The second pre-release (milestone M3): CLI 0.2 with the complete `darcq-platform`. Its release
notes, which open with the DR-29 sentence above, are
docs/release-notes/0.2.0.md. Decisions DR-100 to DR-118.

### Added

- `rewrap` (header rewrite: add, remove or replace recipients and the file passphrase, hide key
  ids) and `rekey` (re-encryption under a new key), in `darcq-core` and the CLI, with the
  reference-generated rewrap KATs for both writer versions (DR-107, DR-112), the rewrite lock and
  the lock-out guard for in-place jobs (DR-115, DR-116).
- The OS-keychain protector on macOS, Linux and Windows: silent unlock, `keys init
  --remember-on-device`, `keys protectors list|add|remove`, the lost-key fallback and re-enable
  (DR-101, DR-111, DR-113).
- `keys retire`, `rotate`, `repair`, `relabel` and `reset-passphrase`; `contacts
  add|list|remove|verify|rebuild|rebind` and `encrypt -c`; `doctor` (DR-108); `completions`.
- `darcq-platform` 0.2: the job journal and startup sweep (DR-103, DR-104), the locked allocator
  (DR-100), the system queries (available memory, full-disk encryption, AES), sync-root detection
  and the sync-root rule, cloud placeholders and `--hydrate`, clone and range copy, the persistent
  log (DR-105).
- CLI deb and rpm packages of the static musl binary (DR-106, DR-114); `ci/checks/tty-flows`;
  the keychain and volume-fixture CI jobs (exFAT, FAT32, NFS, SMB).

### Changed

- The workspace is version 0.2.0: files are written with `version_minor = 2` (DR-107).
- `keys` commands report a damaged, missing or retired default identity instead of acting on
  another one (DR-97, DR-115).
- An `E_CANCELLED` from a question answered "n" ends only its input (DR-115).
- The release profile builds third-party code other than the cryptography for size, and the
  static Linux CLI has an 8 MB size budget; macOS and Windows keep 6 MB (DR-118).

## 0.1.0 (unreleased)

The first pre-release (milestone M2): the `darcq` command-line tool for Linux (static musl,
x86_64 and aarch64), macOS (universal) and Windows (x86_64 and Arm64), built on the
specifications of M0 and the core and format of M1. Its release notes, which open with the DR-29
sentence above, are docs/release-notes/0.1.0.md.

### Added

- Release engineering (M2): `release.yml`, triggered by a `vX.Y.Z` or `vX.Y.Z-rc.N` tag, builds
  every 0.1 CLI binary twice in independent jobs and compares the builds bit for bit, runs the
  release-artefact checks (size budget, static linking, build paths, version, `cargo audit bin`,
  the aarch64 NEON checks), produces one CycloneDX SBOM per binary and `THIRD-PARTY-NOTICES`,
  attests every binary and archive with `actions/attest-build-provenance`, ad-hoc signs the macOS
  universal binary and uploads everything to a draft release. The scripts under `ci/release/`
  build, check and package the binaries, write `SHA512SUMS`, sign it in the offline ceremony
  (minisign and SLH-DSA-SHAKE-256s, every signature verified) and verify a download as
  `docs/unsigned-builds.md` §3 describes. Also: the release-checklist issue template,
  `release-keys/` with the ceremony record, and the 0.1.0 release notes.
- The specifications under `docs/`: the design brief v1.4 with decisions DR-01 to DR-80,
  requirements, architecture, the container format (`format-v1.md`), cryptography
  (`crypto-v1.md`), identity (`identity-v1.md`), platform (`platform-v1.md`), the error
  catalog (`errors.md`), the core API, the CLI and GUI specifications, the threat model,
  testing, development and roadmap documents, and ADR-001 to ADR-031.
- The Rust workspace: eight crates (`darcq-format`, `darcq-kem-libcrux`, `darcq-crypto`,
  `darcq-core`, `darcq-platform`, `darcq-cli`, `darcq-gui`, `darcq-testkit`) and the dev-only
  tools `errgen` (generates the error module from `docs/errors.md`) and `ct-harness`.
- Toolchain and build configuration: Rust 1.98.1 pinned in `rust-toolchain.toml`,
  `.cargo/config.toml` with `LIBCRUX_DISABLE_SIMD128 = "1"` and the static-CRT MSVC flags,
  and the `release`, `release-nolto` and `release-ct` profiles.
- Supply-chain policy: `deny.toml`, `cargo vet` configuration and imported audits, and the
  pinned dependency budget of ADR-031.
- The Python reference implementation (`tools/refimpl/`) and testkit v0
  (`tests/vectors/darcq/`): container, identity, contacts, recipient-string and journal
  vectors, mutation vectors for every rejection rule, and the traceability map `rules.csv`.
- Vendored primitive vectors (ACVP, CCTV, X-Wing draft-11, Wycheproof, RFC 9106, BIP39,
  Bech32m) with `VECTORS.md` provenance and hashes.
- CI workflows: the PR gate on six runners (`ci.yml`), the aarch64 NEON symbol and
  `build.rs` log checks, the policy checks (`policy.yml`: `cargo deny`, `vet`, `audit`, the
  errors drift check, `unsafe` placement, the ADR-031 graph and `testkit` feature checks, the
  testkit package check, vector hashes, traceability, MSRV), the Valgrind CT gate
  (`ct.yml`), the reference regeneration check (`refimpl.yml`), nightly fuzzing with a weekly
  corpus minimisation (`fuzz.yml`), the nightly audit (`nightly.yml`) and the floor job
  (`floor.yml`); the repository check scripts under `ci/checks/`.
- `README.md`, `SECURITY.md`, `CONTRIBUTING.md`, `MAINTAINERS.md`, the pull-request template
  and the licences (code MIT OR Apache-2.0, documentation CC-BY-4.0).
- M1 library crates: `darcq-format` (every codec, limit and canonical rule; the error module
  generated from `docs/errors.md`), `darcq-kem-libcrux` (ML-KEM-768 adapter, `WipedKeyPair`
  wipe, the aarch64 `LIBCRUX_DISABLE_SIMD128` assertion), `darcq-crypto` (X-Wing, committing
  wraps, AES-256-GCM chunks, Argon2id with DARCQ-owned memory, the secret containers and
  seams) and `darcq-core` (encryptor, decryptor, unlock order, identity file, contacts,
  recipient strings, recovery words, passphrase policy, registry).
- The `darcq-testkit` library runner over all 408 testkit vectors, the six `darcq-format` fuzz
  targets with a seed corpus, the Argon2 and AES-GCM chunk benchmark, and the CI checks for
  fuzz regressions and the wipe test under Miri.
- `darcq-platform` 0.1 (M2): the file engine (`open_input`, `resolve_dir`, the volume probe,
  `open_temp`, `place_new` with its three strategies and `NAME (2)` collision naming,
  `replace_existing`, discard through the handle, fsync of file and directory, metadata and
  origin-mark restore, `--each-file` listing, `delete_original`), the 0.1 part of `harden()`,
  the data directory with its permission checks, `config.toml` read without serde (DR-93), the
  passphrase channels (byte-exact reads into secret memory; terminal and standard-input aliases
  refused), the no-echo terminal prompt and the interrupt flag (SIGINT, SIGTERM, SIGHUP,
  SIGQUIT; DR-99).
- The `darcq` command-line tool 0.1 (M2): `encrypt`, `decrypt`, `verify`, `inspect`, the
  `darcq PATH…` shorthand, and `keys init`, `show`, `list`, `default`, `verify-words`,
  `export-words`, `import-words` and `change-passphrase`; human output (control characters escaped, DR-96) and
  `--json` events; messages, hints and notes generated from `docs/errors.md` by `errgen`. The
  CLI conformance runner drives the binary through the testkit vectors, and the `config` fuzz
  target covers the new loader.

### Changed

- `ci/tools.toml` pins the release tools (`cargo-about`, `cargo-auditable`, `cargo-bloat`,
  `cargo-cyclonedx`), Xcode 16.4 and the MSVC toolset 14.44.35207 (DR-63). Release builds of the
  Apple targets link with `-Wl,-S`, so that `LC_UUID` does not depend on the build directory.
- MSRV 1.88 (was 1.85): every patched `time` release requires it (DR-79).
- `cargo audit` ignores RUSTSEC-2026-0173 (`proc-macro-error2`), which is never compiled for a
  DARCQ target, in `.cargo/audit.toml` (DR-80).
- The written version comes from the package version at compile time (DR-87): 0.x builds write
  `version_major = 0` and their 0.x minor in containers, identity files and contacts files and
  read versions 0 and 1; 1.x builds write 1 and 0 and read only 1. The testkit runner follows
  the build: in 0.x the `v0/` KATs are the encryption KATs.
- Version-1 files are read-only in a 0.x build (DR-89): every rewrite of a version-1 identity
  or contacts file is refused with `E_UNSUPPORTED` before any secret is used. The CI job
  `v1-mode` runs the library crates and the testkit at version 1.0.0 on every pull request.
