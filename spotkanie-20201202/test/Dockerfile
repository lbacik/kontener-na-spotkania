FROM debian:buster

# hadolint ignore=DL3008
RUN apt-get update && apt-get -y install --no-install-recommends \
    procps \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

