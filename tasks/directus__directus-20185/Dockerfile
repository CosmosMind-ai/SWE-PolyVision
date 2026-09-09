FROM node:18.20.8-bookworm-slim

ENV DEBIAN_FRONTEND=noninteractive \
    CI=true \
    NODE_OPTIONS=--max-old-space-size=8192 \
    npm_config_update_notifier=false \
    npm_config_fetch_retries=5 \
    npm_config_fetch_retry_mintimeout=10000 \
    npm_config_fetch_retry_maxtimeout=60000

RUN set -eux; \
    printf '%s\n' \
      'Acquire::Retries "3";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries; \
    for attempt in 1 2 3; do \
      if apt-get update && apt-get install -y --no-install-recommends ca-certificates; then \
        break; \
      fi; \
      if [ "$attempt" -eq 3 ]; then exit 1; fi; \
      sleep $((attempt * 5)); \
    done; \
    for source_file in /etc/apt/sources.list /etc/apt/sources.list.d/debian.sources; do \
      if [ -f "$source_file" ]; then \
        sed -i \
          's|http://deb.debian.org|https://deb.debian.org|g; s|http://security.debian.org|https://security.debian.org|g' \
          "$source_file"; \
      fi; \
    done; \
    for attempt in 1 2 3; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        bash \
        curl \
        git \
        patch \
        python3 \
        make \
        g++ \
        pkg-config; then \
        break; \
      fi; \
      if [ "$attempt" -eq 3 ]; then exit 1; fi; \
      sleep $((attempt * 5)); \
    done; \
    apt-get clean; \
    find /var/lib/apt/lists -mindepth 1 -delete

RUN set -eux; \
    for attempt in 1 2 3; do \
      if npm install --global corepack@0.31.0; then \
        break; \
      fi; \
      if [ "$attempt" -eq 3 ]; then exit 1; fi; \
      sleep $((attempt * 5)); \
    done; \
    corepack enable; \
    corepack prepare pnpm@8.9.0 --activate; \
    test "$(pnpm --version)" = "8.9.0"

RUN set -eux; \
    for attempt in 1 2 3; do \
      if git clone --filter=blob:none --no-checkout https://github.com/directus/directus.git /testbed; then \
        break; \
      fi; \
      if [ -d /testbed ]; then \
        find /testbed -mindepth 1 -delete; \
        rmdir /testbed; \
      fi; \
      if [ "$attempt" -eq 3 ]; then exit 1; fi; \
      sleep $((attempt * 5)); \
    done; \
    cd /testbed; \
    git checkout --detach 9d80bd899be567042410e7254fd1287853e9580c; \
    test "$(git rev-parse HEAD)" = "9d80bd899be567042410e7254fd1287853e9580c"

WORKDIR /testbed

RUN set -eux; \
    sed -i \
      's|npm:@paescuj/eslint-plugin-prettier@4.3.0-1|4.2.1|' \
      docs/package.json; \
    pnpm install --filter '@directus/app...' --no-frozen-lockfile

RUN pnpm --filter '@directus/app^...' --workspace-concurrency=1 run build

CMD ["/bin/bash"]
