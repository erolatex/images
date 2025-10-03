# syntax=docker/dockerfile:1.6

# Base image must be Ubuntu 22.04 (jammy)
ARG BASE_IMAGE_TAG=1.0.1
FROM erolatex/selenoid_base_image:${BASE_IMAGE_TAG}

ENV DEBIAN_FRONTEND=noninteractive

# ---- Install base tools ----
RUN set -eux; \
    # Ensure base is jammy
    grep -q '^UBUNTU_CODENAME=jammy' /etc/os-release || (cat /etc/os-release; exit 1); \
    apt-get update -yq; \
    apt-get install -yq --no-install-recommends \
      ca-certificates curl gnupg apt-transport-https; \
    update-ca-certificates

# ---- Install Chromium from xtradeb pool (not using apt repo) ----
RUN set -eux; \
  ARCH="$(dpkg --print-architecture)"; \
  BASE_URL="https://ppa.launchpadcontent.net/xtradeb/apps/ubuntu/pool/main/c/chromium/"; \
  HTML="$(curl -fsSL "$BASE_URL")"; \
  \
  # Detect latest version with available deb for this arch
  LATEST_VER="$(printf '%s' "$HTML" \
    | grep -oP "chromium_\\K[0-9]+(?:\\.[0-9]+){3}(?=[^\"<]*_${ARCH}\\.deb)" \
    | sort -V | tail -n1)"; \
  test -n "$LATEST_VER"; \
  \
  # Locate package URLs
  CHROMIUM_URL="$(printf '%s' "$HTML" \
    | grep -oP "href=\\\"(chromium_${LATEST_VER}-1xtradeb1\\.[^\"<>]*_${ARCH}\\.deb)\\\"" \
    | sed -E 's/^href=\"|\"$//g' | head -n1)"; \
  DRIVER_URL="$(printf '%s' "$HTML" \
    | grep -oP "href=\\\"(chromium-driver_${LATEST_VER}-1xtradeb1\\.[^\"<>]*_${ARCH}\\.deb)\\\"" \
    | sed -E 's/^href=\"|\"$//g' | head -n1)"; \
  COMMON_URL="$(printf '%s' "$HTML" \
    | grep -oP "href=\\\"(chromium-common_${LATEST_VER}-1xtradeb1\\.[^\"<>]*_(?:all|${ARCH})\\.deb)\\\"" \
    | sed -E 's/^href=\"|\"$//g' | head -n1)"; \
  test -n "$CHROMIUM_URL" && test -n "$DRIVER_URL" && test -n "$COMMON_URL"; \
  \
  # Optional extras
  L10N_URL="$(printf '%s' "$HTML" \
    | grep -oP "href=\\\"(chromium-l10n_${LATEST_VER}-1xtradeb1\\.[^\"<>]*_all\\.deb)\\\"" \
    | sed -E 's/^href=\"|\"$//g' | head -n1 || true)"; \
  SANDBOX_URL="$(printf '%s' "$HTML" \
    | grep -oP "href=\\\"(chromium-sandbox_${LATEST_VER}-1xtradeb1\\.[^\"<>]*_${ARCH}\\.deb)\\\"" \
    | sed -E 's/^href=\"|\"$//g' | head -n1 || true)"; \
  \
  # Download deb files
  curl -fsSLO "${BASE_URL}${CHROMIUM_URL}"; \
  curl -fsSLO "${BASE_URL}${DRIVER_URL}"; \
  curl -fsSLO "${BASE_URL}${COMMON_URL}"; \
  [ -n "$L10N_URL" ] && curl -fsSLO "${BASE_URL}${L10N_URL}" || true; \
  [ -n "$SANDBOX_URL" ] && curl -fsSLO "${BASE_URL}${SANDBOX_URL}" || true; \
  \
  # Install them; dependencies will be resolved from jammy repos
  apt-get update -yq; \
  set -- ./chromium_${LATEST_VER}-1xtradeb1.*_${ARCH}.deb \
         ./chromium-driver_${LATEST_VER}-1xtradeb1.*_${ARCH}.deb \
         ./chromium-common_${LATEST_VER}-1xtradeb1.*_*.deb; \
  [ -n "$L10N_URL" ] && set -- "$@" ./chromium-l10n_${LATEST_VER}-1xtradeb1.*_all.deb; \
  [ -n "$SANDBOX_URL" ] && set -- "$@" ./chromium-sandbox_${LATEST_VER}-1xtradeb1.*_${ARCH}.deb; \
  apt-get install -yq --no-install-recommends "$@"; \
  \
  # Cleanup
  rm -f chromium-*_${LATEST_VER}-1xtradeb1.*.deb; \
  apt-get purge -y --auto-remove curl gnupg || true; \
  apt-get clean; \
  rm -rf /var/lib/apt/lists/*

# ---- Entrypoint ----
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

USER selenium
EXPOSE 4444
ENTRYPOINT ["/entrypoint.sh"]
