FROM node:22.12.0-bookworm

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV DEBIAN_FRONTEND=noninteractive \
    CI=1 \
    CHROME_BIN=/usr/local/bin/chromium-headless \
    PUPPETEER_SKIP_DOWNLOAD=true \
    YARN_CACHE_FOLDER=/tmp/yarn-cache

RUN sed -i \
        -e 's|http://deb.debian.org|https://deb.debian.org|g' \
        -e 's|http://security.debian.org|https://security.debian.org|g' \
        /etc/apt/sources.list.d/debian.sources \
    && printf '%s\n' \
        'Acquire::Retries "5";' \
        'Acquire::http::Timeout "30";' \
        'Acquire::https::Timeout "30";' \
        'Acquire::http::Pipeline-Depth "0";' \
        'Acquire::https::Pipeline-Depth "0";' \
        'Acquire::http::No-Cache "true";' \
        'Acquire::https::No-Cache "true";' \
        'Acquire::BrokenProxy "true";' \
        > /etc/apt/apt.conf.d/80-retries \
    && for attempt in 1 2 3; do \
        if apt-get update \
            && apt-get install -y --no-install-recommends \
                build-essential \
                ca-certificates \
                chromium \
                curl \
                fonts-liberation \
                fonts-noto-color-emoji \
                git \
                libgbm1 \
                libgtk-3-0 \
                libnss3 \
                libxss1 \
                patch \
                python3 \
                wget \
                xauth \
                xdg-utils \
                xvfb; then \
            break; \
        fi; \
        if [ "$attempt" -eq 3 ]; then \
            exit 1; \
        fi; \
        sleep $((attempt * 5)); \
    done \
    && apt-get clean \
    && find /var/lib/apt/lists -mindepth 1 -delete

RUN install -d /usr/local/libexec \
    && mv /usr/bin/chromium /usr/local/libexec/chromium \
    && printf '%s\n' \
        '#!/bin/bash' \
        'exec /usr/local/libexec/chromium --no-sandbox --disable-dev-shm-usage "$@"' \
        > /usr/bin/chromium \
    && printf '%s\n' \
        '#!/bin/bash' \
        'exec /usr/bin/chromium "$@"' \
        > /usr/local/bin/chromium-headless \
    && chmod 0755 /usr/bin/chromium /usr/local/bin/chromium-headless

RUN yarn_version="$(yarn --version)" \
    && [[ "$yarn_version" == 1.* ]]

RUN git clone --filter=blob:none --no-checkout \
        https://github.com/ckeditor/ckeditor5.git /testbed \
    && cd /testbed \
    && git checkout --detach 88c7c7890f65c45f80344b4095d1defec1375284 \
    && git config --system --add safe.directory /testbed \
    && chown -R node:node /testbed \
    && install -d -o node -g node /tmp/yarn-cache

WORKDIR /testbed

USER node

RUN yarn install \
        --non-interactive \
        --network-timeout 600000 \
    && yarn cache clean
