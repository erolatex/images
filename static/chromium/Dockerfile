ARG BASE_IMAGE_TAG=1.0

FROM erolatex/selenoid_base_image:$BASE_IMAGE_TAG

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends gnupg dirmngr curl lsb-release && \
    mkdir -p /usr/share/keyrings && \
    curl -fsSL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x5301FA4FD93244FBC6F6149982BB6851C64F6880" | gpg --dearmor > /usr/share/keyrings/xtradeb_apps.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/xtradeb_apps.gpg] https://ppa.launchpadcontent.net/xtradeb/apps/ubuntu $(lsb_release -cs) main" > /etc/apt/sources.list.d/xtradeb_apps.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends chromium-browser chromium-driver chromium-common && \
    apt-get purge -y gnupg dirmngr curl && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

COPY entrypoint.sh /

RUN chmod +x /entrypoint.sh

USER selenium

EXPOSE 4444

ENTRYPOINT ["/entrypoint.sh"]
