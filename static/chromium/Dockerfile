ARG BASE_IMAGE_TAG=1.0

FROM base_image:$BASE_IMAGE_TAG

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y software-properties-common wget curl ca-certificates && \
    add-apt-repository ppa:xtradeb/apps && \
    apt-get update

RUN apt-get install -y chromium-browser chromium-driver chromium-common

RUN apt-get clean && rm -rf /var/lib/apt/lists/*

COPY entrypoint.sh /

RUN chmod +x /entrypoint.sh

USER selenium

EXPOSE 4444 5900

ENTRYPOINT ["/entrypoint.sh"]
