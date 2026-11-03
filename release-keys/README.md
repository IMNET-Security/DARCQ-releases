# Release keys

The **public** halves of DARCQ's two release keys, copied from the `release-keys/` folder of the
DARCQ repository at each release:

| File | What it is |
|---|---|
| `darcq-minisign.pub` | The minisign (Ed25519) key: it signs `SHA512SUMS` and, from the first final 0.4 release, each updater artefact. Two lines: `untrusted comment: minisign public key <KEY ID>`, then the key itself, starting with `RW`. |
| `darcq-slhdsa.pub.pem` | The SLH-DSA-SHAKE-256s key: it signs `SHA512SUMS`. A PEM `PUBLIC KEY` block, for OpenSSL 3.5 or later. |

No private material exists in this repository or anywhere online.
