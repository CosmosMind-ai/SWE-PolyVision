FROM node:22-bookworm

ENV DEBIAN_FRONTEND=noninteractive \
    NODE_OPTIONS=--max-old-space-size=8192 \
    npm_config_workspace_concurrency=2

RUN set -eux; \
    printf '%s\n' \
      'Acquire::Retries "3";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      > /etc/apt/apt.conf.d/99retries; \
    success=0; \
    for attempt in 1 2 3; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        bash \
        ca-certificates \
        curl \
        git \
        patch \
        python3 \
        build-essential; then \
        success=1; \
        break; \
      fi; \
      echo "apt attempt ${attempt} failed, retrying..." >&2; \
      sleep 5; \
    done; \
    if [ "${success}" -ne 1 ]; then \
      echo "apt installation failed after retries" >&2; \
      exit 1; \
    fi; \
    rm -rf /var/lib/apt/lists/*

RUN git clone https://github.com/directus/directus.git /testbed \
    && cd /testbed \
    && git checkout 54a35009d15177d6a9dde22c67ac2b9c5036a74e

WORKDIR /testbed

RUN corepack enable \
    && corepack prepare pnpm@10.27.0 --activate \
    && pnpm install --frozen-lockfile \
    && pnpm run build
