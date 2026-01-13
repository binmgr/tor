# Tor Static Builds

[![Build Status](https://img.shields.io/badge/build-automated-brightgreen)]()
[![License](https://img.shields.io/badge/license-BSD--3--Clause-blue)](LICENSE.md)
[![Tor](https://img.shields.io/badge/Tor-latest-purple)](https://www.torproject.org)

> **Fully static, multi-platform Tor binaries with all features enabled**

---

## Overview

This repository provides automated builds of Tor as **fully static binaries** for multiple platforms and architectures. Binaries are built using **musl libc** for maximum portability and are stripped for minimal size.

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

Each release includes SHA256 checksums in `checksums.txt` for verification:

```bash
# Verify download
sha256sum -c checksums.txt
```

## Features

### Enabled Components

| Component | Description |
|-----------|-------------|
| **OpenSSL** | TLS/SSL support for secure connections |
| **libevent** | High-performance event notification |
| **zlib** | Compression support |
| **zstd** | Zstandard compression (faster) |
| **liblzma** | LZMA compression support |
| **libcap** | Linux capabilities (Linux only) |

### Build Configuration

```bash
--enable-static-tor
--enable-static-libevent
--enable-static-openssl
--enable-static-zlib
--enable-zstd
--enable-lzma
--disable-asciidoc
--disable-manpage
--disable-html-manual
--disable-module-relay
--disable-module-dirauth
```

### Security Features

- Position Independent Executable (PIE)
- Stack protector
- RELRO (Relocation Read-Only)
- FORTIFY_SOURCE

## Build Schedule

| Trigger | Description |
|---------|-------------|
| **Push** | Builds on every push to main branch |
| **Monthly** | Scheduled build on the 1st of each month |
| **Manual** | Can be triggered manually via workflow dispatch |

## Version Scheme

Release versions follow Tor's version numbering:

```
v{TOR_VERSION}

Example: v0.4.8.10, v0.4.8.11
```

## Multi-Architecture Support

All builds include proper OCI annotations for multi-architecture support:

| Label | Description |
|-------|-------------|
| `org.opencontainers.image.title` | Tor Static Build |
| `org.opencontainers.image.version` | Tor version |
| `org.opencontainers.image.os` | Target operating system |
| `org.opencontainers.image.architecture` | Target architecture |
| `org.opencontainers.image.created` | Build timestamp |
| `org.opencontainers.image.source` | Repository URL |
| `org.opencontainers.image.revision` | Git commit SHA |

## CI/CD Platform Support

This repository includes workflows for multiple CI/CD platforms:

| Platform | Workflow File | Status |
|----------|---------------|--------|
| **GitHub Actions** | `.github/workflows/build.yml` | Full support |
| **Gitea Actions** | `.gitea/workflows/build.yml` | Full support |
| **Nektos/act** | Compatible with GitHub Actions | Local testing |

### Running Locally with Nektos/act

```bash
# Install act (https://github.com/nektos/act)
# Then run:
act push
```

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

## Repository Structure

```
.
├── .github/
│   └── workflows/
│       └── build.yml          # GitHub Actions workflow
├── .gitea/
│   └── workflows/
│       └── build.yml          # Gitea Actions workflow
├── LICENSE.md                 # License information
└── README.md                  # This file
```

## Contributing

Contributions are welcome! Please feel free to submit issues and pull requests.

## License

Tor is licensed under the **BSD 3-Clause License**.

Build scripts in this repository are licensed under the **MIT License**.

See [LICENSE.md](LICENSE.md) for full details.

## Links

- [Tor Project Official Website](https://www.torproject.org)
- [Tor Documentation](https://support.torproject.org)
- [Tor Source Code](https://gitlab.torproject.org/tpo/core/tor)

---

<p align="center">
  <i>Built with automated CI/CD pipelines</i>
</p>
