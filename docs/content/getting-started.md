---
title: "Getting Started"
date: 2018-05-02T00:00:00+00:00
anchor: "getting-started"
weight: 10
---

## Installation

We won't cover further details how to properly setup [Prometheus](https://prometheus.io) itself, we will only cover some basic setup based on [docker-compose](https://docs.docker.com/compose/). But if you want to run this exporter without [docker-compose](https://docs.docker.com/compose/) you should be able to adopt that to your needs.

First of all we need to prepare a configuration for [Prometheus](https://prometheus.io) that includes the exporter as a target based on a static host mapping which is just the [docker-compose](https://docs.docker.com/compose/) container name, e.g. `hcloud_exporter`.

{{< highlight yaml >}}
global:
  scrape_interval: 1m
  scrape_timeout: 10s
  evaluation_interval: 1m

scrape_configs:
- job_name: github
  static_configs:
  - targets:
    - hcloud_exporter:9501
{{< / highlight >}}

After preparing the configuration we need to create the `docker-compose.yml` within the same folder, this `docker-compose.yml` starts a simple [Prometheus](https://prometheus.io) instance together with the exporter. Don't forget to update the exporter envrionment variables with the required credentials.

{{< highlight yaml >}}
version: '2'

volumes:
  prometheus:

services:
  prometheus:
    image: prom/prometheus:latest
    restart: always
    ports:
      - 9090:9090
    volumes:
      - prometheus:/prometheus
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  hcloud_exporter:
    image: promhippie/hcloud-exporter:latest
    restart: always
    environment:
      - HCLOUD_EXPORTER_TOKEN=bldyecdtysdahs76ygtbw51w3oeo6a4cvjwoitmb
      - HCLOUD_EXPORTER_LOG_PRETTY=true
{{< / highlight >}}

Since our `latest` Docker tag always refers to the `master` branch of the Git repository you should always use some fixed version. You can see all available tags at our [DockerHub repository](https://hub.docker.com/r/promhippie/hcloud-exporter/tags/), there you will see that we also provide a manifest, you can easily start the exporter on various architectures without any change to the image name. You should apply a change like this to the `docker-compose.yml`:

{{< highlight diff >}}
  hcloud_exporter:
-   image: promhippie/hcloud_exporter:latest
+   image: promhippie/hcloud_exporter:1.0.0
    restart: always
    environment:
      - HCLOUD_EXPORTER_TOKEN=bldyecdtysdahs76ygtbw51w3oeo6a4cvjwoitmb
      - HCLOUD_EXPORTER_LOG_PRETTY=true
{{< / highlight >}}

If you want to access the exporter directly you should bind it to a local port, otherwise only [Prometheus](https://prometheus.io) will have access to the exporter. For debugging purpose or just to discover all available metrics directly you can apply this change to your `docker-compose.yml`, after that you can access it directly at [http://localhost:9501/metrics](http://localhost:9501/metrics):

{{< highlight diff >}}
  hcloud_exporter:
    image: promhippie/hcloud_exporter:latest
    restart: always
+   ports:
+     - 127.0.0.1:9501:9501
    environment:
      - HCLOUD_EXPORTER_TOKEN=bldyecdtysdahs76ygtbw51w3oeo6a4cvjwoitmb
      - HCLOUD_EXPORTER_LOG_PRETTY=true
{{< / highlight >}}

If you want to secure the access to the exporter or also the HTTP service discovery endpoint you can provide a web config. You just need to provide a path to the config file in order to enable the support for it, for details about the config format look at the [documentation](#web-configuration) section:

{{< highlight diff >}}
  hcloud_exporter:
    image: promhippie/hcloud-exporter:latest
    restart: always
    environment:
+     - HCLOUD_EXPORTER_WEB_CONFIG=path/to/web-config.json
      - HCLOUD_EXPORTER_TOKEN=bldyecdtysdahs76ygtbw51w3oeo6a4cvjwoitmb
      - HCLOUD_EXPORTER_LOG_PRETTY=true
{{< / highlight >}}

Finally the exporter should be configured fine, let's start this stack with [docker-compose](https://docs.docker.com/compose/), you just need to execute `docker-compose up` within the directory where you have stored the `prometheus.yml` and `docker-compose.yml`.

That's all, the exporter should be up and running. Have fun with it and hopefully you will gather interesting metrics and never run into issues. You can access the exporter at [http://localhost:9501/metrics](http://localhost:9501/metrics) and [Prometheus](https://prometheus.io) at [http://localhost:9090](http://localhost:9090).

## Configuration

{{< partial "envvars.md" >}}

### Web Configuration

If you want to secure the service by TLS or by some basic authentication you can provide a `YAML` configuration file whch follows the [Prometheus](https://prometheus.io) toolkit format. You can see a full configration example within the [toolkit documentation](https://github.com/prometheus/exporter-toolkit/blob/master/docs/web-configuration.md).

## Metrics

You can a rough list of available metrics below, additionally to these metrics you will always get the standard metrics exported by the Golang client of [Prometheus](https://prometheus.io). If you want to know more about these standard metrics take a look at the [process collector](https://github.com/prometheus/client_golang/blob/master/prometheus/process_collector.go) and the [Go collector](https://github.com/prometheus/client_golang/blob/master/prometheus/go_collector.go).

{{< partial "metrics.md" >}}
