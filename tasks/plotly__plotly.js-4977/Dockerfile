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
        find /testbed -depth -mindepth 1 -delete; \
        if git clone --no-checkout https://github.com/plotly/plotly.js.git /testbed && \
           git -C /testbed fetch --depth 1 origin c925cd4688df2fc5fc9aacce1d87f36877ae5196 && \
           git -C /testbed checkout --detach c925cd4688df2fc5fc9aacce1d87f36877ae5196; then \
            fetched=1; \
            break; \
        fi; \
        sleep "$((attempt * 5))"; \
    done; \
    [ "$fetched" -eq 1 ] || exit 1; \
    test "$(git -C /testbed rev-parse HEAD)" = "c925cd4688df2fc5fc9aacce1d87f36877ae5196"

USER root
RUN mkdir -p /testbed/build/test_images && chown -R node:node /testbed
USER node

RUN npm ci --ignore-scripts --no-audit --no-fund && \
    npm run pretest
