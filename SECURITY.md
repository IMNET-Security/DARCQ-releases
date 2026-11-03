# Security policy

Report vulnerabilities in DARCQ privately through **GitHub private vulnerability reporting** on
this repository: open its **Security** tab and choose **Report a vulnerability** (the page
`security/advisories/new`). Do not open a public issue, pull request or discussion for a
suspected vulnerability.

Please include the affected version or file and the operating system, the component (file
format, cryptography, identity and keys, file engine, CLI, GUI, updater, build or release
process), steps to reproduce with a minimal input where possible, and the impact you expect.
Never send real passphrases, recovery words, identity files or private keys; reproduce with a
throwaway test identity instead.

The maintainer acknowledges each report, agrees a disclosure date with the reporter, and
publishes a security advisory with the fix. Security fixes go into the latest release only;
files written by a 0.x release are not readable by DARCQ 1.0, so 0.x releases get no fixes
after 1.0 ships.
