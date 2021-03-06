################################################################################
# This Dockerfile was generated from the template at distribution/src/docker/Dockerfile
#
# Beginning of multi stage Dockerfile
################################################################################

################################################################################
# Build stage 0 `builder`:
# Extract elasticsearch artifact
# Install required plugins
# Set gid=0 and make group perms==owner perms
################################################################################

FROM centos:8 AS builder

RUN for iter in {1..10}; do yum update --setopt=tsflags=nodocs -y &&     yum install --setopt=tsflags=nodocs -y gzip shadow-utils tar &&     yum clean all && exit_code=0 && break || exit_code=$? && echo "yum error: retry $iter in 10s" && sleep 10; done;     (exit $exit_code)

ENV PATH /usr/share/elasticsearch/bin:$PATH

RUN groupadd -g 1000 elasticsearch &&     adduser -u 1000 -g 1000 -d /usr/share/elasticsearch elasticsearch

WORKDIR /usr/share/elasticsearch

RUN cd /opt && curl --retry 8 -s -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.6.1-no-jdk-linux-x86_64.tar.gz && cd -

RUN tar zxf /opt/elasticsearch-7.6.1-no-jdk-linux-x86_64.tar.gz --strip-components=1
RUN grep ES_DISTRIBUTION_TYPE=tar /usr/share/elasticsearch/bin/elasticsearch-env     && sed -i -e 's/ES_DISTRIBUTION_TYPE=tar/ES_DISTRIBUTION_TYPE=docker/' /usr/share/elasticsearch/bin/elasticsearch-env
RUN curl -sL https://corretto.aws/downloads/latest/amazon-corretto-11-aarch64-linux-jdk.tar.gz | tar -C /usr/share/elasticsearch -zxf - && \
    mv /usr/share/elasticsearch/amazon* /usr/share/elasticsearch/jdk
RUN mkdir -p config data logs
RUN chmod 0775 config data logs
COPY config/elasticsearch-nosecurity.yml config/elasticsearch.yml
COPY config/log4j2.properties config/
RUN chmod 0660 config/elasticsearch.yml config/log4j2.properties

################################################################################
# Build stage 1 (the actual elasticsearch image):
# Copy elasticsearch from stage 0
# Add entrypoint
################################################################################

FROM centos:8 AS base

ENV ELASTIC_CONTAINER true

RUN for iter in {1..10}; do yum update --setopt=tsflags=nodocs -y &&     yum install --setopt=tsflags=nodocs -y nc shadow-utils zip unzip &&     yum clean all && exit_code=0 && break || exit_code=$? && echo "yum error: retry $iter in 10s" && sleep 10; done;     (exit $exit_code)

RUN groupadd -g 1000 elasticsearch &&     adduser -u 1000 -g 1000 -G 0 -d /usr/share/elasticsearch elasticsearch &&     chmod 0775 /usr/share/elasticsearch &&     chgrp 0 /usr/share/elasticsearch

WORKDIR /usr/share/elasticsearch
COPY --from=builder --chown=1000:0 /usr/share/elasticsearch /usr/share/elasticsearch

# Replace OpenJDK's built-in CA certificate keystore with the one from the OS
# vendor. The latter is superior in several ways.
# REF: https://github.com/elastic/elasticsearch-docker/issues/171
RUN ln -sf /etc/pki/ca-trust/extracted/java/cacerts /usr/share/elasticsearch/jdk/lib/security/cacerts

ENV PATH /usr/share/elasticsearch/bin:$PATH

COPY --chown=1000:0 bin/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh

# Openshift overrides USER and uses ones with randomly uid>1024 and gid=0
# Allow ENTRYPOINT (and ES) to run even with a different user
RUN chgrp 0 /usr/local/bin/docker-entrypoint.sh &&     chmod g=u /etc/passwd &&     chmod 0775 /usr/local/bin/docker-entrypoint.sh

EXPOSE 9200 9300

LABEL org.label-schema.build-date="2020-02-29T00:15:25.529771Z"   org.label-schema.license="Elastic-License"   org.label-schema.name="Elasticsearch"   org.label-schema.schema-version="1.0"   org.label-schema.url="https://www.elastic.co/products/elasticsearch"   org.label-schema.usage="https://www.elastic.co/guide/en/elasticsearch/reference/index.html"   org.label-schema.vcs-ref="aa751e09be0a5072e8570670309b1f12348f023b"   org.label-schema.vcs-url="https://github.com/elastic/elasticsearch"   org.label-schema.vendor="Elastic"   org.label-schema.version="7.6.1"   org.opencontainers.image.created="2020-02-29T00:15:25.529771Z"   org.opencontainers.image.documentation="https://www.elastic.co/guide/en/elasticsearch/reference/index.html"   org.opencontainers.image.licenses="Elastic-License"   org.opencontainers.image.revision="aa751e09be0a5072e8570670309b1f12348f023b"   org.opencontainers.image.source="https://github.com/elastic/elasticsearch"   org.opencontainers.image.title="Elasticsearch"   org.opencontainers.image.url="https://www.elastic.co/products/elasticsearch"   org.opencontainers.image.vendor="Elastic"   org.opencontainers.image.version="7.6.1"

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
# Dummy overridable parameter parsed by entrypoint
CMD ["eswrapper"]

################################################################################
# Add Open Distro plugins
################################################################################
FROM base AS nosecurity

RUN elasticsearch-plugin install --batch https://github.com/opendistro-for-elasticsearch/alerting/releases/download/v1.6.0.0/opendistro_alerting-1.6.0.0.zip && \
    elasticsearch-plugin install --batch https://github.com/opendistro-for-elasticsearch/job-scheduler/releases/download/v1.6.0.0/opendistro-job-scheduler-1.6.0.0.zip && \
    elasticsearch-plugin install --batch https://github.com/opendistro-for-elasticsearch/index-management/releases/download/v1.6.0.0/opendistro_index_management-1.6.0.0.zip

################################################################################
# Add Open Distro security
################################################################################
FROM nosecurity

RUN curl -sL -o artifacts.zip https://github.com/opendistro-for-elasticsearch/security/releases/download/v1.6.0.0/artifacts.zip && \
    unzip -p artifacts.zip artifacts/opendistro_security-1.6.0.0.zip > plugin.zip && \
    elasticsearch-plugin install --batch file://$PWD/plugin.zip && \
    rm artifacts.zip plugin.zip && \
    chmod +x /usr/share/elasticsearch/plugins/opendistro_security/tools/install_demo_configuration.sh

COPY config/elasticsearch.yml config/elasticsearch.yml

################################################################################
# End of multi-stage Dockerfile
################################################################################
