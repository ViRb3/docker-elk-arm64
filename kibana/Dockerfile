#
# ** THIS IS AN AUTO-GENERATED FILE **
#

################################################################################
# Build stage 0
# Extract Kibana and make various file manipulations.
################################################################################
FROM centos:7 AS prep_files
# Add tar and gzip
RUN yum update -y && yum install -y tar gzip && yum clean all
RUN cd /opt && curl --retry 8 -s -L -O https://artifacts.elastic.co/downloads/kibana/kibana-7.6.1-linux-x86_64.tar.gz && cd -
RUN mkdir /usr/share/kibana
WORKDIR /usr/share/kibana
RUN tar --strip-components=1 -zxf /opt/kibana-7.6.1-linux-x86_64.tar.gz

# replace amd64 node with arm64
RUN rm -rf /usr/share/kibana/node/* && \
    curl -sL https://nodejs.org/dist/v10.19.0/node-v10.19.0-linux-arm64.tar.gz | tar -C /usr/share/kibana/node/ --strip-components=1 -xzf -

# Ensure that group permissions are the same as user permissions.
# This will help when relying on GID-0 to run Kibana, rather than UID-1000.
# OpenShift does this, for example.
# REF: https://docs.openshift.org/latest/creating_images/guidelines.html
RUN chmod -R g=u /usr/share/kibana
RUN find /usr/share/kibana -type d -exec chmod g+s {} \;

################################################################################
# Build stage 1
# Copy prepared files from the previous stage and complete the image.
################################################################################
FROM centos:7 AS base
EXPOSE 5601

# Add Reporting dependencies.
RUN yum update -y && yum install -y fontconfig freetype shadow-utils && yum clean all

# Add an init process, check the checksum to make sure it's a match
RUN curl -L -o /usr/local/bin/dumb-init https://github.com/Yelp/dumb-init/releases/download/v1.2.2/dumb-init_1.2.2_arm64
RUN echo "45b1bbf56cc03edda81e4220535a025bfe3ed6e93562222b9be4471005b3eeb3  /usr/local/bin/dumb-init" | sha256sum -c -
RUN chmod +x /usr/local/bin/dumb-init


# Bring in Kibana from the initial stage.
COPY --from=prep_files --chown=1000:0 /usr/share/kibana /usr/share/kibana
WORKDIR /usr/share/kibana
RUN ln -s /usr/share/kibana /opt/kibana

ENV ELASTIC_CONTAINER true
ENV PATH=/usr/share/kibana/bin:$PATH

# Set some Kibana configuration defaults.
COPY --chown=1000:0 config/kibana.yml /usr/share/kibana/config/kibana.yml

# Add the launcher/wrapper script. It knows how to interpret environment
# variables and translate them to Kibana CLI options.
COPY --chown=1000:0 bin/kibana-docker /usr/local/bin/

# Ensure gid 0 write permissions for OpenShift.
RUN chmod g+ws /usr/share/kibana && find /usr/share/kibana -gid 0 -and -not -perm /g+w -exec chmod g+w {} \;

# Provide a non-root user to run the process.
RUN groupadd --gid 1000 kibana && useradd --uid 1000 --gid 1000 --home-dir /usr/share/kibana --no-create-home kibana
USER kibana

LABEL org.label-schema.schema-version="1.0" org.label-schema.vendor="Elastic" org.label-schema.name="kibana" org.label-schema.version="7.6.1" org.label-schema.url="https://www.elastic.co/products/kibana" org.label-schema.vcs-url="https://github.com/elastic/kibana" org.label-schema.license="Elastic License" org.label-schema.usage="https://www.elastic.co/guide/en/kibana/index.html" org.label-schema.build-date="2020-02-29T01:05:44.941Z" license="Elastic License"

ENTRYPOINT ["/usr/local/bin/dumb-init", "--"]

CMD ["/usr/local/bin/kibana-docker"]

################################################################################
# Add Open Distro plugins
################################################################################
FROM base AS nosecurity

RUN kibana-plugin install https://github.com/opendistro-for-elasticsearch/alerting-kibana-plugin/releases/download/v1.6.0.0/opendistro-alerting-1.6.0.0.zip && \
    kibana-plugin install https://github.com/opendistro-for-elasticsearch/index-management-kibana-plugin/releases/download/v1.6.0.0/opendistro_index_management_kibana-1.6.0.0.zip

################################################################################
# Add Open Distro security
################################################################################
FROM nosecurity

RUN kibana-plugin install https://github.com/opendistro-for-elasticsearch/security-kibana-plugin/releases/download/v1.6.0.0/opendistro_security_kibana_plugin-1.6.0.0.zip

################################################################################
# End of multi-stage Dockerfile
################################################################################
