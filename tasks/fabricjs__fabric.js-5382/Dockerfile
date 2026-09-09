FROM node:10.24.1-buster

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::http::No-Cache "true";' \
      'Acquire::https::No-Cache "true";' \
      'Acquire::Check-Valid-Until "false";' \
      > /etc/apt/apt.conf.d/80-retries \
    && printf '%s\n' \
      'deb https://archive.debian.org/debian buster main' \
      'deb https://archive.debian.org/debian buster-updates main' \
      'deb https://archive.debian.org/debian-security buster/updates main' \
      > /etc/apt/sources.list \
    && find /etc/apt/sources.list.d -type f -name '*.list' -exec sed -i 's/^deb /# deb /' {} + \
    && for attempt in 1 2 3 4 5; do \
         if apt-get update \
           && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
             bash \
             build-essential \
             ca-certificates \
             curl \
             git \
             patch \
             wget \
             python2.7 \
             pkg-config \
             libcairo2-dev \
             libgif-dev \
             libjpeg-dev \
             libpng-dev \
             libpango1.0-dev \
             librsvg2-dev; then \
           break; \
         fi; \
         if [ "${attempt}" -eq 5 ]; then exit 1; fi; \
         sleep "$((attempt * 5))"; \
       done \
    && find /var/lib/apt/lists -mindepth 1 -delete

RUN git clone https://github.com/fabricjs/fabric.js.git /testbed \
    && cd /testbed \
    && git checkout --detach 3af833930f7e043163a94c0426d17efa6f6def9f \
    && test "$(git rev-parse HEAD)" = "3af833930f7e043163a94c0426d17efa6f6def9f"

WORKDIR /testbed

RUN npm config set python /usr/bin/python2.7 \
    && npm config set fetch-retries 5 \
    && npm config set fetch-retry-mintimeout 10000 \
    && npm config set fetch-retry-maxtimeout 120000 \
    && npm install --unsafe-perm \
    && node -e "require('canvas'); require('jsdom'); require('xmldom')"

CMD ["/bin/bash"]
