FROM circleci/node:10.16-browsers

USER root
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV DEBIAN_FRONTEND=noninteractive \
    PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true

RUN printf '%s\n' \
        'Acquire::Retries "3";' \
        'Acquire::http::Timeout "30";' \
        'Acquire::https::Timeout "30";' \
        'Acquire::http::Pipeline-Depth "0";' \
        'Acquire::https::Pipeline-Depth "0";' \
        'Acquire::Check-Valid-Until "false";' \
        > /etc/apt/apt.conf.d/80-retries \
    && printf '%s\n' \
        'deb http://archive.debian.org/debian stretch main' \
        > /etc/apt/sources.list \
    && for attempt in 1 2 3; do \
        if apt-get -o Dir::Etc::sourceparts="-" update \
            && apt-get -o Dir::Etc::sourceparts="-" install -y --no-install-recommends \
                apt-transport-https \
                ca-certificates; then \
            break; \
        fi; \
        if [ "${attempt}" -eq 3 ]; then \
            echo "HTTPS transport bootstrap failed after 3 attempts" >&2; \
            exit 1; \
        fi; \
        sleep "$((attempt * 5))"; \
    done \
    && printf '%s\n' \
        'deb https://archive.debian.org/debian stretch main' \
        > /etc/apt/sources.list \
    && for attempt in 1 2 3; do \
        if apt-get -o Dir::Etc::sourceparts="-" update \
            && apt-get -o Dir::Etc::sourceparts="-" install -y --no-install-recommends \
                build-essential \
                curl \
                git \
                libgl1-mesa-dev \
                libglew-dev \
                libglu1-mesa-dev \
                libx11-dev \
                libxext-dev \
                libxi-dev \
                patch \
                pkg-config \
                python \
                wget; then \
            break; \
        fi; \
        if [ "${attempt}" -eq 3 ]; then \
            echo "Dependency installation failed after 3 attempts" >&2; \
            exit 1; \
        fi; \
        sleep "$((attempt * 5))"; \
    done \
    && apt-get clean

RUN git clone https://github.com/mapbox/mapbox-gl-js.git /testbed \
    && cd /testbed \
    && git checkout --detach 4e12ca76c45ef3923456f6245fb4565674fdf44f

WORKDIR /testbed

RUN yarn install --frozen-lockfile --network-timeout 600000

CMD ["/bin/bash"]
