# Base image must be Ubuntu 22.04 (jammy)
ARG BASE_IMAGE_TAG=1.0.1
FROM erolatex/selenoid_base_image:${BASE_IMAGE_TAG}

ENV DEBIAN_FRONTEND=noninteractive

# ---- Prereqs ----
RUN set -eux; \
    # Ensure jammy base
    grep -q '^UBUNTU_CODENAME=jammy' /etc/os-release || (cat /etc/os-release; exit 1); \
    apt-get update -yq; \
    apt-get install -yq --no-install-recommends ca-certificates curl gnupg; \
    update-ca-certificates; \
    # Keep jammy as default release so deps come from jammy unless we say otherwise
    echo 'APT::Default-Release "jammy";' > /etc/apt/apt.conf.d/99defaultrelease

# ---- Add xtradeb/apps PPA (noble) ----
RUN set -eux; \
    mkdir -p /usr/share/keyrings; \
    curl -fsSL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x5301FA4FD93244FBC6F6149982BB6851C64F6880" \
      | gpg --dearmor >/usr/share/keyrings/xtradeb_apps.gpg; \
    echo "deb [signed-by=/usr/share/keyrings/xtradeb_apps.gpg] https://ppa.launchpadcontent.net/xtradeb/apps/ubuntu noble main" \
      >/etc/apt/sources.list.d/xtradeb_apps_noble.list; \
    apt-get update -yq

# ---- Install chromium from noble PPA, keep other deps from jammy ----
RUN set -eux; \
    # -t noble pulls only the requested packages from noble; everything else stays on jammy
    apt-get install -yq --no-install-recommends \
      -t noble chromium chromium-common chromium-driver chromium-codecs-ffmpeg-extra; \
    # Cleanup
    apt-get purge -y --auto-remove curl gnupg || true; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/*

# ---- Entrypoint ----
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
USER selenium
EXPOSE 4444
ENTRYPOINT ["/entrypoint.sh"]
