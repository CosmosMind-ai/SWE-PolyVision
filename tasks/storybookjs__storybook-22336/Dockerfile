FROM node:16.20.1-bullseye

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV DEBIAN_FRONTEND=noninteractive \
    CI=true \
    NODE_OPTIONS=--max_old_space_size=6144

RUN set -eux; \
    printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries; \
    sed -i \
      's|http://deb.debian.org|https://deb.debian.org|g; s|http://security.debian.org|https://security.debian.org|g' \
      /etc/apt/sources.list; \
    for attempt in 1 2 3; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        bash \
        build-essential \
        ca-certificates \
        curl \
        git \
        patch \
        python3 \
        wget; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 3 ]; then exit 1; fi; \
      sleep $((attempt * 5)); \
    done; \
    apt-get clean; \
    find /var/lib/apt/lists -mindepth 1 -delete

RUN set -eux; \
    target_commit="3e32fc62fdb93c78bb5c4ee5ce76a10d1b4cb7b6"; \
    clone_dir=""; \
    for attempt in 1 2 3 4 5; do \
      candidate="/tmp/storybook-clone-${attempt}"; \
      if git -c http.version=HTTP/1.1 clone \
        --depth=1 \
        --filter=blob:none \
        --no-checkout \
        https://github.com/storybookjs/storybook.git \
        "${candidate}"; then \
        clone_dir="${candidate}"; \
        break; \
      fi; \
      if [ "${attempt}" -eq 5 ]; then exit 1; fi; \
      sleep $((attempt * 5)); \
    done; \
    mv "${clone_dir}" /testbed; \
    cd /testbed; \
    git config http.version HTTP/1.1; \
    for attempt in 1 2 3 4 5; do \
      if git fetch --no-tags --depth=1 origin "${target_commit}"; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 5 ]; then exit 1; fi; \
      sleep $((attempt * 5)); \
    done; \
    git checkout --detach "${target_commit}"; \
    test "$(git rev-parse HEAD)" = "${target_commit}"

WORKDIR /testbed

RUN set -eux; \
    for attempt in 1 2 3; do \
      if cd /testbed/code && node ../.yarn/releases/yarn-3.5.1.cjs install --immutable; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 3 ]; then exit 1; fi; \
      sleep $((attempt * 5)); \
    done
