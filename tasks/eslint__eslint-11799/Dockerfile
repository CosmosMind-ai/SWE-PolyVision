FROM debian:bookworm-slim

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ARG NODE_VERSION=10.24.1
ARG TARGET_COMMIT=cb1922bdc07e58de0e55c13fd992dd8faf3292a4

ENV DEBIAN_FRONTEND=noninteractive \
    PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true \
    npm_config_audit=false \
    npm_config_fund=false \
    npm_config_update_notifier=false

RUN printf '%s\n' \
        'Acquire::Retries "5";' \
        'Acquire::http::Timeout "30";' \
        'Acquire::https::Timeout "30";' \
        'Acquire::http::Pipeline-Depth "0";' \
        'Acquire::https::Pipeline-Depth "0";' \
        'Acquire::BrokenProxy "true";' \
        > /etc/apt/apt.conf.d/80-retries \
    && success=0 \
    && for attempt in 1 2 3 4 5; do \
        if apt-get update \
            && apt-get install -y --no-install-recommends ca-certificates; then \
            success=1; \
            break; \
        fi; \
        if [ "$attempt" -lt 5 ]; then sleep "$((attempt * 5))"; fi; \
    done \
    && if [ "$success" -ne 1 ]; then exit 1; fi \
    && sed -i \
        's|http://deb.debian.org|https://deb.debian.org|g; s|http://security.debian.org|https://security.debian.org|g' \
        /etc/apt/sources.list.d/debian.sources \
    && success=0 \
    && for attempt in 1 2 3 4 5; do \
        if apt-get update \
            && apt-get install -y --no-install-recommends \
                bash \
                build-essential \
                curl \
                git \
                patch \
                python3 \
                wget \
                xz-utils; then \
            success=1; \
            break; \
        fi; \
        if [ "$attempt" -lt 5 ]; then sleep "$((attempt * 5))"; fi; \
    done \
    && if [ "$success" -ne 1 ]; then exit 1; fi \
    && apt-get clean \
    && find /var/lib/apt/lists -mindepth 1 -delete

RUN curl --fail --location --retry 5 --retry-delay 3 \
        "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.xz" \
        --output /tmp/node.tar.xz \
    && tar -xJf /tmp/node.tar.xz -C /usr/local --strip-components=1 \
    && rm /tmp/node.tar.xz \
    && node --version \
    && npm --version

RUN git clone https://github.com/eslint/eslint.git /testbed \
    && cd /testbed \
    && git checkout --detach "${TARGET_COMMIT}" \
    && test "$(git rev-parse HEAD)" = "${TARGET_COMMIT}"

WORKDIR /testbed

RUN npm config set fetch-retries 5 \
    && npm config set fetch-retry-mintimeout 10000 \
    && npm config set fetch-retry-maxtimeout 120000 \
    && success=0 \
    && for attempt in 1 2 3; do \
        if npm install --no-audit --no-fund; then \
            success=1; \
            break; \
        fi; \
        if [ "$attempt" -lt 3 ]; then sleep "$((attempt * 10))"; fi; \
    done \
    && if [ "$success" -ne 1 ]; then exit 1; fi

CMD ["/bin/bash"]
