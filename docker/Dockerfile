FROM alpine:3.8

LABEL maintainer="Splunk Inc. <DataEdge@splunk.com>"
LABEL Description="Splunk Connect for Kubernetes docker image" Vendor="Splunk Inc."

# fluent-plugin-kubernetes-objects
ENV KO_VERSION=1.1.2
# fluent-plugin-k8s-metrics-agg
ENV KMA_VERSION=1.1.2
# fluent-plugin-kubernetes-metrics
ENV KM_VERSION=1.1.2
# fluent-plugin-splunk-hec
ENV SH_VERSION=1.1.0

ENV DUMB_INIT_VERSION=1.2.1
ENV SU_EXEC_VERSION=0.2

ARG DEBIAN_FRONTEND=noninteractive
# Do not split this into multiple RUN!
# Docker creates a layer for every RUN-Statement
# therefore an 'apk delete' has no effect
RUN apk update \
 && apk upgrade \
 && apk add --no-cache \
        ca-certificates \
 && update-ca-certificates \
 && apk add --no-cache \
        ruby ruby-irb ruby-etc ruby-webrick ruby-json \
        su-exec==${SU_EXEC_VERSION}-r0 \
        dumb-init==${DUMB_INIT_VERSION}-r0 \
 && apk add --no-cache --virtual .build-deps \
        build-base \
        ruby-dev ruby-bundler ruby-rake wget gnupg git \
 && echo 'gem: --no-document' >> /etc/gemrc \
 && gem install -N \
         fluentd:1.4.0 \
         fluent-plugin-record-modifier:2.0.1 \
         fluent-plugin-splunk-hec:${SH_VERSION} \
         oj:3.7.9 \
         multi_json:1.13.1 \
         bigdecimal:1.4.3 \
         kubeclient:4.3.0 \
 && git clone https://github.com/splunk/fluent-plugin-kubernetes-metrics.git \
 && cd fluent-plugin-kubernetes-metrics \
 && git checkout tags/${KM_VERSION} \
 && gem build fluent-plugin-kubernetes-metrics \
 && gem install fluent-plugin-kubernetes-metrics-${KM_VERSION}.gem \
 && cd .. \
 && rm -rf fluent-plugin-kubernetes-metrics \
 && apk del .build-deps \
 && rm -rf /var/cache/apk/* \
 && rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem

ENV LD_PRELOAD="" \
    FLUENTD_CONF="fluent.conf" \
    FLUENTD_OPT="" \
    DUMB_INIT_SETSID=0

COPY entrypoint.sh /bin/entrypoint.sh
ENTRYPOINT ["/bin/bash", "/bin/entrypoint.sh"]

CMD exec fluentd -c /fluentd/etc/${FLUENTD_CONF} -p /fluentd/plugins $FLUENTD_OPT
