# ELK stack Docker images for ARM64
> Elasticsearch, Logstash and Kibana Docker images for ARM64, including Open Distro plugins.

## Description
These releases include the [basic](https://www.elastic.co/subscriptions) X-Pack plugins, as well as the following [Open Distro](https://opendistro.github.io/for-elasticsearch/) plugins:
- Security - [Elasticsearch](https://github.com/opendistro-for-elasticsearch/security) | [Kibana](https://github.com/opendistro-for-elasticsearch/security-kibana-plugin)
- Alerting - [Elasticsearch](https://github.com/opendistro-for-elasticsearch/alerting) | [Kibana](https://github.com/opendistro-for-elasticsearch/alerting-kibana-plugin)
- Job Scheduler - [Elasticsearch](https://github.com/opendistro-for-elasticsearch/job-scheduler)
- Index Management - [Elasticsearch](https://github.com/opendistro-for-elasticsearch/index-management) | [Kibana](https://github.com/opendistro-for-elasticsearch/index-management-kibana-plugin)

## Downloads
There are two flavors of the Docker images - the normal one, which includes all Open Distro plugins, and `nosecurity`, which excludes the security plugin.

- https://hub.docker.com/repository/docker/virb3/elasticsearch
  - `docker pull virb3/elasticsearch:7-arm64`
  - `docker pull virb3/elasticsearch:7-nosecurity-arm64`
- https://hub.docker.com/repository/docker/virb3/logstash
  - `docker pull virb3/logstash:7-arm64`
  - `docker pull virb3/logstash:7-nosecurity-arm64`
- https://hub.docker.com/repository/docker/virb3/kibana
  - `docker pull virb3/kibana:7-arm64`

## Beats
Looking for pre-compiled [Beats](https://www.elastic.co/beats/)? You can find them [here](https://github.com/ViRb3/beats/releases/latest).

## Build
1. Initial base from official ELK stack Docker files:
   - https://github.com/elastic/dockerfiles
2. ARM patches inspired by [gagara](https://github.com/gagara):
   - https://github.com/gagara/docker-elk-arm64
3. env2yaml from [Logstash](https://github.com/elastic/logstash) repo:
   - https://github.com/elastic/logstash/tree/7.6/docker/data/logstash/env2yaml
4. [Open Distro](https://opendistro.github.io/for-elasticsearch/) plugins and patches:
   - https://github.com/opendistro-for-elasticsearch/opendistro-build
