# Tor Static Build — Specification

## Overview

Fully static, stripped Tor binaries for 4 platforms × 2 architectures, built
and released automatically via GitHub Actions and Gitea Actions. No external
libraries required at runtime.

---

## Build Matrix

| Platform | Arch  | Binary name               |
|----------|-------|---------------------------|
| Linux    | amd64 | `tor-linux-amd64`         |
| Linux    | arm64 | `tor-linux-arm64`         |
| Windows  | amd64 | `tor-windows-amd64.exe`   |
| Windows  | arm64 | `tor-windows-arm64.exe`   |
| macOS    | amd64 | `tor-darwin-amd64`        |
| macOS    | arm64 | `tor-darwin-arm64`        |
| FreeBSD  | amd64 | `tor-freebsd-amd64`       |
| FreeBSD  | arm64 | `tor-freebsd-arm64`       |

---

## Toolchains

All builds run on `ubuntu-latest` GitHub/Gitea runners. No pre-built container
is used for the binary build step.

### Linux (amd64 + arm64) — zig cross-compiler

Zig is used as the cross-compiler rather than musl.cc or any distribution
package, matching the approach in `docker/Dockerfile.build`. Zig is downloaded
as a self-contained static binary; wrapper scripts are created so build systems
that expect conventional compiler names (`*-gcc`, `*-ar`, etc.) work without
modification.

**Zig version:** 0.14.0  
**Download:** `https://ziglang.org/download/0.14.0/zig-linux-x86_64-0.14.0.tar.xz`

Wrapper scripts created at `$HOME/cross/bin/`:

| Script name                  | Invokes                                                               |
|------------------------------|-----------------------------------------------------------------------|
| `x86_64-linux-musl-gcc`      | `zig cc  -target x86_64-linux-musl -static -L{deps}/lib`             |
| `x86_64-linux-musl-g++`      | `zig c++ -target x86_64-linux-musl -static -L{deps}/lib`             |
| `x86_64-linux-musl-ar`       | `zig ar`                                                              |
| `x86_64-linux-musl-ranlib`   | `zig ranlib`                                                          |
| `x86_64-linux-musl-strip`    | `llvm-strip`                                                          |
| `aarch64-linux-musl-gcc`     | `zig cc  -target aarch64-linux-musl -static -L{deps}/lib`            |
| `aarch64-linux-musl-g++`     | `zig c++ -target aarch64-linux-musl -static -L{deps}/lib`            |
| `aarch64-linux-musl-ar`      | `zig ar`                                                              |
| `aarch64-linux-musl-ranlib`  | `zig ranlib`                                                          |
| `aarch64-linux-musl-strip`   | `llvm-strip`                                                          |

`-static` and `-L{deps}/lib` are baked into the `gcc`/`g++` wrappers at creation time (with
`{deps}` expanded to the actual `$DEPS_PREFIX` path). This is required because autoconf's
initial "C compiler works" test links `int main(){}` using only `CFLAGS` + `LIBS` — it does
**not** pass `LDFLAGS`. Without the `-L` path in the wrapper, zig's `paths_first` library
resolution strategy has zero search paths when it encounters `-lssl` etc., causing the CC
test to fail with `configure: error: C compiler cannot create executables`.

OpenSSL targets: `linux-x86_64` (amd64), `linux-aarch64` (arm64).

### Windows (amd64) — mingw-w64

Installed from the Ubuntu apt repository (`mingw-w64`).

| Variable | Value                     |
|----------|---------------------------|
| `CC`     | `x86_64-w64-mingw32-gcc`  |
| `HOST`   | `x86_64-w64-mingw32`      |
| OpenSSL  | `mingw64`                 |

### Windows (arm64) — llvm-mingw

