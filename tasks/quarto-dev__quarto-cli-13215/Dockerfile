FROM node:20-bookworm

ARG DEBIAN_FRONTEND=noninteractive
ARG TARGET_COMMIT=77a82a631bcc6219c4f7892f7d538f5e2b58cd57

RUN set -eux; \
    printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries; \
    find /etc/apt -type f \( -name '*.list' -o -name '*.sources' \) \
      -exec sed -i 's|http://deb.debian.org|https://deb.debian.org|g; s|http://security.debian.org|https://security.debian.org|g' {} +; \
    installed=0; \
    for attempt in 1 2 3 4 5; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        bash \
        ca-certificates \
        curl \
        wget \
        git \
        patch \
        unzip \
        xz-utils \
        libcurl4-openssl-dev \
        libxml2-utils \
        libharfbuzz-dev \
        libfribidi-dev \
        poppler-utils \
        fonts-dejavu-core; then \
        installed=1; \
        break; \
      fi; \
      apt-get clean; \
      sleep 5; \
    done; \
    if [ "$installed" -ne 1 ]; then exit 1; fi; \
    apt-get clean; \
    update-ca-certificates; \
    node --version; \
    npm --version

RUN printf '%s\n' \
      'retry = 5' \
      'retry-all-errors' \
      'connect-timeout = 30' \
      > /root/.curlrc

RUN git clone https://github.com/quarto-dev/quarto-cli.git /testbed && \
    cd /testbed && \
    git checkout --detach "$TARGET_COMMIT" && \
    test "$(git rev-parse HEAD)" = "$TARGET_COMMIT"

WORKDIR /testbed

RUN ./configure.sh

ENV PATH="/testbed/package/dist/bin:${PATH}" \
    QUARTO_TESTS_NO_CONFIG="true" \
    QUARTO_TESTS_FORCE_NO_VENV="true"

CMD ["/bin/bash"]
