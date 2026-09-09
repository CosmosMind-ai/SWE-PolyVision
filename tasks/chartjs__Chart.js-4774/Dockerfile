FROM debian:bullseye-slim

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ARG NODE_VERSION=10.24.1
ARG REPO_URL=https://github.com/chartjs/Chart.js.git
ARG TARGET_COMMIT=52145de5db5dc3bae093880429b35954f0034993

RUN printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::https::Pipeline-Depth "0";' \
      'Acquire::http::No-Cache "true";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries \
 && for attempt in 1 2 3; do \
      if apt-get update \
        && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
          ca-certificates; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 3 ]; then exit 1; fi; \
      sleep $((attempt * 5)); \
    done

RUN sed -i \
      -e 's|http://deb.debian.org|https://deb.debian.org|g' \
      -e 's|http://security.debian.org|https://security.debian.org|g' \
      /etc/apt/sources.list \
 && for attempt in 1 2 3; do \
      if apt-get update \
        && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
          bash \
          chromium \
          curl \
          dbus-x11 \
          firefox-esr \
          fonts-dejavu-core \
          fonts-liberation \
          g++ \
          git \
          make \
          patch \
          procps \
          python2 \
          wget \
          xauth \
          xvfb \
          xz-utils; then \
        break; \
      fi; \
      if [ "${attempt}" -eq 3 ]; then exit 1; fi; \
      sleep $((attempt * 5)); \
    done \
 && apt-get clean \
 && find /var/lib/apt/lists -mindepth 1 -delete

RUN curl --fail --location --retry 5 --retry-delay 3 \
      "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.xz" \
      -o /tmp/node.tar.xz \
 && curl --fail --location --retry 5 --retry-delay 3 \
      "https://nodejs.org/dist/v${NODE_VERSION}/SHASUMS256.txt" \
      -o /tmp/SHASUMS256.txt \
 && cd /tmp \
 && grep " node-v${NODE_VERSION}-linux-x64.tar.xz\$" SHASUMS256.txt \
      | sed "s|node-v${NODE_VERSION}-linux-x64.tar.xz|node.tar.xz|" \
      | sha256sum --check --strict - \
 && tar -xJf /tmp/node.tar.xz -C /usr/local --strip-components=1 \
 && find /tmp -maxdepth 1 -type f \
      \( -name 'node.tar.xz' -o -name 'SHASUMS256.txt' \) -delete \
 && node --version \
 && npm --version

ENV CHROME_BIN=/usr/bin/chromium
ENV FIREFOX_BIN=/usr/bin/firefox-esr
ENV DISPLAY=:99
ENV TRAVIS=true
ENV npm_config_audit=false
ENV npm_config_fund=false
ENV npm_config_update_notifier=false

RUN git clone "${REPO_URL}" /testbed \
 && cd /testbed \
 && git checkout --detach "${TARGET_COMMIT}" \
 && test "$(git rev-parse HEAD)" = "${TARGET_COMMIT}"

WORKDIR /testbed

RUN for attempt in 1 2 3; do \
      if npm install --unsafe-perm; then break; fi; \
      if [ "${attempt}" -eq 3 ]; then exit 1; fi; \
      sleep $((attempt * 5)); \
    done \
 && for attempt in 1 2 3; do \
      if npm install --global gulp-cli@1.4.0 --unsafe-perm; then break; fi; \
      if [ "${attempt}" -eq 3 ]; then exit 1; fi; \
      sleep $((attempt * 5)); \
    done \
 && gulp --version

CMD ["/bin/bash"]
