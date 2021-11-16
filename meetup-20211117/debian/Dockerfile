FROM debian:bullseye

# hadolint ignore=DL3008
RUN apt-get update && apt-get -y install --no-install-recommends \
    procps \
    iproute2 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

