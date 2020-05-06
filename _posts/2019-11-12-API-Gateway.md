---
layout: post
title: API Gateway
description: >
  API Gateway란?
noindex: true
---
# API Gateway

API Gateway는 접근 제어 및 트래픽을 모니터링하고 보안 기능을 제공하여 전체적인 마이크로 서비스를 보호하는 그 유입점을 담당한다.

![API_Gateway](/Users/hanjoo/github_blog/assets/image/cloud/API_Gateway.png)



![order_system_api_gateway](/Users/hanjoo/github_blog/assets/image/cloud/order_system_api_gateway.png)

모든 사용자의 API 요청은 제일 먼저 API Gateway에 도착하게 된다. API Gateway는 받은 요청을 기반으로 필요한 마이크로서비스에 개별적인 요청을 다시 보내게 된다. 이렇게 각각의 마이크로서비스로부터 받은 응답들을 API Gateway는 다시 취합하여 클라이언트에게 전달하는 역할을 수행한다. 이 과정에서 API Gateway는 사용자의 HTTP 요청을 마이크로서비스가 받을 수 있는 다른 형태의 프로토콜로 전환하는 역할을 수행하기도 한다.

API Gateway는 이처럼 클라이언트의 요청을 일괄적으로 처리할 뿐만 아니라. 전체 시스템의 부하를 분산시키는 로드 밸런서의 역할, 동일한 요청에 대한 불필요한 반복작업을 줄일 수 있는 캐싱, 시스템상을 오고가는 요청과 응답에 대한 모니터링 역할도 수행할 수 있다.

이렇게 API Gateway를 이용하여 서비스 요청에 대한 처리를 하게 되면 특정 서비스의 변경사항이 생기거나 서비스가 통합/분리 되더라도 클라이언트는 그 사실을 인지할 필요가 없으며 API Gateway 내부의 변경사항만으로 처리가 가능하게 된다.

이렇게 시스템 내부의 아키텍처를 숨길 수 있는 특성이 API Gateway가 갖는 가장 큰 장점이라고 할 수 있다.

API Gateway가 갖는 단점은 병목 지점이 될 수 있다는 것이다. API Gateway를 비동기적이고 non-blocking I/O 처리가 가능하도록 구현하는 것이 적은 비용으로 최대의 성능을 발휘할 수 있는 관건이 된다.

---

출처

https://waspro.tistory.com/433?category=857035