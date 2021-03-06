FROM golang:1.8 AS prep_env2yaml

ADD env2yaml/env2yaml.go .

RUN go get gopkg.in/yaml.v2 && \
    mkdir env2yaml && \
    mv env2yaml.go env2yaml/ && \
    cd env2yaml/ && \
    go build

FROM centos:8 AS prep_files

RUN mkdir -p /usr/share/logstash

RUN curl -sL https://artifacts.elastic.co/downloads/logstash/logstash-7.6.1.tar.gz | \
    tar -C /usr/share/logstash --strip-components=1 -zxf -
# Bug: https://github.com/elastic/logstash/issues/10888
RUN yum install -y zip unzip && \
    mkdir -p /tmp/jruby-complete && \
    unzip -q /usr/share/logstash/logstash-core/lib/jars/jruby-complete-9.2.9.0.jar -d /tmp/jruby-complete && \
    cp /tmp/jruby-complete/META-INF/jruby.home/lib/ruby/stdlib/ffi/platform/aarch64-linux/{types,platform}.conf && \
    cd /tmp/jruby-complete && \
    zip -qr /tmp/jruby-complete-9.2.9.0.jar * && \
    cp -f /tmp/jruby-complete-9.2.9.0.jar /usr/share/logstash/logstash-core/lib/jars/jruby-complete-9.2.9.0.jar

# This Dockerfile was generated from templates/Dockerfile.j2
FROM centos:8

# Install Java and the "which" command, which is needed by Logstash's shell
# scripts.
RUN yum update -y && yum install -y java-11-openjdk-devel which && \
    yum clean all

# Provide a non-root user to run the process.
RUN groupadd --gid 1000 logstash && \
    adduser --uid 1000 --gid 1000 \
      --home-dir /usr/share/logstash --no-create-home \
      logstash

# Add Logstash itself.
COPY --from=prep_files --chown=1000:0 /usr/share/logstash /usr/share/logstash
RUN chmod -R g=u /usr/share/logstash && \
    find /usr/share/logstash -type d -exec chmod g+s {} \; && \
    ln -s /usr/share/logstash /opt/logstash

WORKDIR /usr/share/logstash

ENV ELASTIC_CONTAINER true
ENV PATH=/usr/share/logstash/bin:$PATH

# Provide a minimal configuration, so that simple invocations will provide
# a good experience.
ADD config/pipelines.yml config/pipelines.yml
ADD config/logstash-full.yml config/logstash.yml
ADD config/log4j2.properties config/
ADD pipeline/default.conf pipeline/logstash.conf
RUN chown --recursive logstash:root config/ pipeline/

# Ensure Logstash gets a UTF-8 locale by default.
ENV LANG='en_US.UTF-8' LC_ALL='en_US.UTF-8'

# Place the startup wrapper script.
ADD bin/docker-entrypoint /usr/local/bin/
RUN chmod 0755 /usr/local/bin/docker-entrypoint

USER 1000

COPY --from=prep_env2yaml --chown=logstash:root /go/env2yaml/env2yaml /usr/local/bin/

EXPOSE 9600 5044


LABEL org.label-schema.schema-version="1.0" \
  org.label-schema.vendor="Elastic" \
  org.label-schema.name="logstash" \
  org.label-schema.version="7.6.1" \
  org.label-schema.url="https://www.elastic.co/products/logstash" \
  org.label-schema.vcs-url="https://github.com/elastic/logstash" \
license="Elastic License"
ENTRYPOINT ["/usr/local/bin/docker-entrypoint"]
