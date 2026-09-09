FROM node:16.20.2-bullseye

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN if [ -f /etc/apt/sources.list ]; then \
      sed -i 's|http://deb.debian.org|https://deb.debian.org|g; s|http://security.debian.org|https://security.debian.org|g' /etc/apt/sources.list; \
    fi && \
    printf '%s\n' \
      'Acquire::Retries "3";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries && \
    for attempt in 1 2 3; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
        ca-certificates \
        curl \
        git \
        patch \
        python3 \
        wget; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 3 ]; then exit 1; fi; \
      sleep "$((attempt * 5))"; \
    done && \
    apt-get clean && \
    find /var/lib/apt/lists -mindepth 1 -delete

RUN git clone https://github.com/grommet/grommet.git /testbed && \
    cd /testbed && \
    git checkout --detach 739c72f7dfbaf4adfdf26377b3c704a3a3463faa

WORKDIR /testbed

RUN yarn install --frozen-lockfile --network-timeout 600000

CMD ["/bin/bash"]
