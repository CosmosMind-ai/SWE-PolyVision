FROM node:24.15.0-bookworm

ENV DEBIAN_FRONTEND=noninteractive

RUN set -eux; \
    sed -i 's|http://deb.debian.org|https://deb.debian.org|g; s|http://security.debian.org|https://security.debian.org|g' \
        /etc/apt/sources.list /etc/apt/sources.list.d/* 2>/dev/null || true; \
    printf '%s\n' \
        'Acquire::Retries "5";' \
        'Acquire::http::Timeout "30";' \
        'Acquire::https::Timeout "30";' \
        'Acquire::http::Pipeline-Depth "0";' \
        'Acquire::https::Pipeline-Depth "0";' \
        'Acquire::BrokenProxy "true";' \
        > /etc/apt/apt.conf.d/80-retries; \
    apt_succeeded=0; \
    for attempt in 1 2 3 4 5; do \
        if apt-get update && apt-get install -y --no-install-recommends \
            bash \
            git \
            patch \
            curl \
            wget \
            ca-certificates \
            python3 \
            build-essential \
            pkg-config \
            libx11-dev \
            libx11-xcb-dev \
            libxkbfile-dev \
            libnotify-bin \
            libkrb5-dev \
            libsecret-1-dev \
            xvfb \
            xauth \
            libgtk-3-0 \
            libgbm1 \
            libnss3 \
            libatk1.0-0 \
            libatk-bridge2.0-0 \
            libatspi2.0-0 \
            libcups2 \
            libdrm2 \
            libxcomposite1 \
            libxdamage1 \
            libxfixes3 \
            libxrandr2 \
            libxshmfence1 \
            libasound2 \
            libpango-1.0-0 \
            libcairo2 \
            libgl1 \
            libxcb-dri3-0 \
            fonts-liberation \
            dbus-x11 \
            rpm \
            bubblewrap \
            socat; then \
            apt_succeeded=1; \
            break; \
        fi; \
        if [ "$attempt" -eq 5 ]; then \
            echo "apt installation failed after 5 attempts" >&2; \
            exit 1; \
        fi; \
        sleep "$((attempt * 5))"; \
    done; \
    [ "$apt_succeeded" -eq 1 ] || exit 1; \
    apt-get clean

RUN install -d -o node -g node /testbed \
    && git config --system --add safe.directory /testbed

USER node
WORKDIR /testbed

RUN set -eux; \
    git init /testbed; \
    git config gc.auto 0; \
    git config maintenance.auto false; \
    git remote add origin https://github.com/microsoft/vscode.git; \
    git -c gc.auto=0 -c maintenance.auto=false fetch --depth=1 --no-tags origin fc8f90cdb7b4b5c8446d6e5212021c8a19d87163; \
    git checkout --detach FETCH_HEAD; \
    mkdir -p .git/refs/heads .git/refs/tags; \
    test "$(git rev-parse HEAD)" = "fc8f90cdb7b4b5c8446d6e5212021c8a19d87163"

ENV ELECTRON_SKIP_BINARY_DOWNLOAD=1 \
    PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1 \
    npm_config_fetch_retries=5 \
    npm_config_fetch_retry_mintimeout=20000 \
    npm_config_fetch_retry_maxtimeout=120000

RUN set -eux; \
    for attempt in 1 2 3 4 5; do \
        if npm ci; then \
            break; \
        fi; \
        if [ "$attempt" -eq 5 ]; then \
            echo "npm ci failed after 5 attempts" >&2; \
            exit 1; \
        fi; \
        sleep "$((attempt * 10))"; \
    done

RUN npm run electron

ENV DISPLAY=:99
