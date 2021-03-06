FROM ruby:2.5.1-slim

LABEL maintainer="Splunk Inc. <DataEdge@splunk.com>"
LABEL Description="Splunk Connect for Kubernetes docker image" Vendor="Splunk Inc."

ARG VERSION
ARG FLUENT_SPLUNK_HEC_GEM_VERSION
ENV VERSION=${VERSION}
ENV FLUENT_SPLUNK_HEC_GEM_VERSION=${FLUENT_SPLUNK_HEC_GEM_VERSION}


# skip runtime bundler installation
ENV FLUENTD_DISABLE_BUNDLER_INJECTION 1

COPY *.gem /tmp/

RUN set -e \
 && apt-get update \
 && apt-get upgrade -y \
 && apt-get install -y --no-install-recommends libjemalloc1 jq \
 && buildDeps="make gcc g++ wget" \
 && apt-get install -y --no-install-recommends $buildDeps \
 && gem install -N fluentd -v "1.4.0" \
 && gem install -N fluent-plugin-prometheus -v "1.3.0" \
 && gem install -N fluent-plugin-jq -v "0.5.1" \
 && gem install -N oj -v "3.7.9" \
 && gem install -N fluent-plugin-splunk-hec:${FLUENT_SPLUNK_HEC_GEM_VERSION} \
 && gem install -N /tmp/*.gem \
 && dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')" \
 && wget -O /usr/bin/dumb-init https://github.com/Yelp/dumb-init/releases/download/v1.2.1/dumb-init_1.2.1_$dpkgArch \
 && chmod +x /usr/bin/dumb-init \
 && apt-get purge -y --auto-remove \
                  -o APT::AutoRemove::RecommendsImportant=false \
                  $buildDeps \
 && rm -rf /var/lib/apt/lists/* \
 && rm -rf /tmp/* /var/tmp/* $GEM_HOME/cache/*.gem \
 && mkdir -p /fluentd/etc

 # See https://packages.debian.org/stretch/amd64/libjemalloc1/filelist
ENV LD_PRELOAD "/usr/lib/x86_64-linux-gnu/libjemalloc.so.1"
COPY entrypoint.sh /fluentd/entrypoint.sh

ENTRYPOINT ["/fluentd/entrypoint.sh"]
CMD ["-h"]