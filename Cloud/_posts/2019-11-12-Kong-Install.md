---
layout: post
title: Kong 설치
description: >
  Kong , Konga 설치
---
# Kong 설치

## Docker로 설치하기



### Create a Docker network

~~~cmd
$ docker network create kong-net
~~~

컨테이너가 서로 검색하고 통신할 수 있도록 사용자 지정 네트워크를 만들어야 한다.
여기서는 'kong-net' 이라는 이름으로 사용한다.



### Start your database

~~~cmd
 $ docker run -d --name kong-database \
               --network=kong-net \
               -p 5432:5432 \
               -e "POSTGRES_USER=kong" \
               -e "POSTGRES_DB=kong" \
               postgres:9.6
~~~



### Prepare your database

~~~cmd
 $ docker run --rm \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
     kong:latest kong migrations bootstrap
~~~



### Start Kong

~~~cmd
$ docker run -d --name kong \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
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

마이그레이션이 실행되고 데이터베이스가 준비되면 데이터베이스 컨테이너에 연결할 콩 컨테이너를 시작한다



### Use Kong

~~~cmd
$ curl -i http://localhost:8001/
~~~



## Kong Admin Dashboard

[Github](https://github.com/pantsel/konga)

### Production Docker Image

~~~cmd
$ docker pull pantsel/konga
~~~

~~~cmd
$ docker run -p 1337:1337 \
             --network {{kong-network}} \ // optional
             --name konga \
             -e "NODE_ENV=production" \ // or "development" | defaults to 'development'
             -e "TOKEN_SECRET={{somerandomstring}}" \
             pantsel/konga
~~~



---

출처

https://docs.konghq.com/install/docker/

https://github.com/pantsel/konga