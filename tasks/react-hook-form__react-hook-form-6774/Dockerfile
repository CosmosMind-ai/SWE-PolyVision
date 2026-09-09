FROM node:16.20.2-bullseye

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      > /etc/apt/apt.conf.d/80-retries \
    && sed -i \
      -e 's|http://deb.debian.org|https://deb.debian.org|g' \
      -e 's|http://security.debian.org|https://security.debian.org|g' \
      /etc/apt/sources.list \
    && for attempt in 1 2 3; do \
         if apt-get update \
           && apt-get install -y --no-install-recommends \
                ca-certificates \
                curl \
                git \
                patch \
                build-essential; then \
           break; \
         fi; \
         if [ "${attempt}" -eq 3 ]; then exit 1; fi; \
         sleep $((attempt * 5)); \
       done \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN git clone https://github.com/react-hook-form/react-hook-form.git /testbed \
    && cd /testbed \
    && git checkout --detach c124399551e15ebc2d19a5ba94f336db3f02564e

WORKDIR /testbed

ENV CYPRESS_INSTALL_BINARY=0

RUN yarn install --frozen-lockfile --non-interactive
