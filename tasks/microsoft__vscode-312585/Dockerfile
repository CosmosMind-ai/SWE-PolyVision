FROM node:24.15.0-bookworm

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV DEBIAN_FRONTEND=noninteractive \
    ELECTRON_SKIP_BINARY_DOWNLOAD=1 \
    PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1 \
    npm_config_fetch_retries=5 \
    npm_config_fetch_retry_mintimeout=20000 \
    npm_config_fetch_retry_maxtimeout=120000 \
    npm_config_fetch_timeout=300000

RUN printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::No-Cache "true";' \
      'Acquire::https::No-Cache "true";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      > /etc/apt/apt.conf.d/80-retries \
    && sed -i 's|http://deb.debian.org|https://deb.debian.org|g; s|http://security.debian.org|https://security.debian.org|g' \
      /etc/apt/sources.list /etc/apt/sources.list.d/*.sources 2>/dev/null || true

RUN for attempt in 1 2 3; do \
      if apt-get update \
        && apt-get install -y --no-install-recommends \
          build-essential \
          ca-certificates \
          curl \
          git \
          libkrb5-dev \
          libsecret-1-dev \
          libx11-dev \
          libxkbfile-dev \
          patch \
          pkg-config \
          python3 \
          wget; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 3 ]; then exit 1; fi; \
      sleep $((attempt * 5)); \
    done \
    && apt-get clean \
    && find /var/lib/apt/lists -mindepth 1 -delete

RUN git clone --filter=blob:none --no-checkout \
      https://github.com/microsoft/vscode.git /testbed \
    && git -C /testbed checkout --detach 57aca5e8fdd82f329405161a407d231174e40608 \
    && test "$(git -C /testbed rev-parse HEAD)" = "57aca5e8fdd82f329405161a407d231174e40608"

WORKDIR /testbed

RUN for attempt in 1 2 3; do \
      if npm ci; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 3 ]; then exit 1; fi; \
      sleep $((attempt * 10)); \
    done