Downloaded from the [mstorsjo/llvm-mingw](https://github.com/mstorsjo/llvm-mingw)
releases. Uses clang-based cross-compiler with UCRT runtime.

**Release:** `llvm-mingw-20241217-ucrt-ubuntu-20.04-x86_64`

| Variable | Value                       |
|----------|-----------------------------|
| `CC`     | `aarch64-w64-mingw32-clang` |
| `HOST`   | `aarch64-w64-mingw32`       |
| OpenSSL  | `mingw64`                   |

zlib is built with cmake (`-DCMAKE_SYSTEM_NAME=Windows`) because zlib 1.3.x
removed the legacy `win32/Makefile.gcc`.

### macOS (amd64 + arm64) — osxcross

Cloned from [tpoechtrager/osxcross](https://github.com/tpoechtrager/osxcross)
and built with the MacOSX 14.0 SDK.

**SDK:** `MacOSX14.0.sdk.tar.xz` from
[joseluisq/macosx-sdks](https://github.com/joseluisq/macosx-sdks)

| Arch  | `CC`                            | `HOST`                    | OpenSSL target          |
|-------|---------------------------------|---------------------------|-------------------------|
| amd64 | `x86_64-apple-darwin23-clang`   | `x86_64-apple-darwin23`   | `darwin64-x86_64-cc`    |
| arm64 | `aarch64-apple-darwin23-clang`  | `aarch64-apple-darwin23`  | `darwin64-arm64-cc`     |

Tor configure adds `--disable-gcc-hardening` for macOS.

### FreeBSD (amd64 + arm64) — clang + FreeBSD sysroot

FreeBSD system libraries are extracted from the official base tarball and used
as a cross-compilation sysroot. Ubuntu's `clang` and `lld` handle the actual
compilation.

**FreeBSD version:** 14.3-RELEASE  
**Sysroot:** `/opt/freebsd` (extracted: `./lib ./usr/lib ./usr/include`)

Download URLs:

```
https://download.freebsd.org/releases/amd64/amd64/14.3-RELEASE/base.txz
https://download.freebsd.org/releases/arm64/aarch64/14.3-RELEASE/base.txz
```

| Arch  | `--target` / `--sysroot`               | OpenSSL target   |
|-------|----------------------------------------|------------------|
| amd64 | `x86_64-unknown-freebsd14`             | `BSD-x86_64`     |
| arm64 | `aarch64-unknown-freebsd14`            | `BSD-generic64`  |

Both `CFLAGS` and `LDFLAGS` carry `--sysroot=/opt/freebsd --target=<triple>`
and `-fuse-ld=lld`.

---

## Dependencies

| Library      | Version         | Source URL                                                          |
|--------------|-----------------|---------------------------------------------------------------------|
| **OpenSSL**  | 3.2.1           | `https://www.openssl.org/source/openssl-3.2.1.tar.gz`              |
| **libevent** | 2.1.12-stable   | GitHub releases (`libevent/libevent`)                               |
| **zlib**     | 1.3.2           | `https://zlib.net/zlib-1.3.2.tar.gz`                               |
| **zstd**     | 1.5.5           | GitHub releases (`facebook/zstd`)                                   |
| **xz/liblzma** | 5.4.5         | GitHub releases (`tukaani-project/xz`)                              |

All dependencies are built as static libraries from source for each target.
No host-installed libraries are used.

### OpenSSL configure flags

```
--prefix=<deps> no-shared no-tests no-ui-console
```

`--cross-compile-prefix` is **not** used; the `CC` environment variable points
directly to the appropriate compiler wrapper.

### libevent configure flags

```
--host=<triple> --enable-static --disable-shared
--disable-samples --disable-libevent-regress
OPENSSL_CFLAGS/OPENSSL_LIBS pointing to deps prefix
```

### zstd build

```
make lib-release CC=<cross-cc> AR=<cross-ar> RANLIB=<cross-ranlib>
```

Shared library output (`libzstd.so*`, `libzstd.dylib*`) is removed after
install to prevent accidental dynamic linking.

### xz/liblzma configure flags

```
--host=<triple> --enable-static --disable-shared
--disable-xz --disable-xzdec --disable-lzmadec --disable-lzmainfo
--disable-scripts --disable-doc
```

---

## Tor Configure Flags

```
--host=<triple>
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
```

Windows only: `--enable-static-libevent --enable-static-openssl --enable-static-zlib`

macOS only: `--disable-gcc-hardening`

Windows link extras in `LIBS`: `-lws2_32 -lcrypt32 -lgdi32 -liphlpapi`

---

## Version Detection

Latest Tor version is detected at build time:

```bash
curl -s https://dist.torproject.org/ \
  | grep -oP 'tor-\K[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' \
  | sort -V | tail -1
```

Can be overridden via `workflow_dispatch` input `tor_version`.

Release tags follow Tor's own versioning: `v{TOR_VERSION}` (e.g. `v0.4.9.8`).

---

## CI/CD

| Platform        | Workflow files           | Trigger                                      |
|-----------------|--------------------------|----------------------------------------------|
| GitHub Actions  | `.github/workflows/`     | push to main/master, monthly cron, dispatch  |
| Gitea Actions   | `.gitea/workflows/`      | push to main/master, monthly cron, dispatch  |

### Workflows

**`build-env-image.yml`** — builds and pushes `ghcr.io/binmgr/tor:build` (the
build environment Docker image). Triggers on changes to `docker/Dockerfile.build`
or the workflow file itself, plus quarterly cron and manual dispatch.

**`build-binaries.yml`** — builds all 8 binaries, creates a GitHub/Gitea
release, and pushes the runtime Docker image (`ghcr.io/binmgr/tor:latest`).

### Action SHA pinning

All third-party actions are pinned to a full commit SHA, never a tag:

| Action                         | SHA                                        |
|--------------------------------|--------------------------------------------|
| `actions/checkout`             | `34e114876b0b11c390a56381ad16ebd13914f8d5` |
| `docker/setup-buildx-action`   | `8d2750c68a42422c14e847fe6c8ac0403b4cbd6f` |
| `docker/login-action`          | `c94ce9fb468520275223c153574b00df6fe4bcc9` |
| `docker/build-push-action`     | `ca052bb54ab0790a636c9b5f226502c73d547a25` |
| `actions/upload-artifact`      | `ea165f8d65b6e75b540449e92b4886f43607fa02` |
| `actions/download-artifact`    | `d3f86a106a0bac45b974a628896c90dbdf5c8093` |

---

## Runtime Image

`Dockerfile` (FROM scratch) contains only the Linux amd64 and arm64 binaries.
Built and pushed during the `release` job after all 8 binaries are assembled.

Tags pushed: `:latest`, `:{TOR_VERSION}`, `:{YYMM}`
