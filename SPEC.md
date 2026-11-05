# Project-specific rule overrides

This file wins over AI.md which wins over global CLAUDE.md.
Only rules that actively differ from the defaults are listed here.

---

## Exception 1 — Gitea/Forgejo/GitLab/Jenkins builds pull-or-build the build image

**Default rule:** CI binary builds run inside `ghcr.io/binmgr/tor:build`.

**Override:** The Gitea, Forgejo, and GitLab pipelines run `docker pull ghcr.io/binmgr/tor:build` and, if that fails, build `docker/Dockerfile.build` locally as `local/tor:build`, then call `build-tor` inside it.

**Rationale:** Self-hosted runners may not reach `ghcr.io`. Falling back to a local image build keeps those pipelines runnable without registry access, at the cost of a long first run.

---

## Exception 2 — Dependency source tarballs are not checksum-pinned

**Default rule:** Downloaded artifacts are verified against a known-good checksum.

**Override:** `docker/Dockerfile.build` downloads zlib, OpenSSL, libevent, zstd, xz, and toolchain archives over HTTPS without pinned SHA-256 digests. `build-tor` does the same for the Tor tarball.

**Rationale:** A digest table for every upstream release is a recurring maintenance burden. The build image is rebuilt from scratch quarterly, and `checksums.txt` is published for every release asset.

**Accepted risk:** A compromised upstream tarball would not be caught before the binary is built.

---

## Exception 3 — Build and release are combined in `build-binaries.yml`

**Default rule:** Separate `build.yml` and `release.yml` workflows.

**Override:** One workflow builds all targets and publishes the release and runtime image. There is no intermediate test stage between build and release.

**Rationale:** Builds are deterministic for a given upstream version; a separate release gate adds no signal.

---

## Exception 4 — No language template in AI.md

**Default rule:** Projects copy the Go or Rust `AI.md` template.

**Override:** This is a CI/distribution project with no maintained Go, Rust, Node, or Python source. `AI.md` documents the build pipeline instead.

---

## Exception 5 — Runtime Dockerfile is `docker/Dockerfile` with the repo root as build context

**Default rule:** Dockerfile and build context live together under `docker/`.

**Override:** `docker/Dockerfile` is built with context `.` because the release job copies `torc-linux-amd64` and `torc-linux-arm64` into the repo root before `docker buildx build`. Do not move the binaries into `docker/`.

---

## Exception 6 — Relay and directory-authority modules are disabled

**Default rule:** Build upstream with its default feature set.

**Override:** Tor is configured with `--disable-module-relay --disable-module-dirauth`, producing a client-only binary. Do not enable them without also reviewing the dependency set.

---

## Exception 7 — Jenkins is verification-only

**Default rule:** All five CI providers publish releases.

**Override:** The Jenkinsfile builds and archives binaries but never publishes a release or image.

**Rationale:** Releases are owned by the hosted providers; Jenkins mirrors the build gates only.
