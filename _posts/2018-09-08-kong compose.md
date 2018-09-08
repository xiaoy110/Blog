---
layout: post
title:  Managing microservices and APIs with Kong and Konga
date:   2018-09-08 15:20:58
categories: Docker
tags: Docker
---


# Managing microservices and APIs with Kong and Konga
This article will be focusing on how to manage microservices using Kong and Konga. We will go through the deployment process of both applications and provide a basic example of accessing an API through the gateway as well as securing it’s resources using Authentication plugins and Access Control Lists.

# What is Kong?
https://konghq.com/  
Kong is a widely adopted, open source API Gateway, written in Lua. It runs on top of Nginx, leveraging the OpenResty framework and provides a simple RESTful API that can be used to provision your infrastructure in a dynamic way.

# What is Konga?
https://github.com/pantsel/konga  
Konga is a fully featured open source, multi-user GUI, that makes the hard task of managing multiple Kong installations a breeze.

It can be integrated with some of the most popular databases out of the box and provides the visual tools you need to better understand and maintain your architecture.

# Install Kong

For this tutorial, we’re going to use docker containers. Other ways to install Kong can be found here: https://konghq.com/install

1. To begin with, we need to create a network so that the containers can discover and communicate with each other.
~~~
docker network create kong-net
~~~
2. We will be using PostgreSQL as Kong’s database (it can be either that or Cassandra).
~~~
docker run -d --name kong-database \
                --network=kong-net \
                -p 5432:5432 \
                -e “POSTGRES_USER=kong” \
                -e “POSTGRES_DB=kong” \
                postgres:9.6
~~~
3. Let us now prepare our database by starting an ephemeral Kong container which will run the appropriate migrations and die!
~~~
docker run --rm \     
             --network=kong-net \     
             -e "KONG_DATABASE=postgres" \     
             -e "KONG_PG_HOST=kong-database" \     
             kong:latest kong migrations up
~~~
4. Everything is now set and ready to start Kong!
~~~
docker run -d --name kong \     
             --network=kong-net \     
             -e "KONG_DATABASE=postgres" \     
             -e "KONG_PG_HOST=kong-database" \         
             -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \     
             -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \     
             -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \     
             -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \     
             -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \     
             -p 8000:8000 \     
             -p 8443:8443 \     
             -p 8001:8001 \     
             -p 8444:8444 \     
             kong:latest
~~~
5. Test it out!

Kong’s admin API is exposed on port 8001 and the gateway on port 8000
~~~
curl -i http://localhost:8001/
curl -i http://localhost:8000/
~~~
# Install Konga
Konga ports with it’s own file system storage and although it’s not recommended for production, it can be safely used as long as kongadata folder persists in a volume.

If you decide to go down that road, running Konga is as simple as this:
~~~
docker run -d -p 1337:1337 \
             --network=kong-net \
             --name konga \
             -v /var/data/kongadata:/app/kongadata \
             -e "NODE_ENV=production" \
             pantsel/konga
~~~
For this tutorial, since we already have a running PostgreSQL instance, we might as well make our lifes a bit more complicated and use it.

Like before, we will need to prepare Konga’s database by starting an ephemeral container.
~~~
docker run --rm \
    --network=kong-net \
    pantsel/konga -c prepare -a postgres -u postgresql://kong@kong-database:5432/konga_db
~~~
When the migrations run, we can start the app.
~~~
docker run -p 1337:1337 \
             --network=kong-net \
             -e "DB_ADAPTER=postgres" \
             -e "DB_HOST=kong-database" \
             -e "DB_USER=kong" \
             -e "DB_DATABASE=konga_db" \
             -e "KONGA_HOOK_TIMEOUT=120000" \
             -e "NODE_ENV=production" \
             --name konga \
             pantsel/konga
~~~
Here’s a docker-compose example for the lazy: https://gist.github.com/pantsel/73d949774bd8e917bfd3d9745d71febfi  
and this is mine
~~~
version: '2'

services:

  kong-database:
    networks:
      - "kong-net"
    image: "postgres:9.6"
    ports:
      - "5432:5432"
    volumes:
      - ./postgres-kong:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: "kong"
      POSTGRES_DB: "kong"
  kong-migrations:
    networks:
      - 'kong-net'
    image: "kong:latest"
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
    command: kong migrations up
  kong:
    networks:
      - 'kong-net'
    image: "kong:latest"
    environment:
      KONG_DATABASE: "postgres"
      KONG_PG_HOST: "kong-database"
      KONG_ADMIN_LISTEN: "0.0.0.0:8001"
    ports:
      - "8000:8000"
      - "8001:8001"
      - "8443:8433"
      - "8444:8444"
  konga:
    networks:
      - 'kong-net'
    image: 'pantsel/konga:latest'
    volumes:
      - ./kongadata:/app/kongadata:rw
    ports:
      - "1337:1337"
    environment:
      NODE_ENV: 'production'
~~~
After a while, Konga will be available at:
~~~
http://<your-servers-public-ip-or-host>:1337
~~~

recommand links
https://medium.com/@tselentispanagis/managing-microservices-and-apis-with-kong-and-konga-7d14568bb59d
