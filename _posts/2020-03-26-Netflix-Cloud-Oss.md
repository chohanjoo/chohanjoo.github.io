---
title: "[sns project] Netflix Oss를 사용하여 마이크로서비스 아키텍처 구성하기"
description: Netflix Oss (Zuul, eureka,ribbon)를 사용하여 마이크로서비스 아키텍처로 구성한다.
categories:
 - project
 - cloud
tags:
- project
- sns
- kubernetes
- cloud
---

# Spring Cloud Netflix

기존의 Monolithic Architecture 인 `Mwohae` SNS 프로젝트를 Microservice Architecture 로 변경해보자.

스프링 클라우드의 Netflix Oss - Zuul, Eureka, Ribbon 을 이용해서 서비스를 구축할 것이다.

> 넷플릭스는 모놀리식 애플리케이션 기반의 전톧적인 개발 모델에서 클라우드-네이티브 마이크로서비스 기반 개발 방식으로 전환한 선구자이다. 
>
> *"우리(넷플릭스)가 넷플릭스의 모든 것을 클라우드로 옮길 것이라고 했을 때 모두가 완전히 미쳤다고 했다. 그들은 우리가 실제로 해냈다고 믿지 않고 꾸며낸 것이라고 생각했다."*



## 서비스 디스커버리 [Eureka]

* 서비스 레지스트리 (Service Registry) - 모든 서비스 인스턴스와 인스턴스의 이름을 추적
* 레지스터 에이전트 (Register Agent) - 모든 서비스가 서로를 찾을 수 있도록 설정을 제공하는 에이전트
* 서비스 디스커버리 클라이언트 (Service Discovery Client) - 서비스 레지스트리에서 별칭으로 서비스를 조회

## API Gateway [Zuul]

### dependency

~~~xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
~~~

### application-dev.yml

로컬 환경에서 개발할 때 yaml 파일이다.  유레카에 등록된 serviceId로 리다이텍트해준다.

~~~yml
server:
  port: 8000

zuul:
  ignored-services: '*'
  prefix: /api    # 모든 요청의 접두사를 설정한다. 주울로 들어오는 모든 요청은 URL 내부에 리다이렉트할 때 주울이 제거하는 부분이 있어야 한다.
  routes:
    auth:
      path: /auth/**
      serviceId: auth     # bootstrap.properties에 설정한 값과 같아야 한다.
      strip-prefix: false
    user:
      path: /user/**
      serviceId: user
      strip-prefix: false
    post:
      path: /post/**
      serviceId: post
      strip-prefix: false
    generator:
      path: /generator/**
      serviceId: generator
      strip-prefix: false

endpoints:
  routes:
    sensitive: false
  trace:
    sensitive: false  # 주울의 /trace 엔드포인트에 접속할 때 인증이 필요 없도록 설정


eureka:
  client:
    service-url:
      default-zone: http://localhost:8761/eureka/
~~~

### application-prod.yml

Kubernetes로 배포할 때 사용되는 yaml 파일이다. 

유레카 서버는 `http://service-registry:8761/eureka/` 이다.

처음에는 serviceId로 개발하였으나 서비스를 찾지 못해 리다이렉션을 못하는 현상이 발생하였다. 그래서 지금은 url로 하드코딩하였다. 이는 후에 serviceId가 pod 이름에 매핑이 되는지, service 이름에 매핑이 되는지 공부해 변경해야 한다.

~~~yml
server:
  port: 8000

zuul:
  ignoredServices: '*'
  prefix: /api    # 모든 요청의 접두사를 설정한다. 주울로 들어오는 모든 요청은 URL 내부에 리다이렉트할 때 주울이 제거하는 부분이 있어야 한다.
  routes:
    auth:
      path: /auth/**
      url: http://auth:8080
      stripPrefix: false
    user:
      path: /user/**
      url: http://user:8081
      stripPrefix: false
    post:
      path: /post/**
      url: http://post:8082
      stripPrefix: false
    generator:
      path: /generator/**
      url: http://generator:8079
      stripPrefix: false

endpoints:
  routes:
    sensitive: false
  trace:
    sensitive: false  # 주울의 /trace 엔드포인트에 접속할 때 인증이 필요 없도록 설정


eureka:
  client:
    serviceUrl:
      defaultZone: http://service-registry:8761/eureka/
~~~



모든 요청은 접두사를 설정한다. 주울로 들어오는 모든 요청은 URL 내부에 리다이렉트할 때 주울이 제거하는 부분이 있어야 한다.

~~~yml
zuul:
  prefix: /api
~~~

`http://localhost:8000/api/auth` => `http://localhost:8000/auth` 로 리다이렉트된다.

이런 방법은 경로를 그룹화하고 다양한 정책을 적용하는 편리한 방법이다. 예를 들어 `/internal` 과 `/public` 접두사를 사용하는 두 개의 게이트웨이 설정을 사용할 수 있다.

API Gateway를 사용하면 API 소비자는 마이크로서비스에 대해 전혀 알지 못해도 상관없다. 모든 요청은 API Gateway인 `localhost:8000` 을 통과한다.



## 로드 밸런싱 [Ribbon]



## Troubleshooting

### CORS (Cross-Origin Resource Sharing) 문제

스프링 시큐리티 (spring security) 는 기본적으로 Same-Origin 정책을 사용하기 때문에 백엔드를 수정하지 않으면 몇 가지 문제가 발생한다. 이 문제를 해결하려면 다른 오리진에서 오늘 요청을 허용하기 위해 백엔드 서비스에서 CORS를 활성화해야 한다.

### 서비스들이 Eureka에 등록되지 않는 문제 

---

출처

스프링 부트를 활용한 마이크로 서비스 개발 - 모이세스 메이세로 지음

마스터링 스프링 클라우드 - 피요트르 민코프스키 지음