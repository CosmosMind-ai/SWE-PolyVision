FROM gpt56-sol-batch01-base-ionic-team__ionic-framework-28049:latest AS node18_runtime
FROM gpt56-sol-batch01-base-ckeditor__ckeditor5-19107:latest

USER root

COPY --from=node18_runtime /usr/bin/node /usr/bin/node
COPY --from=node18_runtime /usr/lib/node_modules/ /usr/lib/node_modules/
RUN ln -sf ../lib/node_modules/npm/bin/npm-cli.js /usr/bin/npm \
 && ln -sf ../lib/node_modules/npm/bin/npx-cli.js /usr/bin/npx \
 && ln -sf ../lib/node_modules/corepack/dist/corepack.js /usr/bin/corepack

ENV DEBIAN_FRONTEND=noninteractive
ENV CHROME_BIN=/usr/bin/chromium
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true
ENV PATH=/usr/bin:/usr/local/bin:/usr/local/sbin:/usr/sbin:/sbin:/bin

WORKDIR /
RUN rm -rf /testbed
COPY repo /testbed

WORKDIR /testbed

RUN node --version \
 && npm --version \
 && npm install --global pnpm@7.9.0 \
 && pnpm install --frozen-lockfile

COPY eval-assets /opt/eval-assets
