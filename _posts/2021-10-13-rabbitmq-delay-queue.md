---
title: "RabbitMQ Delayed Message Plugin 설치하기"
description: RabbitMQ에 Delayed Message Plugin 설치한다.
categories:
 - rabbitmq
tags:
- rabbitmq
- amqp
---

# rabbitmq delayed message exchange 플러그인 추가

기본적으로 rabbitmq에 플러그인을 설치하기 위해 다음 명령어를 사용한다.

```bash
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

<br/>

하지만 해당 플러그인 설치 시에는 다음과 같은 오류가 발생하였다.

```bash
Enabling plugins on node rabbit@92f2558ab8fa:
rabbitmq_delayed_message_exchange
Error:
{:plugins_not_found, [:rabbitmq_delayed_message_exchange]}
```

<br/>

`rabbitmq-plugins` 로 `rabbitmq_delayed_message_exchange` 를 찾지 못하는 것이였다.

<br/>

해당 플러그인을 설치하기 위해 다음과 같이 해결하였다.

1. [rabbitmq_delayed_message_exchange](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases) 에서 원하는 버전을 다운받는다.

2. 아래 도커파일을 작성한다.

```dockerfile
FROM rabbitmq:3.8-management
COPY ./rabbitmq_delayed_message_exchange-3.9.0.ez /opt/rabbitmq/plugins/
RUN rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

3. 도커 컨테이너를 실행한다.

```cmd
docker build -t chohanjoo/rabbitmq:3.8 .


docker run -d --name rabbitmq -p 5672:5672 -p 25672:15672 --restart=unless-stopped chohanjoo/rabbitmq:3.8
```
