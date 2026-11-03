# DARCQ releases

This repository holds the released files of DARCQ, the offline post-quantum file-encryption app
for Linux, macOS and Windows: the installers, the command-line archives and packages, `SHA512SUMS`
with its two signatures, the software bills of materials and, from the first final 0.4 release,
the app's update manifest `latest.json`. It holds no source code.

**Downloads:** the [releases page](https://github.com/IMNET-Security/DARCQ-releases/releases).
A release candidate (`0.4.0-rc.N`) is marked as a pre-release; the app never offers one as an
update.

**Every 0.x release says:** files written by it will not be readable by DARCQ 1.0; keep that build
until you have re-encrypted them.

## Verify a download

No 0.4 file carries a platform signature, so Windows and macOS warn on first launch. Verify each
download against the DARCQ release keys before you open it: [VERIFYING.md](VERIFYING.md) gives the
steps, and [`release-keys/`](release-keys/) holds the public keys.

- minisign: key ID `AF5879C2A469F94D`, public key `RWRN+WmkwnlYr71ZtctzgjwQIVgThL7e8IDiTBnnIWV7EKm+9ni9ZRgH`
- SLH-DSA-SHAKE-256s: SHA-256 of the DER public key
  `d9623e2d7de30fd83576ef3a07d53f6fe48af060bffaf1d0599a0530305b82c9`

These are the 0.x keys, made at an interim key ceremony on 2026-10-04; they sign 0.x releases
only and are replaced before 1.0, in a release that these keys sign. Compare the minisign key
with the one printed in each release announcement; if the key has changed without such an
announcement, stop.

## Reporting a problem

Report a suspected vulnerability privately through this repository's **Security** tab, **Report a
vulnerability** ([SECURITY.md](SECURITY.md)); do not open a public issue for one.
