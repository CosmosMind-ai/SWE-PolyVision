FROM node:14.21.3-bullseye

USER root

RUN set -eux; \
    printf '%s\n' \
        'Acquire::Retries "5";' \
        'Acquire::http::Timeout "30";' \
        'Acquire::https::Timeout "30";' \
        'Acquire::http::Pipeline-Depth "0";' \
        'Acquire::https::Pipeline-Depth "0";' \
        'Acquire::http::No-Cache "true";' \
        'Acquire::BrokenProxy "true";' \
        > /etc/apt/apt.conf.d/80-network-retries; \
    sed -i \
        -e 's|http://deb.debian.org|https://deb.debian.org|g' \
        -e 's|http://security.debian.org|https://security.debian.org|g' \
        /etc/apt/sources.list; \
    installed=0; \
    for attempt in 1 2 3 4 5; do \
        if apt-get update && \
           DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
               build-essential \
               ca-certificates \
               chromium \
               curl \
               fonts-dejavu-core \
               fonts-liberation \
               git \
               patch \
               python3 \
               python3-pytest \
               wget \
               xauth \
               xvfb; then \
            installed=1; \
            break; \
        fi; \
        sleep "$((attempt * 5))"; \
    done; \
    [ "$installed" -eq 1 ] || exit 1; \
    apt-get clean; \
    mkdir -p /testbed; \
    chown node:node /testbed; \
    git config --system --add safe.directory /testbed

ENV CHROME_BIN=/usr/bin/chromium \
    npm_config_fetch_retries=5 \
    npm_config_fetch_retry_mintimeout=10000 \
    npm_config_fetch_retry_maxtimeout=60000

USER node
WORKDIR /testbed

RUN set -eux; \
    fetched=0; \
    for attempt in 1 2 3 4 5; do \
        find /testbed -mindepth 1 -delete; \
        git init /testbed; \
        git remote add origin https://github.com/plotly/plotly.js.git; \
        if git fetch --depth 1 origin b70ef1d7b482109cb6a85f83a4d4ca86093b9ab7 && \
           git checkout --detach FETCH_HEAD; then \
            fetched=1; \
            break; \
        fi; \
        sleep "$((attempt * 5))"; \
    done; \
    [ "$fetched" -eq 1 ] || exit 1; \
    test "$(git rev-parse HEAD)" = "b70ef1d7b482109cb6a85f83a4d4ca86093b9ab7"

RUN npm install --ignore-scripts --no-audit --no-fund && \
    npm run pretest
