FROM node:20-bookworm

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV DEBIAN_FRONTEND=noninteractive \
    CI=true \
    CYPRESS_INSTALL_BINARY=0 \
    PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1 \
    PUPPETEER_SKIP_DOWNLOAD=1

RUN sed -i \
      -e 's|http://deb.debian.org|https://deb.debian.org|g' \
      -e 's|http://security.debian.org|https://security.debian.org|g' \
      /etc/apt/sources.list.d/debian.sources && \
    printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries-timeouts && \
    for attempt in 1 2 3 4 5; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        git \
        patch \
        wget \
        python3 \
        make \
        g++; then \
        find /var/lib/apt/lists -mindepth 1 -delete; \
        break; \
      fi; \
      if [ "${attempt}" -eq 5 ]; then \
        echo "apt-get failed after ${attempt} attempts" >&2; \
        exit 1; \
      fi; \
      sleep "$((attempt * 5))"; \
    done

RUN mkdir -p /testbed && \
    git init /testbed && \
    git -C /testbed config http.version HTTP/1.1 && \
    git -C /testbed remote add origin https://github.com/microsoft/fluentui.git && \
    for attempt in 1 2 3 4 5; do \
      if git -C /testbed fetch --depth=1 origin d61ac31da90bf8f73f54262e196b3a83b9a85932; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 5 ]; then \
        echo "Git fetch failed after ${attempt} attempts" >&2; \
        exit 1; \
      fi; \
      sleep "$((attempt * 10))"; \
    done && \
    git -C /testbed checkout --detach d61ac31da90bf8f73f54262e196b3a83b9a85932 && \
    git -C /testbed submodule update --init --recursive

WORKDIR /testbed

RUN for attempt in 1 2 3; do \
      if node .yarn/releases/cli.js install \
        --frozen-lockfile \
        --network-timeout 600000; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 3 ]; then \
        echo "Yarn install failed after ${attempt} attempts" >&2; \
        exit 1; \
      fi; \
      sleep "$((attempt * 10))"; \
    done

CMD ["/bin/bash"]
