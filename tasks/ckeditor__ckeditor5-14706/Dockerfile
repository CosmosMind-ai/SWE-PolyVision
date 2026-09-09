FROM node:18.17.1-bookworm

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV DEBIAN_FRONTEND=noninteractive \
    CI=1 \
    FIREFOX_BIN=/usr/local/bin/firefox-headless \
    CHROME_BIN=/usr/local/bin/chromium-headless \
    YARN_CACHE_FOLDER=/tmp/yarn-cache

RUN for source_file in /etc/apt/sources.list /etc/apt/sources.list.d/debian.sources; do \
        if [ -f "$source_file" ]; then \
            sed -i \
                -e 's|http://deb.debian.org|https://deb.debian.org|g' \
                -e 's|http://security.debian.org|https://security.debian.org|g' \
                "$source_file"; \
        fi; \
    done \
    && printf '%s\n' \
        'Acquire::Retries "5";' \
        'Acquire::http::Timeout "30";' \
        'Acquire::https::Timeout "30";' \
        'Acquire::http::Pipeline-Depth "0";' \
        'Acquire::https::Pipeline-Depth "0";' \
        'Acquire::http::No-Cache "true";' \
        'Acquire::BrokenProxy "true";' \
        > /etc/apt/apt.conf.d/80-retries \
    && for attempt in 1 2 3; do \
        if apt-get update \
            && apt-get install -y --no-install-recommends \
                ca-certificates \
                chromium \
                curl \
                firefox-esr \
                fonts-liberation \
                fonts-noto-color-emoji \
                g++ \
                git \
                libgbm1 \
                libgtk-3-0 \
                libnss3 \
                libxss1 \
                make \
                patch \
                python3 \
                wget \
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
    && mv /usr/bin/chromium /usr/local/libexec/chromium-launcher \
    && printf '%s\n' \
        '#!/bin/bash' \
        'exec /usr/local/libexec/chromium-launcher --no-sandbox "$@"' \
        > /usr/local/bin/chromium-headless \
    && printf '%s\n' \
        '#!/bin/bash' \
        'exec /usr/bin/firefox-esr --headless "$@"' \
        > /usr/local/bin/firefox-headless \
    && chmod 0755 \
        /usr/local/bin/chromium-headless \
        /usr/local/bin/firefox-headless

RUN npm install --global --force yarn@1.22.19 \
    && yarn --version

RUN git clone --filter=blob:none --no-checkout \
        https://github.com/ckeditor/ckeditor5.git /testbed \
    && git -C /testbed checkout --detach \
        6bbde59ef34a9aa2ce4613f50dff74f24dec7c8b

WORKDIR /testbed

RUN yarn install \
        --non-interactive \
        --ignore-engines \
        --network-timeout 600000 \
    && yarn cache clean
