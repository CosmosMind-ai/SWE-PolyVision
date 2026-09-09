FROM node:12.22.12-bullseye

ENV DEBIAN_FRONTEND=noninteractive

RUN printf '%s\n' \
      'Acquire::Retries "5";' \
      'Acquire::http::Timeout "30";' \
      'Acquire::https::Timeout "30";' \
      'Acquire::http::Pipeline-Depth "0";' \
      'Acquire::http::No-Cache "true";' \
      'Acquire::BrokenProxy "true";' \
      > /etc/apt/apt.conf.d/80-retries \
    && sed -i \
      -e 's|http://deb.debian.org|https://deb.debian.org|g' \
      -e 's|http://security.debian.org|https://security.debian.org|g' \
      /etc/apt/sources.list \
    && installed=0 \
    && for attempt in 1 2 3; do \
         if apt-get update \
           && apt-get install -y --no-install-recommends \
             bash \
             build-essential \
             ca-certificates \
             curl \
             git \
             patch; then \
           installed=1; \
           break; \
         fi; \
         if [ "$attempt" -lt 3 ]; then sleep $((attempt * 5)); fi; \
       done \
    && if [ "$installed" -ne 1 ]; then exit 1; fi \
    && apt-get clean \
    && find /var/lib/apt/lists -mindepth 1 -delete

RUN git clone https://github.com/highlightjs/highlight.js.git /testbed \
    && cd /testbed \
    && git checkout --detach 5b413a6a243e279c40cda662362ad9095ef11d90

WORKDIR /testbed

RUN mkdir -p /opt/vendor /opt/deps \
    && curl --fail --location \
      --retry 5 --retry-delay 2 --retry-all-errors \
      https://registry.npmjs.org/gear/-/gear-0.9.7.tgz \
      --output /opt/vendor/gear-0.9.7.tgz \
    && curl --fail --location \
      --retry 5 --retry-delay 2 --retry-all-errors \
      https://registry.npmjs.org/gear-lib/-/gear-lib-0.9.2.tgz \
      --output /opt/vendor/gear-lib-0.9.2.tgz \
    && printf '%s  %s\n' \
      1ead19eee639319d8e2e655494c61bd8956e777f /opt/vendor/gear-0.9.7.tgz \
      bc8d461ebc81ecaffe99c1da82abe0f56eb93540 /opt/vendor/gear-lib-0.9.2.tgz \
      | sha1sum --check \
    && cp package.json package-lock.json /opt/deps/ \
    && node -e \
      "const fs=require('fs'); const file='/opt/deps/package.json'; const pkg=JSON.parse(fs.readFileSync(file)); pkg.devDependencies.gear='file:/opt/vendor/gear-0.9.7.tgz'; pkg.devDependencies['gear-lib']='file:/opt/vendor/gear-lib-0.9.2.tgz'; fs.writeFileSync(file, JSON.stringify(pkg, null, 2) + '\n');"

RUN cd /opt/deps \
    && npm config set fetch-retries 5 \
    && npm config set fetch-retry-mintimeout 10000 \
    && npm config set fetch-retry-maxtimeout 60000 \
    && npm install --no-audit \
    && node -e \
      "const expected={gear:'0.9.7','gear-lib':'0.9.2',mocha:'6.2.0',jsdom:'15.1.1',should:'13.2.3'}; for (const name of Object.keys(expected)) { const actual=require(name + '/package.json').version; if (actual !== expected[name]) throw new Error(name + ': expected ' + expected[name] + ', got ' + actual); }"

ENV NODE_PATH=/opt/deps/node_modules
ENV PATH=/opt/deps/node_modules/.bin:$PATH

RUN node tools/build.js -t node
