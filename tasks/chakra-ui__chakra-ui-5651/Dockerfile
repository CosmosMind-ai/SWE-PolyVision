FROM node:16.20.2-bullseye

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN set -eux; \
    printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries; \
    sed -i 's|http://deb.debian.org|https://deb.debian.org|g; s|http://security.debian.org|https://security.debian.org|g' /etc/apt/sources.list; \
    for attempt in 1 2 3 4 5; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        bash \
        ca-certificates \
        curl \
        wget \
        git \
        patch \
        build-essential \
        python3 \
        pkg-config; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 5 ]; then \
        echo "apt installation failed after 5 attempts" >&2; \
        exit 1; \
      fi; \
      apt-get clean; \
      sleep "$((attempt * 3))"; \
    done; \
    apt-get clean

RUN git clone https://github.com/chakra-ui/chakra-ui.git /testbed && \
    cd /testbed && \
    git checkout --detach d5807a4ef73baf0aa7687a144ca4a4474abe953b && \
    test "$(git rev-parse HEAD)" = "d5807a4ef73baf0aa7687a144ca4a4474abe953b"

WORKDIR /testbed

ENV CI=true
ENV HUSKY=0

RUN yarn config set network-timeout 600000 && \
    yarn install --frozen-lockfile --ignore-engines
