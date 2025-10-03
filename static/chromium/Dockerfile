# syntax=docker/dockerfile:1.6

ARG BASE_IMAGE_TAG=1.0.1
FROM erolatex/selenoid_base_image:${BASE_IMAGE_TAG}

ENV DEBIAN_FRONTEND=noninteractive

# ---- Base tools ----
RUN set -eux; \
  grep -q '^UBUNTU_CODENAME=jammy' /etc/os-release || (cat /etc/os-release; exit 1); \
  apt-get update -yq; \
  apt-get install -yq --no-install-recommends \
    ca-certificates curl gnupg apt-transport-https \
    unzip xz-utils jq; \
  update-ca-certificates

# ---- Install Chrome for Testing Stable + matching ChromeDriver (arch-aware) ----
# Uses Chrome for Testing for both amd64 and arm64 to avoid missing packages on arm64.
RUN set -eux; \
  ARCH="$(dpkg --print-architecture)"; \
  case "$ARCH" in \
    amd64)  CFT_PLATFORM="linux64";      CHROME_DIR="chrome-linux64";      DRIVER_DIR="chromedriver-linux64" ;; \
    arm64)  CFT_PLATFORM="linux-arm64";  CHROME_DIR="chrome-linux-arm64";  DRIVER_DIR="chromedriver-linux-arm64" ;; \
    armhf|armel) CFT_PLATFORM="linux-arm32"; CHROME_DIR="chrome-linux-arm32"; DRIVER_DIR="chromedriver-linux-arm32" ;; \
    *) echo "Unsupported architecture: $ARCH" >&2; exit 1 ;; \
  esac; \
  # Get stable version once and use it for both chrome and chromedriver
  STABLE_VER="$(curl -fsSL https://googlechromelabs.github.io/chrome-for-testing/last-known-good-versions.json \
    | jq -r '.channels.Stable.version')" ; \
  CHROME_URL="https://storage.googleapis.com/chrome-for-testing-public/${STABLE_VER}/${CFT_PLATFORM}/chrome-${CFT_PLATFORM}.zip"; \
  DRIVER_URL="https://storage.googleapis.com/chrome-for-testing-public/${STABLE_VER}/${CFT_PLATFORM}/chromedriver-${CFT_PLATFORM}.zip"; \
  echo "Chrome for Testing: ${STABLE_VER} (${CFT_PLATFORM})"; \
  echo "Downloading: $CHROME_URL"; \
  curl -fsSLo /tmp/chrome.zip "$CHROME_URL"; \
  echo "Downloading: $DRIVER_URL"; \
  curl -fsSLo /tmp/chromedriver.zip "$DRIVER_URL"; \
  # Install Chrome
  rm -rf /opt/chrome && mkdir -p /opt/chrome; \
  unzip -q /tmp/chrome.zip -d /opt/; \
  mv "/opt/${CHROME_DIR}"/* /opt/chrome/; \
  ln -sf /opt/chrome/chrome /usr/bin/google-chrome; \
  ln -sf /opt/chrome/chrome /usr/bin/chrome; \
  ln -sf /opt/chrome/chrome /usr/bin/chromium-browser; \
  /usr/bin/google-chrome --version; \
  # Install Chromedriver
  unzip -q /tmp/chromedriver.zip -d /tmp/; \
  install -m 0755 "/tmp/${DRIVER_DIR}/chromedriver" /usr/local/bin/chromedriver; \
  chromedriver --version; \
  # Cleanup zips
  rm -rf /tmp/chrome.zip /tmp/chromedriver.zip "/tmp/${DRIVER_DIR}"

# ---- Cleanup ----
RUN set -eux; \
  apt-get purge -y --auto-remove jq xz-utils unzip || true; \
  apt-get clean; \
  rm -rf /var/lib/apt/lists/*

# ---- Entrypoint ----
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

USER selenium
EXPOSE 4444
ENTRYPOINT ["/entrypoint.sh"]
