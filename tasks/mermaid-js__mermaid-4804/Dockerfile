FROM node:18.17.1-bookworm

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV DEBIAN_FRONTEND=noninteractive \
    NODE_OPTIONS=--max_old_space_size=4096 \
    CYPRESS_INSTALL_BINARY=0 \
    CYPRESS_CACHE_FOLDER=/testbed/.cache/Cypress \
    HUSKY=0

RUN set -eux; \
    printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries; \
    sed -i 's|http://deb.debian.org|https://deb.debian.org|g; s|http://security.debian.org|https://security.debian.org|g' \
      /etc/apt/sources.list.d/debian.sources; \
    for attempt in 1 2 3; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        git \
        patch \
        wget \
        build-essential \
        python3; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 3 ]; then exit 1; fi; \
      sleep "$((attempt * 5))"; \
    done; \
    apt-get clean; \
    find /var/lib/apt/lists -mindepth 1 -delete

RUN corepack enable && corepack prepare pnpm@8.7.0 --activate

RUN git clone https://github.com/mermaid-js/mermaid.git /testbed && \
    cd /testbed && \
    git checkout --detach 6e51f8fd9840a788bcca1f2cd2aabd78bcf3b74a

WORKDIR /testbed

RUN pnpm install --frozen-lockfile
