FROM node:12.22.12-bullseye-slim

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries \
    && for attempt in 1 2 3; do \
         if apt-get update \
           && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends ca-certificates; then \
           break; \
         fi; \
         if [ "${attempt}" -eq 3 ]; then exit 1; fi; \
         sleep $((attempt * 5)); \
       done \
    && sed -i \
         's|http://deb.debian.org|https://deb.debian.org|g; s|http://security.debian.org|https://security.debian.org|g' \
         /etc/apt/sources.list \
    && for attempt in 1 2 3; do \
         if apt-get update \
           && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
              bash \
              git \
              patch \
              curl \
              wget \
              chromium \
              fonts-liberation \
              build-essential \
              python3 \
              pkg-config \
              libcairo2-dev \
              libpango1.0-dev \
              libjpeg62-turbo-dev \
              libgif-dev \
              librsvg2-dev; then \
           break; \
         fi; \
         if [ "${attempt}" -eq 3 ]; then exit 1; fi; \
         sleep $((attempt * 5)); \
       done \
    && apt-get clean \
    && find /var/lib/apt/lists -mindepth 1 -delete \
    && printf '%s\n' \
         '#!/bin/bash' \
         'exec /usr/bin/chromium --no-sandbox --disable-dev-shm-usage "$@"' \
         > /usr/local/bin/chromium-for-puppeteer \
    && chmod 0755 /usr/local/bin/chromium-for-puppeteer

RUN for attempt in 1 2 3; do \
      if git clone https://github.com/apache/echarts.git /testbed; then break; fi; \
      if [ "${attempt}" -eq 3 ]; then exit 1; fi; \
      sleep $((attempt * 5)); \
    done \
    && cd /testbed \
    && git checkout --detach 6f78c0d5191378e604fc88eb07b7ffabdb69aa3e \
    && test "$(git rev-parse HEAD)" = "6f78c0d5191378e604fc88eb07b7ffabdb69aa3e" \
    && chown -R node:node /testbed

WORKDIR /testbed

ENV npm_config_python=/usr/bin/python3 \
    PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true \
    PUPPETEER_EXECUTABLE_PATH=/usr/local/bin/chromium-for-puppeteer

USER node

RUN npm config set fetch-retries 5 \
    && npm config set fetch-retry-mintimeout 10000 \
    && npm config set fetch-retry-maxtimeout 120000 \
    && npm ci \
    && npm install --no-save --no-package-lock puppeteer@5.5.0
