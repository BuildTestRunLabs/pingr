# Security Policy

## Supported versions

Only the latest release gets fixes.

## Reporting a vulnerability

Report privately through GitHub: the repository's **Security** tab, then **Report a vulnerability**. Please do not open a public issue.

Include the pingr version (`pingr --version`), your distro and kernel, and steps to reproduce. Expect an acknowledgement within a week.

## Verifying downloads

Release archives ship with a `.sha256` file and a signed build attestation. See the release notes for `sha256sum -c` and `gh attestation verify` (needs a recent GitHub CLI).
