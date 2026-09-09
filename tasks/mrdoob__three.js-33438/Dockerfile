FROM node:24-bookworm

ENV DEBIAN_FRONTEND=noninteractive \
    npm_config_fetch_retries=5 \
    npm_config_fetch_retry_mintimeout=20000 \
    npm_config_fetch_retry_maxtimeout=120000

RUN set -eux; \
    sed -i 's|http://deb.debian.org|https://deb.debian.org|g; s|http://security.debian.org|https://security.debian.org|g' /etc/apt/sources.list.d/debian.sources; \
    printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-network-retries; \
    for attempt in 1 2 3; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        git \
        patch \
        wget \
        python3 \
        python-is-python3 \
        python3-pip \
        python3-pytest \
        python3-venv \
        xauth \
        xdg-utils \
        xvfb \
        mesa-vulkan-drivers \
        libvulkan1 \
        fonts-dejavu-core \
        fonts-liberation \
        fonts-noto-color-emoji \
        libasound2 \
        libatk-bridge2.0-0 \
        libatk1.0-0 \
        libcairo2 \
        libcups2 \
        libdbus-1-3 \
        libexpat1 \
        libfontconfig1 \
        libgbm1 \
        libglib2.0-0 \
        libgtk-3-0 \
        libnspr4 \
        libnss3 \
        libpango-1.0-0 \
        libpangocairo-1.0-0 \
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
        libxss1 \
        libxtst6; then \
        break; \
      fi; \
      if [ "$attempt" -eq 3 ]; then exit 1; fi; \
      sleep "$((attempt * 5))"; \
    done; \
    apt-get clean; \
    find /var/lib/apt/lists -mindepth 1 -delete

RUN git clone --filter=blob:none https://github.com/mrdoob/three.js.git /testbed && \
    cd /testbed && \
    git checkout --detach 5f343712b4f88b4bc827759c9357d4c17c75d6e7

WORKDIR /testbed

RUN set -eux; \
    for attempt in 1 2 3; do \
      if npm ci; then \
        break; \
      fi; \
      if [ "$attempt" -eq 3 ]; then exit 1; fi; \
      sleep "$((attempt * 10))"; \
    done
