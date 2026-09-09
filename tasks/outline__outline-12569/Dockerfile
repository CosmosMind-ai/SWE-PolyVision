FROM node:24.16.0-bookworm

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN set -eux; \
    printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries; \
    sed -i \
      -e 's|http://deb.debian.org|https://deb.debian.org|g' \
      -e 's|http://security.debian.org|https://security.debian.org|g' \
      /etc/apt/sources.list.d/debian.sources; \
    for attempt in 1 2 3; do \
      if apt-get update && \
        DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
          bash \
          build-essential \
          ca-certificates \
          cmake \
          curl \
          git \
          patch \
          pkg-config \
          postgresql \
          postgresql-contrib \
          python3 \
          redis-server \
          wget; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 3 ]; then \
        exit 1; \
      fi; \
      sleep "$((attempt * 5))"; \
    done; \
    rm -rf /var/lib/apt/lists/*

RUN set -eux; \
    service postgresql start; \
    su postgres -c "psql --set ON_ERROR_STOP=1 --command=\"ALTER USER postgres WITH PASSWORD 'password';\""; \
    su postgres -c "createdb outline_test"; \
    service postgresql stop

RUN git clone --no-checkout https://github.com/outline/outline.git /testbed && \
    cd /testbed && \
    git checkout --detach 8464d995899c84917aad8d085a967776568748fc && \
    test "$(git rev-parse HEAD)" = "8464d995899c84917aad8d085a967776568748fc"

WORKDIR /testbed

RUN corepack enable && \
    corepack prepare yarn@4.11.0 --activate && \
    yarn install --immutable

ENV NODE_ENV=test \
    DATABASE_URL=postgres://postgres:password@127.0.0.1:5432/outline_test \
    REDIS_URL=redis://127.0.0.1:6379 \
    URL=http://localhost:3000 \
    NODE_OPTIONS=--max-old-space-size=8192 \
    SECRET_KEY=F0E5AD933D7F6FD8F4DBB3E038C501C052DC0593C686D21ACB30AE205D2F634B \
    UTILS_SECRET=123456 \
    SLACK_VERIFICATION_TOKEN=123456 \
    SMTP_USERNAME=localhost
