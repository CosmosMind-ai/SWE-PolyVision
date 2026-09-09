FROM node:18.19.0-bookworm

ENV DEBIAN_FRONTEND=noninteractive \
    CI=true \
    PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1 \
    PNPM_HOME=/pnpm \
    PATH=/pnpm:$PATH

RUN set -eux; \
    printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries; \
    if [ -f /etc/apt/sources.list ]; then \
      sed -i 's|http://deb.debian.org|https://deb.debian.org|g; s|http://security.debian.org|https://security.debian.org|g' /etc/apt/sources.list; \
    fi; \
    if [ -f /etc/apt/sources.list.d/debian.sources ]; then \
      sed -i 's|http://deb.debian.org|https://deb.debian.org|g; s|http://security.debian.org|https://security.debian.org|g' /etc/apt/sources.list.d/debian.sources; \
    fi; \
    apt_succeeded=0; \
    for attempt in 1 2 3 4 5; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        bash \
        ca-certificates \
        curl \
        git \
        patch \
        wget \
        build-essential \
        python3; then \
        apt_succeeded=1; \
        break; \
      fi; \
      sleep "$((attempt * 5))"; \
    done; \
    if [ "$apt_succeeded" -ne 1 ]; then exit 1; fi; \
    apt-get clean; \
    find /var/lib/apt/lists -mindepth 1 -delete

RUN set -eux; \
    corepack enable; \
    corepack prepare pnpm@8.15.0 --activate; \
    pnpm config set fetch-retries 5; \
    pnpm config set fetch-retry-mintimeout 10000; \
    pnpm config set fetch-retry-maxtimeout 120000

RUN git clone --filter=blob:none --no-checkout https://github.com/mui/material-ui.git /testbed \
    && cd /testbed \
    && git checkout --detach 4e797158fe59df81bc604f57d2620726e93a82a8 \
    && test "$(git rev-parse HEAD)" = "4e797158fe59df81bc604f57d2620726e93a82a8"

WORKDIR /testbed

RUN pnpm install --frozen-lockfile

CMD ["/bin/bash"]
