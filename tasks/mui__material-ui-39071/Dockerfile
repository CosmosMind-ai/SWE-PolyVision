FROM node:18.18.0-bullseye

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN sed -i \
      -e 's|http://deb.debian.org|https://deb.debian.org|g' \
      -e 's|http://security.debian.org|https://security.debian.org|g' \
      /etc/apt/sources.list \
    && printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries \
    && for attempt in 1 2 3; do \
         if apt-get update \
           && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
             ca-certificates \
             curl \
             g++ \
             git \
             make \
             patch \
             python3 \
             wget; then \
           break; \
         fi; \
         if [ "${attempt}" -eq 3 ]; then exit 1; fi; \
         sleep $((attempt * 5)); \
       done \
    && apt-get clean \
    && find /var/lib/apt/lists -mindepth 1 -delete

RUN mkdir /testbed \
    && for attempt in 1 2 3 4 5; do \
         find /testbed -mindepth 1 -delete; \
         git -C /testbed init \
           && git -C /testbed remote add origin https://github.com/mui/material-ui.git \
           && git -C /testbed -c http.version=HTTP/1.1 fetch \
             --no-tags \
             --depth=1 \
             origin ba4c5596cdbbdf07e78cb10ca7231db9968812db \
           && git -C /testbed checkout --detach FETCH_HEAD \
           && break; \
         if [ "${attempt}" -eq 5 ]; then exit 1; fi; \
         sleep $((attempt * 5)); \
       done \
    && test "$(git -C /testbed rev-parse HEAD)" = "ba4c5596cdbbdf07e78cb10ca7231db9968812db"

WORKDIR /testbed

ENV PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1

RUN yarn --version \
    && yarn install --frozen-lockfile --non-interactive

CMD ["/bin/bash"]
