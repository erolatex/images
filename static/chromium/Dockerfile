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

# ---- Install Google Chrome Stable ----
RUN set -eux; \
  install -d -m 0755 /etc/apt/keyrings; \
  curl -fsSL https://dl.google.com/linux/linux_signing_key.pub \
    | gpg --dearmor -o /etc/apt/keyrings/google-linux.gpg; \
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/google-linux.gpg] https://dl.google.com/linux/chrome/deb/ stable main" \
    > /etc/apt/sources.list.d/google-chrome.list; \
  apt-get update -yq; \
  apt-get install -yq --no-install-recommends google-chrome-stable; \
  ln -sf /usr/bin/google-chrome /usr/bin/chrome || true; \
  ln -sf /usr/bin/google-chrome /usr/bin/chromium-browser || true; \
  google-chrome --version

# ---- Install matching ChromeDriver ----
RUN set -eux; \
  CHROME_VER="$(/usr/bin/google-chrome --version | awk '{print $3}')"; \
  MAJOR="${CHROME_VER%%.*}"; \
  EXACT_JSON="https://googlechromelabs.github.io/chrome-for-testing/known-good-versions-with-downloads.json"; \
  DRIVER_URL="$( \
    curl -fsSL "$EXACT_JSON" \
    | jq -r --arg major "$MAJOR" '
        [ .versions[]
          | select(.version | startswith($major + "."))
          | .downloads.chromedriver[]
          | select(.platform=="linux64")
        ][-1].url // empty
      ' \
  )"; \
  if [ -z "$DRIVER_URL" ]; then \
    DRIVER_URL="$( \
      curl -fsSL https://googlechromelabs.github.io/chrome-for-testing/last-known-good-versions.json \
      | jq -r '.channels.Stable.version as $v | "https://storage.googleapis.com/chrome-for-testing-public/\($v)/linux64/chromedriver-linux64.zip"' \
    )"; \
  fi; \
  echo "Downloading: $DRIVER_URL"; \
  curl -fsSLo /tmp/chromedriver.zip "$DRIVER_URL"; \
  unzip -q /tmp/chromedriver.zip -d /tmp/; \
  install -m 0755 /tmp/chromedriver-linux64/chromedriver /usr/local/bin/chromedriver; \
  rm -rf /tmp/chromedriver.zip /tmp/chromedriver-linux64; \
  chromedriver --version

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
