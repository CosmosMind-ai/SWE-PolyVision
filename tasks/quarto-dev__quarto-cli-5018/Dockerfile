FROM debian:bookworm-slim

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ARG DEBIAN_FRONTEND=noninteractive

RUN set -eux; \
    printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::http::No-Cache "true";' \
      'Acquire::https::No-Cache "true";' \
      > /etc/apt/apt.conf.d/80-retries; \
    success=0; \
    for attempt in 1 2 3 4 5; do \
      if apt-get update && \
         apt-get install -y --no-install-recommends ca-certificates; then \
        success=1; \
        break; \
      fi; \
      if [ "${attempt}" -eq 5 ]; then exit 1; fi; \
      sleep "$((attempt * 5))"; \
    done; \
    if [ "${success}" -ne 1 ]; then exit 1; fi; \
    update-ca-certificates

RUN set -eux; \
    sed -i \
      -e 's|http://deb.debian.org|https://deb.debian.org|g' \
      -e 's|http://security.debian.org|https://security.debian.org|g' \
      /etc/apt/sources.list.d/debian.sources; \
    success=0; \
    for attempt in 1 2 3 4 5; do \
      if apt-get update && \
         apt-get install -y --no-install-recommends \
           bash \
           bzip2 \
           curl \
           fontconfig \
           git \
           gzip \
           libfontconfig1 \
           libfreetype6 \
           libgcc-s1 \
           libstdc++6 \
           patch \
           perl \
           tar \
           unzip \
           wget \
           xz-utils \
           zlib1g; then \
        success=1; \
        break; \
      fi; \
      if [ "${attempt}" -eq 5 ]; then exit 1; fi; \
      sleep "$((attempt * 5))"; \
    done; \
    if [ "${success}" -ne 1 ]; then exit 1; fi; \
    apt-get clean; \
    find /var/lib/apt/lists -mindepth 1 -delete

RUN git clone https://github.com/quarto-dev/quarto-cli.git /testbed && \
    cd /testbed && \
    git -c advice.detachedHead=false checkout --detach 7f8bc06d2ddbc5ddd1b8ae21e63d4ed5ee0c5861 && \
    test "$(git rev-parse HEAD)" = "7f8bc06d2ddbc5ddd1b8ae21e63d4ed5ee0c5861"

WORKDIR /testbed

ENV QUARTO_NO_SYMLINK=1 \
    QUARTO_TESTS_NO_CONFIG=1 \
    QUARTO_TESTS_FORCE_NO_PIPENV=1 \
    DENO_NO_UPDATE_CHECK=1 \
    PATH="/testbed/package/dist/bin:/root/.TinyTeX/bin/universal-linux:/root/.TinyTeX/bin/x86_64-linux:${PATH}"

RUN ./configure.sh

RUN package/dist/bin/quarto install --no-prompt --update-path tinytex
