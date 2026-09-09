FROM node:22-bookworm-slim

ENV DEBIAN_FRONTEND=noninteractive \
    CI=true \
    NODE_OPTIONS=--max-old-space-size=8192 \
    npm_config_update_notifier=false

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
    sed -i \
      's|http://deb.debian.org|https://deb.debian.org|g; s|http://security.debian.org|https://security.debian.org|g' \
      /etc/apt/sources.list.d/debian.sources; \
    for attempt in 1 2 3; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        bash \
        curl \
        git \
        patch \
        python3 \
        python3-setuptools \
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
    corepack prepare pnpm@10.14.0 --activate; \
    test "$(pnpm --version)" = "10.14.0"

RUN set -eux; \
    for attempt in 1 2 3; do \
      if git clone https://github.com/directus/directus.git /testbed; then \
        break; \
      fi; \
      if [ "$attempt" -eq 3 ]; then exit 1; fi; \
      sleep $((attempt * 5)); \
    done; \
    cd /testbed; \
    git checkout --detach f7ae717e6ec7086571d9bb2040bb0b32c6a087b4; \
    test "$(git rev-parse HEAD)" = "f7ae717e6ec7086571d9bb2040bb0b32c6a087b4"

WORKDIR /testbed

RUN pnpm install --recursive --frozen-lockfile

RUN npm_config_workspace_concurrency=2 pnpm run build

CMD ["/bin/bash"]
