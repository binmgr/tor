# Security Policy

## Supported Versions

This repository builds and publishes the latest stable Tor release. Only the most recent published release receives security updates — older tagged releases are not patched.

| Version | Supported |
|---------|-----------|
| Latest published release | Yes |
| Older releases | No — rebuild from `main` |

## Reporting a Vulnerability

If you discover a security issue with the build pipeline, container image, or published binaries, please report it privately:

- Use GitHub's **Private vulnerability reporting** on this repository, or
- Email the maintainers at `security@binmgr.invalid`

**Do not open a public issue for security reports.**

You will receive an acknowledgement within 48 hours. Confirmed critical issues will be patched within 14 days; the fix will be released in a new tagged build with the issue referenced in the release notes.

## Disclosure Timeline

This project follows coordinated disclosure. Once a fix is released, security advisories will be published via GitHub Security Advisories with full details and credit to the reporter (unless anonymity is requested).

## Out of Scope

- Vulnerabilities in upstream Tor itself — report those to the [Tor Project](https://www.torproject.org/about/contact/).
- Vulnerabilities in bundled libraries (OpenSSL, libevent, zlib, zstd, xz) — report to their upstream maintainers.
- Misuse of the binaries to run a relay or access content in violation of applicable law.
