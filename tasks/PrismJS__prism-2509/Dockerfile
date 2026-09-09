FROM node:14.21.3-bullseye

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV DEBIAN_FRONTEND=noninteractive
ENV PRISM_COMMIT=bf4f323391d546d4b2e1a388f05c512a27491e9c

RUN sed -i 's|http://deb.debian.org|https://deb.debian.org|g; s|http://security.debian.org|https://security.debian.org|g' /etc/apt/sources.list \
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
            && apt-get install -y --no-install-recommends \
                bash \
                ca-certificates \
                curl \
                git \
                patch \
                wget; then \
            break; \
        fi; \
        if [ "${attempt}" -eq 3 ]; then \
            echo "apt installation failed after ${attempt} attempts" >&2; \
            exit 1; \
        fi; \
        sleep $((attempt * 5)); \
    done

RUN git clone https://github.com/PrismJS/prism.git /testbed \
    && cd /testbed \
    && git checkout --detach "${PRISM_COMMIT}" \
    && test "$(git rev-parse HEAD)" = "${PRISM_COMMIT}"

WORKDIR /testbed

RUN npm config set fetch-retries 5 \
    && npm config set fetch-retry-mintimeout 10000 \
    && npm config set fetch-retry-maxtimeout 120000 \
    && npm ci
