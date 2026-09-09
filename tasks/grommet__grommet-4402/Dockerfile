FROM node:12.22.12-bullseye

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::http::No-Cache "true";' \
      'Acquire::https::No-Cache "true";' \
      > /etc/apt/apt.conf.d/80-retries \
    && sed -i \
      -e 's|http://deb.debian.org|https://deb.debian.org|g' \
      -e 's|http://security.debian.org|https://security.debian.org|g' \
      /etc/apt/sources.list \
    && for attempt in 1 2 3; do \
      if apt-get update \
        && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
          ca-certificates \
          git \
          patch \
          curl \
          wget \
          build-essential \
          python3; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 3 ]; then \
        exit 1; \
      fi; \
      sleep "$((attempt * 5))"; \
    done \
    && apt-get clean \
    && find /var/lib/apt/lists -mindepth 1 -delete

ARG TARGET_COMMIT=b59cd644fd026d4d0befefe02f0b831d773bc4b8

RUN git clone https://github.com/grommet/grommet.git /testbed \
    && cd /testbed \
    && git checkout --detach "${TARGET_COMMIT}" \
    && test "$(git rev-parse HEAD)" = "${TARGET_COMMIT}"

WORKDIR /testbed

RUN yarn install --frozen-lockfile --non-interactive

CMD ["/bin/bash"]
