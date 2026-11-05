# =============================================================================
# Tor Runtime Image
# =============================================================================
# Minimal image containing only the Tor binary
# Multi-arch: linux/amd64, linux/arm64
#
# Usage:
#   docker run --rm ghcr.io/binmgr/tor:latest --version
# =============================================================================

FROM scratch

ARG TARGETARCH

# Copy the pre-built binary (from GitHub release or build artifact)
COPY tor-linux-${TARGETARCH} /usr/local/bin/tor

ENTRYPOINT ["/usr/local/bin/tor"]
CMD ["--help"]
