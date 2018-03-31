---
layout: post
title:  granfana+prometheus monitoring mysql redis
date:   2018-03-31 18:38:58
categories: Linux
tags: Linux
---


# granfana+prometheus

all of those things are docker,if you do not install docker install docker first

## prometheus
~~~
mkdir /data/prometheus
chmod +x 65534.65534 /data/prometheus
~~~

### the content of prometheus.conf
~~~
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ['localhost:9090']


  - job_name: linux1
    static_configs:
      - targets: ['192.168.1.19:9100']
        labels:
          instance: mysql_master
  - job_name: mysql
    static_configs:
      - targets: ['192.168.1.19:9104']
        labels:
          instance: mysql_master
  - job_name: redis9
    static_configs:
      - targets: ['172.27.0.9:9121']
        labels:
          instance: redis
~~~


### start prometheus
~~~
docker run --name="prometheus" -d -v /data/prometheus:/prometheus -v /etc/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml -p 9090:9090 --restart unless-stopped prom/prometheus
~~~

## install node-exporter for prometheus
~~~
docker run --name="node-exporter" -d --net="host" --pid="host" --restart unless-stopped quay.io/prometheus/node-exporter
~~~
## install mysqld-exporter for prometheus
install mysqld-exporter on mysql Server
### modify mysqld ocnfig files
if mysql listen address is 127.0.0.1 you should add below
~~~
bind-address=0.0.0.0
~~~
### grant privileges to prom
~~~
CREATE USER 'prom'@'172.17.0.%' IDENTIFIED BY 'abcd12345';
GRANT PROCESS, REPLICATION CLIENT ON *.* TO 'prom'@'172.17.0.%';
GRANT SELECT ON performance_schema.* TO 'prom'@'172.17.0.%';
~~~
or it will have problem like this
~~~
 time="2018-03-31T01:03:34Z" level=error msg="Error pinging mysqld: dial tcp 192.168.1.19:3306: getsockopt: connection refused" source="mysqld_exporter.go:268"
~~~
### start mysqld-exporter
~~~
docker run -d -p 9104:9104 -e DATA_SOURCE_NAME="prom:abcd@(172.27.0.9:3306)/performance_schema" --restart unless-stopped prom/mysqld-exporter
~~~

## monitor redis
~~~
docker run -d --name redis_exporter -p 9121:9121 -e REDIS_ADDR=172.27.0.9:6379 --restart unless-stopped oliver006/redis_exporter
~~~

## install granfana
~~~
docker run -p 3000:3000 -d --restart unless-stopped  grafana/grafana:5.0.0
~~~
### next of the this is config grafana connect to prometheus
Inline-style:
![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")
