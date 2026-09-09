FROM node:12-buster

ENV CI=true \
    YARN_CACHE_FOLDER=/usr/local/share/.cache/yarn

RUN set -eux; \
    printf '%s\n' \
      'deb https://archive.debian.org/debian buster main' \
      'deb https://archive.debian.org/debian buster-updates main' \
      'deb https://archive.debian.org/debian-security buster/updates main' \
      > /etc/apt/sources.list; \
    printf '%s\n' \
      'Acquire::Check-Valid-Until "false";' \
      'Acquire::Retries "3";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      > /etc/apt/apt.conf.d/99-retries; \
    installed=0; \
    for attempt in 1 2 3 4 5; do \
      if apt-get update && \
         apt-get install -y --no-install-recommends \
           bash \
           build-essential \
           ca-certificates \
           curl \
           git \
           patch \
           python2; then \
        installed=1; \
        break; \
      fi; \
      sleep 2; \
    done; \
    if [ "$installed" -ne 1 ]; then \
      echo "Unable to install system dependencies after retries" >&2; \
      exit 1; \
    fi; \
    npm config set python /usr/bin/python2; \
    rm -rf /var/lib/apt/lists/*

RUN set -eux; \
    cloned=0; \
    for attempt in 1 2 3 4 5; do \
      rm -rf /testbed; \
      if git \
        -c http.version=HTTP/1.1 \
        -c http.lowSpeedLimit=1000 \
        -c http.lowSpeedTime=60 \
        clone --no-tags https://github.com/ariakit/ariakit.git /testbed && \
        git -C /testbed checkout --detach d5e1e8f070c435acdbab92a95a938be23e5803ae; then \
        cloned=1; \
        break; \
      fi; \
      sleep 5; \
    done; \
    if [ "$cloned" -ne 1 ]; then \
      echo "Unable to clone the target repository after retries" >&2; \
      exit 1; \
    fi

WORKDIR /testbed

RUN yarn install --frozen-lockfile --non-interactive
