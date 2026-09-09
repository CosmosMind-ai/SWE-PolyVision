FROM mcr.microsoft.com/playwright:v1.37.1-jammy

ENV DEBIAN_FRONTEND=noninteractive
ENV PLAYWRIGHT_BROWSERS_PATH=/ms-playwright

RUN set -eux; \
    for source_file in \
      /etc/apt/sources.list.d/nodesource.list \
      /etc/apt/sources.list.d/nodesource.sources; do \
      if [ -f "$source_file" ]; then \
        mv "$source_file" "${source_file}.disabled"; \
      fi; \
    done; \
    printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries; \
    sed -i \
      's|http://archive.ubuntu.com|https://archive.ubuntu.com|g; s|http://security.ubuntu.com|https://security.ubuntu.com|g' \
      /etc/apt/sources.list; \
    success=0; \
    for attempt in 1 2 3 4 5; do \
      if apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        git \
        patch \
        wget; then \
        success=1; \
        break; \
      fi; \
      sleep $((attempt * 5)); \
    done; \
    if [ "$success" -ne 1 ]; then exit 1; fi; \
    apt-get clean; \
    find /var/lib/apt/lists -mindepth 1 -delete

RUN git clone https://github.com/ionic-team/ionic-framework.git /testbed && \
    cd /testbed && \
    git checkout --detach 314055cf7a66a58e4e88c22e6e755fadd494ed70

WORKDIR /testbed

RUN cd core && \
    npm ci && \
    npm run build -- --ci

CMD ["/bin/bash"]
