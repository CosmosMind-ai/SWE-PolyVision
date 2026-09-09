FROM cypress/browsers:node12.13.0-chrome80-ff74

ENV DEBIAN_FRONTEND=noninteractive \
    npm_config_loglevel=warn \
    npm_config_progress=false \
    npm_config_unsafe_perm=true \
    TZ=Etc/UTC

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# The Cypress browser image already provides Chrome, Firefox, Xvfb, build tools,
# and their runtime libraries. Only ensure the small essential-tool set exists.
RUN find /etc/apt/sources.list /etc/apt/sources.list.d -type f -print0 2>/dev/null \
      | xargs -0 -r sed -i \
          -e '/google\.com\/linux\/chrome/d' \
          -e '/stretch-updates/d' \
          -e 's#http://deb\.debian\.org/debian#https://archive.debian.org/debian#g' \
          -e 's#https://deb\.debian\.org/debian#https://archive.debian.org/debian#g' \
          -e 's#http://security\.debian\.org/debian-security#https://archive.debian.org/debian-security#g' \
          -e 's#https://security\.debian\.org/debian-security#https://archive.debian.org/debian-security#g' \
    && printf '%s\n' \
         'Acquire::Check-Valid-Until "false";' \
         'Acquire::Retries "5";' \
         'Acquire::http::Timeout "30";' \
         'Acquire::https::Timeout "30";' \
         'Acquire::http::Pipeline-Depth "0";' \
         'Acquire::https::Pipeline-Depth "0";' \
         'Acquire::http::No-Cache "true";' \
         'Acquire::https::No-Cache "true";' \
         'Acquire::BrokenProxy "true";' \
         > /etc/apt/apt.conf.d/99cypress-network \
    && updated=0 \
    && for attempt in 1 2 3 4 5; do \
         if apt-get update; then updated=1; break; fi; \
         echo "apt-get update failed (attempt ${attempt}/5)" >&2; \
         sleep $((attempt * 3)); \
       done \
    && if [ "${updated}" -ne 1 ]; then \
         echo "apt-get update failed after 5 attempts" >&2; \
         exit 1; \
       fi \
    && installed=0 \
    && for attempt in 1 2 3 4 5; do \
         if apt-get install -y --no-install-recommends \
           bash \
           ca-certificates \
           curl \
           git \
           patch \
           wget; then \
             installed=1; \
             break; \
         fi; \
         echo "apt-get install failed (attempt ${attempt}/5)" >&2; \
         sleep $((attempt * 3)); \
       done \
    && if [ "${installed}" -ne 1 ]; then \
         echo "apt-get install failed after 5 attempts" >&2; \
         exit 1; \
       fi \
    && apt-get clean \
    && find /var/lib/apt/lists -mindepth 1 -delete

RUN cloned=0 \
    && for attempt in 1 2 3; do \
         if git clone https://github.com/cypress-io/cypress.git /testbed; then \
           cloned=1; \
           break; \
         fi; \
         echo "git clone failed (attempt ${attempt}/3)" >&2; \
         sleep $((attempt * 5)); \
       done \
    && if [ "${cloned}" -ne 1 ]; then \
         echo "git clone failed after 3 attempts" >&2; \
         exit 1; \
       fi \
    && cd /testbed \
    && git checkout --detach 7c5819393d5792c8d1bb79668823e423c4ec7527 \
    && test "$(git rev-parse HEAD)" = "7c5819393d5792c8d1bb79668823e423c4ec7527"

WORKDIR /testbed

RUN installed=0 \
    && for attempt in 1 2 3; do \
         if yarn --frozen-lockfile; then \
           installed=1; \
           break; \
         fi; \
         echo "yarn install failed (attempt ${attempt}/3)" >&2; \
         sleep $((attempt * 5)); \
       done \
    && if [ "${installed}" -ne 1 ]; then \
         echo "yarn install failed after 3 attempts" >&2; \
         exit 1; \
       fi
