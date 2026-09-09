FROM node:12.22.12-bullseye

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV DEBIAN_FRONTEND=noninteractive \
    CI=1 \
    HUSKY_SKIP_INSTALL=1 \
    CHROME_BIN=/usr/local/bin/chromium-no-sandbox

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
        'Acquire::http::No-Cache "true";' \
        'Acquire::https::No-Cache "true";' \
        'Acquire::BrokenProxy "true";' \
        > /etc/apt/apt.conf.d/80-retries \
    && for attempt in 1 2 3; do \
        if apt-get update \
            && apt-get install -y --no-install-recommends \
                ca-certificates \
                chromium \
                curl \
                fonts-liberation \
                fonts-noto-color-emoji \
                g++ \
                git \
                make \
                patch \
                python3 \
                wget \
                xauth \
                xdg-utils \
                xvfb; then \
            break; \
        fi; \
        if [ "${attempt}" -eq 3 ]; then \
            exit 1; \
        fi; \
        sleep $((attempt * 5)); \
    done \
    && apt-get clean \
    && find /var/lib/apt/lists -mindepth 1 -delete

RUN printf '%s\n' \
        '#!/bin/bash' \
        'exec /usr/bin/chromium --no-sandbox --disable-dev-shm-usage "$@"' \
        > /usr/local/bin/chromium-no-sandbox \
    && chmod 0755 /usr/local/bin/chromium-no-sandbox

RUN yarn --version

RUN git clone --filter=blob:none --no-checkout \
        https://github.com/ckeditor/ckeditor5.git /testbed \
    && git -C /testbed checkout --detach \
        db9d1eb7c4afc13ab07495bc14c4cdb8714cee63

WORKDIR /testbed

RUN yarn install \
        --non-interactive \
        --ignore-engines \
        --network-timeout 600000 \
    && yarn cache clean
