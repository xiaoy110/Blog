---
layout: post
title:  prometheus+consul auto discover
date:   2018-04-02 18:38:58
categories: Linux
tags: Linux
---

# prometheus+consul
it's very easy to deploy it
## prometheus.yml
~~~
global:
  scrape_interval: 5s
  scrape_timeout: 5s
  evaluation_interval: 15s
scrape_configs:
  - job_name: consul_sd
    relabel_configs:
    - source_labels:  ["__meta_consul_dc"]
      regex: "(.*)"
      replacement: $1
      action: replace
      target_label: "dc"
    metrics_path: /metrics
    scheme: http
    consul_sd_configs:
      - server: consul:8500
        scheme: http
        services:
~~~

if you want to filter data
## prometheus.yml2
~~~
global:
  scrape_interval: 5s
  scrape_timeout: 5s
  evaluation_interval: 15s
scrape_configs:
  - job_name: consul_sd
    relabel_configs:
    - source_labels: ["__meta_consul_tags"]
      regex: ".*,development,.*"
      #regex: ".*,dev,.*"
      action: keep
    - source_labels:  ["__meta_consul_dc"]
      regex: "(.*)"
      replacement: $1
      action: replace
      target_label: "dc"
    metrics_path: /metrics
    scheme: http
    consul_sd_configs:
      - server: consul:8500
        scheme: http
        services:
~~~
## docker-compose.yml
~~~
version: '2'

services:
  consul:
    image: consul
    ports:
      - 8400:8400
      - 8501:8500
      - 8600:53/udp
    command: agent -server -client=0.0.0.0 -dev -node=node0 -bootstrap-expect=1 -data-dir=/tmp/consul
    labels:
      SERVICE_IGNORE: 'true'
  registrator:
    image: gliderlabs/registrator
    depends_on:
      - consul
    volumes:
      - /var/run:/tmp:rw
    command: consul://consul:8500
  prometheus:
    image: quay.io/prometheus/prometheus
    volumes:
      - /data/prometheus:/prometheus
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - 9090:9090
  node_exporter:
    image: quay.io/prometheus/node-exporter
    pid: "host"
    cap_add:
      - SYS_TIME
    ports:
      - 9100:9100
    labels:
      SERVICE_TAGS: "development"
  cadvisor:
    image: google/cadvisor:latest
    ports:
      - 8080:8080
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /var/lib/docker/:/var/lib/docker:ro
    labels:
      SERVICE_TAGS: "production,scraped"
~~~
Note it will be error if without cap_add

visit ip:9090
- if you're ues prometheus.yaml it would be show all servers
- if you're use protheus.yaml2 it will be show one


## how to register servers
~~~
curl -X PUT -d '{"id": "MySql","name": "MySql","address": "localhost","port": 9104,"tags": ["dev"],"checks": [{"http": "http://localhost:9104/","interval": "5s"}]}'     http://localhost:8501/v1/agent/service/register
~~~
## how to delete servers
~~~
curl  --request PUT  http://localhost:8500/v1/agent/service/deregister/MySql
~~~
