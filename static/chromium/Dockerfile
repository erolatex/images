# Base image must be Ubuntu 22.04 (jammy)
ARG BASE_IMAGE_TAG=1.0.1
FROM erolatex/selenoid_base_image:${BASE_IMAGE_TAG}

ENV DEBIAN_FRONTEND=noninteractive

# ---- Prereqs ----
RUN set -eux; \
    # Ensure jammy base
    grep -q '^UBUNTU_CODENAME=jammy' /etc/os-release || (cat /etc/os-release; exit 1); \
    apt-get update -yq; \
    apt-get install -yq --no-install-recommends \
      ca-certificates curl gnupg apt-transport-https; \
    update-ca-certificates

# ---- Install Chromium by downloading .deb files from PPA pool (no apt repo needed) ----
RUN set -eux; \
    ARCH="$(dpkg --print-architecture)"; \
    BASE_URL="https://ppa.launchpadcontent.net/xtradeb/apps/ubuntu/pool/main/c/chromium/"; \
    HTML="$(curl -fsSL "$BASE_URL")"; \
    \
    # Pick latest chromium <version> for which ARCH .deb exists
    LATEST_VER="$(printf '%s' "$HTML" \
      | grep -oP "chromium_\K[0-9]+(?:\.[0-9]+){3}(?=[^\"<]*_${ARCH}\.deb)" \
      | sort -V | tail -n1)"; \
    test -n "$LATEST_VER"; \
    \
    # Compose filenames for arch-specific and arch-all packages
    CHROMIUM_DEB="chromium_${LATEST_VER}-1xtradeb1.*_${ARCH}.deb"; \
    DRIVER_DEB="chromium-driver_${LATEST_VER}-1xtradeb1.*_${ARCH}.deb"; \
    COMMON_DEB="chromium-common_${LATEST_VER}-1xtradeb1.*_all.deb"; \
    \
    # Resolve exact URLs (globbing via grep)
    CHROMIUM_URL="$(printf '%s' "$HTML" | grep -oP "href=\\\"(chromium_${LATEST_VER}-1xtradeb1\\.[^\"<>]*_${ARCH}\\.deb)\\\"" | sed -E 's/^href=\"|\"$//g' | head -n1)"; \
    DRIVER_URL="$(printf '%s' "$HTML"   | grep -oP "href=\\\"(chromium-driver_${LATEST_VER}-1xtradeb1\\.[^\"<>]*_${ARCH}\\.deb)\\\"" | sed -E 's/^href=\"|\"$//g' | head -n1)"; \
    COMMON_URL="$(printf '%s' "$HTML"   | grep -oP "href=\\\"(chromium-common_${LATEST_VER}-1xtradeb1\\.[^\"<>]*_all\\.deb)\\\"" | sed -E 's/^href=\"|\"$//g' | head -n1)"; \
    \
    # Basic sanity checks
    test -n "$CHROMIUM_URL" && test -n "$DRIVER_URL" && test -n "$COMMON_URL"; \
    \
    # Download .debs
    curl -fsSLO "${BASE_URL}${CHROMIUM_URL}"; \
    curl -fsSLO "${BASE_URL}${DRIVER_URL}"; \
    curl -fsSLO "${BASE_URL}${COMMON_URL}"; \
    \
    # Install from local .debs and let apt resolve dependencies from jammy
    apt-get update -yq; \
    apt-get install -yq --no-install-recommends \
      ./chromium_*_${ARCH}.deb \
      ./chromium-driver_*_${ARCH}.deb \
      ./chromium-common_*_all.deb; \
    \
    # Cleanup
    rm -f chromium_*_${ARCH}.deb chromium-driver_*_${ARCH}.deb chromium-common_*_all.deb; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/*

# ---- Entrypoint ----
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
USER selenium
EXPOSE 4444
ENTRYPOINT ["/entrypoint.sh"]
