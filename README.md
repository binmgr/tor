# Tor Static Builds

[![Build Status](https://img.shields.io/badge/build-automated-brightgreen)]()
[![License](https://img.shields.io/badge/license-BSD--3--Clause-blue)](LICENSE.md)
[![Tor](https://img.shields.io/badge/Tor-latest-purple)](https://www.torproject.org)

> **Fully static, multi-platform Tor binaries — built and released automatically**

---

## Overview

This repository provides automated builds of Tor as **fully static binaries** for multiple platforms and architectures. Binaries are stripped for minimal size and linked statically against all dependencies so they run with no external libraries required.

```
┌─────────────────────────────────────────────────────────────────┐
│                    Supported Platforms                          │
├─────────────────┬─────────────────┬─────────────────────────────┤
│    Platform     │   Architectures │         Binary Name         │
├─────────────────┼─────────────────┼─────────────────────────────┤
│    Linux        │  amd64, arm64   │  tor-linux-{arch}           │
│    Windows      │  amd64, arm64   │  tor-windows-{arch}.exe     │
│    macOS        │  amd64, arm64   │  tor-darwin-{arch}          │
│    FreeBSD      │  amd64, arm64   │  tor-freebsd-{arch}         │
└─────────────────┴─────────────────┴─────────────────────────────┘
```

## Download

### Latest Release

Download the latest binaries from the [Releases](../../releases/latest) page.

| Platform | AMD64 | ARM64 |
|----------|-------|-------|
| **Linux** | [`tor-linux-amd64`](../../releases/latest/download/tor-linux-amd64) | [`tor-linux-arm64`](../../releases/latest/download/tor-linux-arm64) |
| **Windows** | [`tor-windows-amd64.exe`](../../releases/latest/download/tor-windows-amd64.exe) | [`tor-windows-arm64.exe`](../../releases/latest/download/tor-windows-arm64.exe) |
| **macOS** | [`tor-darwin-amd64`](../../releases/latest/download/tor-darwin-amd64) | [`tor-darwin-arm64`](../../releases/latest/download/tor-darwin-arm64) |
| **FreeBSD** | [`tor-freebsd-amd64`](../../releases/latest/download/tor-freebsd-amd64) | [`tor-freebsd-arm64`](../../releases/latest/download/tor-freebsd-arm64) |

### Checksums

Each release includes SHA256 checksums in `checksums.txt`:

```bash
sha256sum -c checksums.txt
```

## Features

### Statically Linked Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| **OpenSSL** | 3.2.1 | TLS/SSL cryptographic library |
| **libevent** | 2.1.12-stable | Event notification library |
| **zlib** | 1.3.2 | Compression support |
| **zstd** | 1.5.5 | Zstandard compression (faster) |
| **liblzma** | 5.4.5 | LZMA compression support |

### Tor Configure Flags

```bash
--enable-static-tor
--with-libevent-dir=<deps>
--with-openssl-dir=<deps>
--with-zlib-dir=<deps>
--enable-zstd
--enable-lzma
--disable-asciidoc
--disable-manpage
--disable-html-manual
--disable-module-relay
--disable-module-dirauth
--disable-systemd
--disable-seccomp
--disable-libscrypt

# Windows only
--enable-static-libevent
--enable-static-openssl
--enable-static-zlib

# macOS only
--disable-gcc-hardening
```

### Toolchains

| Platform | Toolchain |
|----------|-----------|
| Linux | musl-cross (musl libc, fully static) |
| Windows | mingw-w64 (amd64), llvm-mingw (arm64) |
| macOS | osxcross with MacOSX 14.0 SDK |
| FreeBSD | clang + lld with FreeBSD 14.1 sysroot |

## Build Schedule

| Trigger | Description |
|---------|-------------|
| **Push** | Builds on every push to main/master that touches workflow or Dockerfile |
| **Monthly** | Scheduled build on the 1st of each month |
| **Manual** | Triggered via workflow dispatch with optional version override |

## Version Scheme

Release tags follow Tor's version numbering:

```
v{TOR_VERSION}

Example: v0.4.8.10, v0.4.8.11
```

The latest released Tor version is detected automatically at build time from the Tor Project's distribution server.

## Usage Examples

### Run as SOCKS Proxy

```bash
# Start Tor with default SOCKS port (9050)
./tor-linux-amd64

# Start with custom SOCKS port
./tor-linux-amd64 --SocksPort 9150

# Run in background
./tor-linux-amd64 --RunAsDaemon 1
```

### Custom Configuration

```bash
# Use custom torrc
./tor-linux-amd64 -f /path/to/torrc

# Specify data directory
./tor-linux-amd64 --DataDirectory /path/to/data
```

### Use with curl

```bash
# Test Tor connection
curl --socks5-hostname 127.0.0.1:9050 https://check.torproject.org/api/ip

# Browse .onion sites
curl --socks5-hostname 127.0.0.1:9050 http://example.onion
```

### Use with SSH

```bash
# SSH over Tor
ssh -o ProxyCommand="nc -X 5 -x 127.0.0.1:9050 %h %p" user@host
```

## Build Matrix

```
┌──────────┬─────────┬─────────┬─────────┬─────────┐
│          │  Linux  │ Windows │  macOS  │ FreeBSD │
├──────────┼─────────┼─────────┼─────────┼─────────┤
│  amd64   │    ✓    │    ✓    │    ✓    │    ✓    │
│  arm64   │    ✓    │    ✓    │    ✓    │    ✓    │
└──────────┴─────────┴─────────┴─────────┴─────────┘
```

## CI/CD Platform Support

| Platform | Workflow Files | Notes |
|----------|----------------|-------|
| **GitHub Actions** | `.github/workflows/` | Releases to GitHub Releases + ghcr.io |
| **Gitea Actions** | `.gitea/workflows/` | Releases via Gitea API |

### Running Locally with act

```bash
# Validate workflow files
act --list -W .github/workflows/build-binaries.yml

# Run a specific job
act -j build push
```

## Repository Structure

```
.
├── .github/
│   └── workflows/
│       ├── build-env-image.yml   # Build + push ghcr.io/binmgr/tor:build
│       └── build-binaries.yml    # Build all 8 binaries + create release
├── .gitea/
│   └── workflows/
│       ├── build-env-image.yml   # Build + push ghcr.io/binmgr/tor:build
│       └── build-binaries.yml    # Build all 8 binaries + create release
├── docker/
│   └── Dockerfile.build          # Build environment (Alpine + cross-compilers)
├── Dockerfile                    # Minimal runtime image (FROM scratch)
├── LICENSE.md                    # License information
└── README.md                     # This file
```

## Contributing

Contributions are welcome! Please feel free to submit issues and pull requests.

## License

Tor is licensed under the **BSD 3-Clause License**.

Build scripts and CI/CD configurations in this repository are licensed under the **MIT License**.

See [LICENSE.md](LICENSE.md) for full details.

## Links

- [Tor Project Official Website](https://www.torproject.org)
- [Tor Documentation](https://support.torproject.org)
- [Tor Source Code](https://gitlab.torproject.org/tpo/core/tor)

---

<p align="center">
  <i>Built with automated CI/CD pipelines</i>
</p>
