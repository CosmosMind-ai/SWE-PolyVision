FROM gpt56-sol-batch01-base-ionic-team__ionic-framework-28049:latest

USER root
WORKDIR /testbed
RUN git checkout --force aa9ea60357c369452bc121c0d663be7dbf3da28e
RUN rm -rf \
  /testbed/core/.stencil \
  /testbed/core/components \
  /testbed/core/css \
  /testbed/core/dist \
  /testbed/core/hydrate \
  /testbed/core/loader
COPY test_patch.diff /opt/eval-assets/test_patch.diff
