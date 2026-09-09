FROM debian:bookworm-slim

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV DEBIAN_FRONTEND=noninteractive \
    CHROME_BIN=/usr/bin/chromium \
    FIREFOX_BIN=/usr/bin/firefox-esr \
    DISPLAY=:99.0 \
    npm_config_audit=false \
    npm_config_fund=false

RUN printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::http::No-Cache "true";' \
      'Acquire::https::No-Cache "true";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-network-retries \
    && bootstrap_ok=0 \
    && for attempt in 1 2 3 4 5; do \
         if apt-get update \
           && apt-get install -y --no-install-recommends ca-certificates; then \
           bootstrap_ok=1; \
           break; \
         fi; \
         sleep "$((attempt * 5))"; \
       done \
    && if [ "$bootstrap_ok" -ne 1 ]; then exit 1; fi \
    && find /etc/apt -type f \
         \( -name '*.list' -o -name '*.sources' \) \
         -exec sed -i \
           -e 's|http://deb.debian.org|https://deb.debian.org|g' \
           -e 's|http://security.debian.org|https://security.debian.org|g' \
           {} + \
    && install_ok=0 \
    && for attempt in 1 2 3 4 5; do \
         if apt-get update \
           && apt-get install -y --no-install-recommends \
             bash \
             chromium \
             curl \
             dbus-x11 \
             firefox-esr \
             fonts-dejavu-core \
             fonts-liberation \
             g++ \
             git \
             make \
             patch \
             procps \
             wget \
             xauth \
             xz-utils \
             xvfb; then \
           install_ok=1; \
           break; \
         fi; \
         sleep "$((attempt * 5))"; \
       done \
    && if [ "$install_ok" -ne 1 ]; then exit 1; fi \
    && apt-get clean

ARG NODE_VERSION=0.12.18

RUN curl --fail --location --retry 5 --retry-delay 3 \
      "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.gz" \
      --output "/tmp/node-v${NODE_VERSION}-linux-x64.tar.gz" \
    && curl --fail --location --retry 5 --retry-delay 3 \
      "https://nodejs.org/dist/v${NODE_VERSION}/SHASUMS256.txt" \
      --output /tmp/SHASUMS256.txt \
    && cd /tmp \
    && grep " node-v${NODE_VERSION}-linux-x64.tar.gz\$" SHASUMS256.txt \
      | sha256sum --check --strict - \
    && tar -xzf "node-v${NODE_VERSION}-linux-x64.tar.gz" \
      --directory /usr/local \
      --strip-components=1 \
    && node --version \
    && npm --version \
    && npm config set fetch-retries 5 \
    && npm config set fetch-retry-factor 2 \
    && npm config set fetch-retry-mintimeout 10000 \
    && npm config set fetch-retry-maxtimeout 60000

ARG TARGET_COMMIT=efced4780ce2697d0d44bf1e32a4e2efebaa18e2

RUN git clone https://github.com/chartjs/Chart.js.git /testbed \
    && cd /testbed \
    && git checkout --detach "${TARGET_COMMIT}" \
    && test "$(git rev-parse HEAD)" = "${TARGET_COMMIT}"

WORKDIR /testbed

RUN npm install --unsafe-perm

CMD ["/bin/bash"]
