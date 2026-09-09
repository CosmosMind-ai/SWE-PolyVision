FROM mcr.microsoft.com/playwright:v1.48.1-jammy

ENV DEBIAN_FRONTEND=noninteractive \
    HUSKY=0 \
    npm_config_fetch_retries=5 \
    npm_config_fetch_retry_mintimeout=20000 \
    npm_config_fetch_retry_maxtimeout=120000

RUN printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::http::No-Cache "true";' \
      'Acquire::https::No-Cache "true";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries \
    && sed -i \
      -e 's|http://archive.ubuntu.com|https://archive.ubuntu.com|g' \
      -e 's|http://security.ubuntu.com|https://security.ubuntu.com|g' \
      /etc/apt/sources.list \
    && for attempt in 1 2 3; do \
         if apt-get update \
           && apt-get install -y --no-install-recommends \
             bash \
             build-essential \
             ca-certificates \
             curl \
             git \
             patch \
             python3 \
             wget \
             xvfb; then \
           break; \
         fi; \
         if [ "$attempt" -eq 3 ]; then exit 1; fi; \
         sleep 5; \
       done \
    && apt-get clean \
    && find /var/lib/apt/lists -mindepth 1 -delete

ARG TARGET_COMMIT=e6d5c8e1891c22a03da72922955bcc6dbbd64eea

RUN git config --global http.version HTTP/1.1 \
    && git config --global http.lowSpeedLimit 1000 \
    && git config --global http.lowSpeedTime 60 \
    && mkdir /testbed \
    && cd /testbed \
    && git init \
    && git remote add origin https://github.com/ariakit/ariakit.git \
    && fetch_succeeded=0 \
    && for attempt in 1 2 3 4 5; do \
         if git fetch --no-tags --depth=1 origin "$TARGET_COMMIT"; then \
           fetch_succeeded=1; \
           break; \
         fi; \
         if [ "$attempt" -lt 5 ]; then sleep 5; fi; \
       done \
    && test "$fetch_succeeded" -eq 1 \
    && git checkout --detach FETCH_HEAD \
    && test "$(git rev-parse HEAD)" = "$TARGET_COMMIT"

WORKDIR /testbed

RUN npm ci --no-audit --no-fund
