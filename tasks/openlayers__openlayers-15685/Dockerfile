FROM node:20-bookworm

ENV CI=true \
    DEBIAN_FRONTEND=noninteractive \
    PUPPETEER_CACHE_DIR=/root/.cache/puppeteer

RUN set -eux; \
    printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::http::No-Cache "true";' \
      'Acquire::https::No-Cache "true";' \
      > /etc/apt/apt.conf.d/80-network-reliability; \
    sed -i 's|http://deb.debian.org|https://deb.debian.org|g' /etc/apt/sources.list.d/debian.sources; \
    for attempt in 1 2 3 4 5; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        git \
        patch \
        wget \
        xdg-utils \
        fonts-liberation \
        libasound2 \
        libatk-bridge2.0-0 \
        libatk1.0-0 \
        libcairo2 \
        libcups2 \
        libdbus-1-3 \
        libdrm2 \
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
        libxkbcommon0 \
        libxrandr2 \
        libxrender1 \
        libxss1 \
        libxtst6; then \
        break; \
      fi; \
      if [ "$attempt" -eq 5 ]; then exit 1; fi; \
      apt-get clean; \
      sleep "$((attempt * 5))"; \
    done; \
    apt-get clean; \
    find /var/lib/apt/lists -mindepth 1 -delete

RUN git clone https://github.com/openlayers/openlayers.git /testbed \
    && cd /testbed \
    && git checkout --detach 7a88544d0f02253088bedaf8c0adf3a73d24739b \
    && test "$(git rev-parse HEAD)" = "7a88544d0f02253088bedaf8c0adf3a73d24739b"

WORKDIR /testbed

RUN npm config set fetch-retries 5 \
    && npm config set fetch-retry-mintimeout 10000 \
    && npm config set fetch-retry-maxtimeout 120000 \
    && npm ci

CMD ["/bin/bash"]
