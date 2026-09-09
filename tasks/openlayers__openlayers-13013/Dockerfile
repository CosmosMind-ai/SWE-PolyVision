FROM node:16.20.2-bullseye

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV DEBIAN_FRONTEND=noninteractive \
    CI=true

RUN set -eux; \
    sed -i 's|http://deb.debian.org|https://deb.debian.org|g; s|http://security.debian.org|https://security.debian.org|g' /etc/apt/sources.list; \
    printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries; \
    apt_succeeded=0; \
    for attempt in 1 2 3 4 5; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        wget \
        git \
        patch \
        openssh-client \
        python3 \
        make \
        g++ \
        fonts-liberation \
        libasound2 \
        libatk-bridge2.0-0 \
        libatk1.0-0 \
        libcups2 \
        libdbus-1-3 \
        libdrm2 \
        libgbm1 \
        libglib2.0-0 \
        libgtk-3-0 \
        libnspr4 \
        libnss3 \
        libx11-6 \
        libx11-xcb1 \
        libxcb1 \
        libxcomposite1 \
        libxcursor1 \
        libxdamage1 \
        libxext6 \
        libxfixes3 \
        libxi6 \
        libxrandr2 \
        libxrender1 \
        libxshmfence1 \
        libxss1 \
        libxtst6 \
        xdg-utils; then \
        apt_succeeded=1; \
        break; \
      fi; \
      if [ "${attempt}" -eq 5 ]; then exit 1; fi; \
      sleep "$((attempt * 3))"; \
    done; \
    [ "${apt_succeeded}" -eq 1 ]; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/*

RUN git clone https://github.com/openlayers/openlayers.git /testbed && \
    cd /testbed && \
    git checkout --detach a715b1f5c6a64f80298fd424306f2bafb9752e4c

WORKDIR /testbed

RUN npm ci
