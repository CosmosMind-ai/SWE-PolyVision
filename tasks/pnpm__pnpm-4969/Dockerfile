FROM node:14.19.0-bullseye

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN sed -i 's|http://deb.debian.org|https://deb.debian.org|g; s|http://security.debian.org|https://security.debian.org|g' /etc/apt/sources.list \
    && printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      > /etc/apt/apt.conf.d/80-retries \
    && for attempt in 1 2 3; do \
         apt-get update \
         && apt-get install -y --no-install-recommends \
           build-essential \
           ca-certificates \
           curl \
           git \
           patch \
           python3 \
           wget \
         && break; \
         if [ "${attempt}" = "3" ]; then exit 1; fi; \
         sleep $((attempt * 5)); \
       done \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN npm install --global pnpm@7.4.1

RUN git config --global core.autocrlf false \
    && git clone https://github.com/pnpm/pnpm.git /testbed \
    && cd /testbed \
    && git checkout --detach e9946fcfa7e1892fb550a2164c2040fea2574cdd

WORKDIR /testbed

RUN pnpm install --frozen-lockfile
