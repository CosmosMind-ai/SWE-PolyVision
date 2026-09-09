FROM cypress/browsers:node12.18.3-chrome83-ff77

USER root
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN set -eux; \
    . /etc/os-release; \
    case "${VERSION_CODENAME:-}" in \
      buster|stretch) \
        find /etc/apt/sources.list.d -type f -name '*.list' \
          -exec sed -i \
            -e '\|deb.debian.org/debian|s|^[[:space:]]*deb |# deb |' \
            -e '\|security.debian.org|s|^[[:space:]]*deb |# deb |' \
            {} +; \
        printf '%s\n' \
          "deb https://archive.debian.org/debian ${VERSION_CODENAME} main" \
          "deb https://archive.debian.org/debian-security ${VERSION_CODENAME}/updates main" \
          > /etc/apt/sources.list; \
        ;; \
      *) \
        sed -i \
          -e 's|http://deb.debian.org|https://deb.debian.org|g' \
          -e 's|http://security.debian.org|https://security.debian.org|g' \
          /etc/apt/sources.list; \
        ;; \
    esac; \
    find /etc/apt/sources.list.d -type f -iname '*google*' \
      -exec sed -i -e 's/^[[:space:]]*deb /# deb /' {} +; \
    printf '%s\n' \
      'Acquire::Retries "3";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::BrokenProxy "true";' \
      'Acquire::Check-Valid-Until "false";' \
      > /etc/apt/apt.conf.d/80-retries; \
    for attempt in 1 2 3; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        g++ \
        git \
        make \
        patch \
        python \
        wget; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 3 ]; then \
        exit 1; \
      fi; \
      sleep $((attempt * 5)); \
    done; \
    apt-get clean; \
    find /var/lib/apt/lists -mindepth 1 -delete

ENV CI=1 \
    TERM=xterm \
    COLUMNS=100 \
    LINES=24 \
    PLATFORM=linux \
    CYPRESS_KONFIG_ENV=production \
    npm_config_python=/usr/bin/python

RUN git clone --no-checkout https://github.com/cypress-io/cypress.git /testbed \
    && cd /testbed \
    && git checkout --detach 197f3097f7beaa09cd4dd1e2a0e9a39610f5ed69 \
    && test "$(git rev-parse HEAD)" = "197f3097f7beaa09cd4dd1e2a0e9a39610f5ed69"

WORKDIR /testbed

RUN set -eux; \
    test "$(node --version)" = "v12.18.3"; \
    yarn_cache_dir="$(mktemp -d /tmp/cypress-yarn-cache.XXXXXX)"; \
    for attempt in 1 2 3; do \
      if yarn install \
        --frozen-lockfile \
        --cache-folder "${yarn_cache_dir}" \
        --network-timeout 600000; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 3 ]; then \
        exit 1; \
      fi; \
      sleep $((attempt * 10)); \
    done; \
    find "${yarn_cache_dir}" -mindepth 1 -delete; \
    rmdir "${yarn_cache_dir}"; \
    yarn build-prod; \
    git diff --exit-code
