FROM node:14.21.3-bullseye

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN set -eux; \
    printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries; \
    sed -i \
      -e 's|http://deb.debian.org|https://deb.debian.org|g' \
      -e 's|http://security.debian.org|https://security.debian.org|g' \
      /etc/apt/sources.list; \
    success=0; \
    for attempt in 1 2 3 4 5; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        bash \
        build-essential \
        ca-certificates \
        curl \
        git \
        patch \
        python3 \
        wget; then \
        success=1; \
        break; \
      fi; \
      echo "apt attempt ${attempt} failed; retrying"; \
      sleep "$((attempt * 5))"; \
    done; \
    if [ "$success" -ne 1 ]; then exit 1; fi; \
    apt-get clean; \
    find /var/lib/apt/lists -mindepth 1 -delete

RUN if ! command -v yarn >/dev/null 2>&1; then \
      npm install --global yarn@1.22.22; \
    fi; \
    yarn --version

RUN git clone https://github.com/GoogleChrome/lighthouse.git /testbed && \
    cd /testbed && \
    git checkout --detach 3ad529d937a4aa5af4e68575c6780676edd23248

WORKDIR /testbed

ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=1
ENV PUPPETEER_SKIP_DOWNLOAD=1

RUN yarn install --frozen-lockfile --network-timeout 1000000
