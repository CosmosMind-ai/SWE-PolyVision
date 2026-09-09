FROM cypress/browsers:node12.14.1-chrome83-ff77

USER root
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN set -eux; \
    find /etc/apt -type f \( -name '*.list' -o -name '*.sources' \) -exec sed -i \
      -e '/^[[:space:]]*deb .*dl\.google\.com\/linux\/chrome/s/^/# disabled: /' \
      -e '/buster-updates/d' \
      -e 's|https\?://deb.debian.org/debian|https://archive.debian.org/debian|g' \
      -e 's|https\?://security.debian.org/debian-security|https://archive.debian.org/debian-security|g' \
      {} +; \
    find /etc/apt/sources.list.d -type f \( -iname '*google*' -o -iname '*chrome*' \) -delete 2>/dev/null || true; \
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
        python3 \
        wget; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 3 ]; then \
        exit 1; \
      fi; \
      sleep $((attempt * 5)); \
    done; \
    find /var/lib/apt/lists -mindepth 1 -delete

ENV CI=1 \
    PLATFORM=linux \
    COLUMNS=100 \
    LINES=24 \
    TERM=xterm \
    TZ=America/New_York \
    npm_config_python=/usr/bin/python

RUN set -eux; \
    commit=40dd31102c686ad89da1153f7abb7b41b8595e29; \
    acquired=0; \
    for attempt in 1 2 3; do \
      checkout_dir="$(mktemp -d /tmp/cypress-checkout.XXXXXX)"; \
      if git -C "${checkout_dir}" init \
        && git -C "${checkout_dir}" remote add origin https://github.com/cypress-io/cypress.git \
        && git -C "${checkout_dir}" fetch --depth=1 origin "${commit}" \
        && git -C "${checkout_dir}" checkout --detach FETCH_HEAD \
        && test "$(git -C "${checkout_dir}" rev-parse HEAD)" = "${commit}"; then \
        mv "${checkout_dir}" /testbed; \
        acquired=1; \
        break; \
      fi; \
      rm -rf "${checkout_dir}"; \
      if [ "${attempt}" -lt 3 ]; then \
        sleep $((attempt * 10)); \
      fi; \
    done; \
    test "${acquired}" -eq 1

WORKDIR /testbed

RUN set -eux; \
    test "$(node --version)" = "v12.14.1"; \
    install_status=1; \
    for attempt in 1 2 3; do \
      if yarn install \
        --frozen-lockfile \
        --network-timeout 600000; then \
        install_status=0; \
        break; \
      fi; \
      if [ "${attempt}" -eq 3 ]; then \
        exit 1; \
      fi; \
      sleep $((attempt * 10)); \
    done; \
    test "${install_status}" -eq 0; \
    yarn build-prod
