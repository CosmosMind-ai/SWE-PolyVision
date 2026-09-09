FROM node:20.19.5-bookworm

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV DEBIAN_FRONTEND=noninteractive \
    CI=true \
    npm_config_audit=false \
    npm_config_fund=false \
    npm_config_fetch_retries=5 \
    npm_config_fetch_retry_mintimeout=20000 \
    npm_config_fetch_retry_maxtimeout=120000

RUN find /etc/apt -type f \( -name "*.list" -o -name "*.sources" \) \
      -exec sed -i \
        -e 's|http://deb.debian.org|https://deb.debian.org|g' \
        -e 's|http://security.debian.org|https://security.debian.org|g' {} + \
    && printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries \
    && for attempt in 1 2 3 4 5; do \
         if apt-get update \
           && apt-get install -y --no-install-recommends \
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
             libegl1 \
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
             libxtst6 \
             libgl1 \
             mesa-vulkan-drivers; then \
           break; \
         fi; \
         if [ "${attempt}" -eq 5 ]; then exit 1; fi; \
         sleep $((attempt * 5)); \
       done \
    && apt-get clean \
    && find /var/lib/apt/lists -mindepth 1 -delete

RUN git clone https://github.com/processing/p5.js.git /testbed \
    && cd /testbed \
    && git checkout --detach 9206f8d542b248d767ecdc652a26de988926952c \
    && test "$(git rev-parse HEAD)" = "9206f8d542b248d767ecdc652a26de988926952c"

WORKDIR /testbed

RUN npm ci
