## Project description

A CI-driven pipeline that builds and distributes fully static [Tor](https://www.torproject.org) binaries for eight platforms (Linux, macOS, Windows, FreeBSD x amd64/arm64). A pre-baked Ubuntu build image (`ghcr.io/binmgr/tor:build`) holds all cross-compile toolchains and statically built dependencies; binaries are then built from the official upstream release tarball. Published artifacts are standalone binaries, a SHA-256 `checksums.txt`, and a scratch-based Docker runtime image.

## Project variables

project_name: tor
project_org: binmgr
internal_name: tor
internal_org: binmgr
upstream_source: https://dist.torproject.org
registry: ghcr.io
image_name: binmgr/tor
supported_platforms: linux-amd64, linux-arm64, darwin-amd64, darwin-arm64, windows-amd64, windows-arm64, freebsd-amd64, freebsd-arm64

## Business logic

**Must have:**
- Static `tor` for all eight targets (macOS links system libSystem only)
- Statically linked libevent, OpenSSL, zlib, zstd, and liblzma
- `checksums.txt` (SHA-256) covering every release asset
- Docker runtime image `ghcr.io/binmgr/tor:latest` for linux/amd64 and linux/arm64
- Builds on a monthly schedule, on build-image changes, and manually with an optional version override
- All five CI providers (GitHub, Gitea, Forgejo, GitLab, Jenkins) kept in lockstep

**Must NOT have:**
- Patched Tor source — upstream release tarballs only
- Relay and directory-authority modules (`--disable-module-relay`, `--disable-module-dirauth`)

**Constraints:**
- Dependency versions are pinned in `docker/Dockerfile.build`; change them deliberately
