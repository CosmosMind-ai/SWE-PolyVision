FROM node:24.11.0-bookworm-slim

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
    for attempt in 1 2 3 4 5; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
        ca-certificates \
        curl \
        git \
        make \
        patch \
        python3 \
        ripgrep \
        wget; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 5 ]; then \
        echo "apt dependency installation failed after 5 attempts" >&2; \
        exit 1; \
      fi; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

RUN git clone --filter=blob:none --no-checkout \
      https://github.com/grafana/grafana.git /testbed && \
    git -C /testbed fetch --depth=1 origin \
      e4b0f5c7992827a6ca327b94bcdd7833e2ddcf97 && \
    git -C /testbed checkout --detach \
      e4b0f5c7992827a6ca327b94bcdd7833e2ddcf97

WORKDIR /testbed

ENV CI=true \
    NODE_OPTIONS=--max_old_space_size=8000 \
    YARN_ENABLE_GLOBAL_CACHE=false \
    YARN_ENABLE_MIRROR=false \
    YARN_NM_MODE=hardlinks-local \
    PUPPETEER_SKIP_DOWNLOAD=true \
    CYPRESS_INSTALL_BINARY=0

RUN corepack enable && \
    yarn install --immutable --inline-builds

CMD ["/bin/bash"]
