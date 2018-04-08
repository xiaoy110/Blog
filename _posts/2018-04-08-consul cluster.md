---
layout: post
title:  prometheus+consul auto discover
date:   2018-04-08 8:38:58
categories: Linux
tags: Linux
---

# consul cluster
```
mkdir /home/k/consulcluster
```
then makes following files
## consul configure
```

echo 'consul agent -config-file=/wk/consul/client1.json' > runClient1.sh

echo 'consul agent -config-file=/wk/consul/server1.json -ui-dir=/opt/consul/web' > runServer1.sh

echo 'consul agent -config-file=/wk/consul/server2.json' > runServer2.sh

echo 'consul agent -config-file=/wk/consul/server3.json' > runServer3.sh


cat >>client1.json<< EOF
{
	"data_dir": "/opt/consulclient",
	"log_level": "INFO",
	"node_name": "client1",
	"server": false,
        "ui": true,
        "http_config": {
           "response_headers": {
              "Access-Control-Allow-Origin": "*"
            }
        },
	"addresses": {
		"http": "0.0.0.0"
	},
	"start_join": ["172.17.0.2", "172.17.0.3", "172.17.0.4"]
}
EOF

cat >>server1 <<EOF
{
	"data_dir": "/opt/consul",
	"log_level": "INFO",
	"node_name": "server1",
	"server": true,
	"bind_addr": "172.17.0.2",
	"bootstrap": true,
	"retry_join": ["172.17.0.2", "172.17.0.3", "172.17.0.4"]
}
EOF

cat >>server2 <<EOF
{
	"data_dir": "/opt/consul",
	"log_level": "INFO",
	"node_name": "server2",
	"server": true,
	"bind_addr": "172.17.0.3",
	"retry_join": ["172.17.0.2", "172.17.0.3", "172.17.0.4"]
}
EOF

cat >>server3 <<EOF
{
	"data_dir": "/opt/consul",
	"log_level": "INFO",
	"node_name": "server3",
	"server": true,
	"bind_addr": "172.17.0.4",
	"retry_join": ["172.17.0.2", "172.17.0.3", "172.17.0.4"]
}
EOF
```

## runing command
```
docker run --name=server1 -itd -v /home/k/consulcluster:/wk consul sh /wk/consul/runServer1.sh
docker run --name=server2 -itd -v /home/k/consulcluster:/wk consul sh /wk/consul/runServer2.sh
docker run --name=server3 -itd -v /home/k/consulcluster:/wk consul sh /wk/consul/runServer3.sh
docker run --name=client1 -itd -p 8500:8500 -v /share:/wk consul sh /wk/consul/runClient1.sh
```
