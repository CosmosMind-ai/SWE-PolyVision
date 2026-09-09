FROM node:14-bullseye-slim

ENV DEBIAN_FRONTEND=noninteractive \
    npm_config_audit=false \
    npm_config_fund=false

RUN set -eux; \
    printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      > /etc/apt/apt.conf.d/80-retries; \
    success=0; \
    for attempt in 1 2 3 4 5; do \
      if apt-get update && apt-get install -y --no-install-recommends ca-certificates; then \
        success=1; \
        break; \
      fi; \
      if [ "$attempt" -eq 5 ]; then exit 1; fi; \
      sleep "$((attempt * 2))"; \
    done; \
    [ "$success" -eq 1 ] || exit 1; \
    sed -i 's|http://|https://|g' /etc/apt/sources.list; \
    success=0; \
    for attempt in 1 2 3 4 5; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        bash \
        bzip2 \
        curl \
        git \
        patch \
        wget \
        xauth \
        xvfb \
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
        success=1; \
        break; \
      fi; \
      if [ "$attempt" -eq 5 ]; then exit 1; fi; \
      sleep "$((attempt * 2))"; \
    done; \
    [ "$success" -eq 1 ] || exit 1; \
    apt-get clean; \
    find /var/lib/apt/lists -mindepth 1 -delete

RUN set -eux; \
    target_commit=f3394441ad73b84fdc23790556c4aadcdcbd4b82; \
    mkdir -p /testbed; \
    git -C /testbed init; \
    git -C /testbed remote add origin https://github.com/bpmn-io/bpmn-js.git; \
    success=0; \
    for attempt in 1 2 3 4 5; do \
      if git -C /testbed -c http.version=HTTP/1.1 fetch --depth=1 origin "$target_commit"; then \
        success=1; \
        break; \
      fi; \
      if [ "$attempt" -eq 5 ]; then exit 1; fi; \
      sleep "$((attempt * 2))"; \
    done; \
    [ "$success" -eq 1 ] || exit 1; \
    git -C /testbed checkout --detach FETCH_HEAD; \
    test "$(git -C /testbed rev-parse HEAD)" = "$target_commit"; \
    chown -R node:node /testbed

WORKDIR /testbed

USER node

RUN npm ci
