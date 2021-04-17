FROM debian:buster

# hadolint ignore=DL3008
RUN apt-get update && apt-get -y install --no-install-recommends \
    procps \
    netcat \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

EXPOSE 4444

CMD ["nc", "-lv", "-p", "4444"]
