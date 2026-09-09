FROM debian:bookworm-slim

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV DEBIAN_FRONTEND=noninteractive \
    FIREFOX_BIN=/usr/bin/firefox-esr \
    DISPLAY=:99 \
    npm_config_cafile=/etc/ssl/certs/ca-certificates.crt \
    npm_config_audit=false \
    npm_config_fund=false \
    npm_config_fetch_retries=5 \
    npm_config_fetch_retry_mintimeout=10000 \
    npm_config_fetch_retry_maxtimeout=60000

RUN printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::http::No-Cache "true";' \
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
    && if [ -f /etc/apt/sources.list ]; then \
         sed -i \
           -e 's|http://deb.debian.org|https://deb.debian.org|g' \
           -e 's|http://security.debian.org|https://security.debian.org|g' \
           /etc/apt/sources.list; \
       fi \
    && if [ -f /etc/apt/sources.list.d/debian.sources ]; then \
         sed -i \
           -e 's|http://deb.debian.org|https://deb.debian.org|g' \
           -e 's|http://security.debian.org|https://security.debian.org|g' \
           /etc/apt/sources.list.d/debian.sources; \
       fi \
    && install_ok=0 \
    && for attempt in 1 2 3 4 5; do \
         if apt-get update \
           && apt-get install -y --no-install-recommends \
             bash \
             curl \
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

ARG NODE_VERSION=4.9.1

RUN curl --fail --location --retry 5 --retry-delay 3 \
      "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.xz" \
      --output "/tmp/node-v${NODE_VERSION}-linux-x64.tar.xz" \
    && curl --fail --location --retry 5 --retry-delay 3 \
      "https://nodejs.org/dist/v${NODE_VERSION}/SHASUMS256.txt" \
      --output /tmp/SHASUMS256.txt \
    && cd /tmp \
    && grep " node-v${NODE_VERSION}-linux-x64.tar.xz\$" SHASUMS256.txt \
      | sha256sum --check --strict - \
    && tar -xJf "node-v${NODE_VERSION}-linux-x64.tar.xz" \
      --directory /usr/local \
      --strip-components=1 \
    && node --version \
    && npm --version

ARG TARGET_COMMIT=b33aff20919ca65e7914aa368fb99961e36e89be

RUN git clone https://github.com/chartjs/Chart.js.git /testbed \
    && cd /testbed \
    && git checkout --detach "${TARGET_COMMIT}" \
    && test "$(git rev-parse HEAD)" = "${TARGET_COMMIT}"

WORKDIR /testbed

RUN npm install --unsafe-perm

CMD ["/bin/bash"]
