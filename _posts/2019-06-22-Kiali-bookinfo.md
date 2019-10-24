---
layout: post
title:  "Kiali boookinfo Applecation"
date:   2019-06-22 15:00:00
tags:    [Cloud, Istio, Kiali]
comments: true
---
# Kiali boookinfo Applecation



본 포스트에서는 다양한 Istio 특징을 입증하는 데 사용되는 4개의 개별 마이크로 서비스로 구성된 샘플 애플리케이션을 배포한다. 이 애플리케이션은 온라인 서점의 단일 카탈로그 항목과 유사하게 책에 대한 정보를 보여준다. 페이지에 표시되는 내용은 책, 책 세부사항(ISBN, 페이지 수 등) 및 몇 가지 서평이다. 샘플 애플리케이션은 쿠버네티스 환경에서 동작함을 가정한다.



Bookinfo 애플리케이션은 네 가지 개별 마이크로 서비스로 구분된다.

- product page : details and reviews micro services 호출하여 페이지를 채운다.
- details : 책 정보를 포함하고 있다.
- reviews : 책 리뷰를 포함하고 있다. 그것은 또한 그 등급들을 ratings microservice 라고 부른다.
- ratings : 서평과 함께 제공되는 서평 정보가 포함되어 있다.

리뷰 마이크로서비스에는 세 가지 버전이 있다.

- v1 버전은 등급 서비스를 호출하지 않는다.
- v2 버전은 등급 서비스를 호출하며, 각 등급은 1~5개의 흑인 스타로 표시한다.
- v3 버전은 등급 서비스를 호출하고, 각 등급을 1~5개의 빨간색 별로 표시한다.



애플리케이션의 end to end 아키텍처는 아래와 같다.

![noistio](/Users/hanjoo/github_blog/assets/image/istio/booinfo/noistio.svg)

이 애플리케이션은 polyglot이다. 즉, 마이크로서비스는 서로 다른 언어로 작성된다.
이러한 서비스는 Istio에 의존하지 않지만, 특히 리뷰 서비스에 대한 다수의 서비스, 언어 및 버전 때문에 흥미로운 서비스 메시 예를 들 수 있다.



Istio로 샘플을 실행하려면 애플리케이션 자체를 변경할 필요가 없다. 대신에, 우리는 단지 각 서비스 편을 따라 특수 사이드카를 투입하여 Istio가 활성화된 환경에서 서비스를 구성하고 실행하기만 하면 된다. 필요한 명령과 구성은 런타임 환경에 따라 다르지만 모든 경우 다음과 같이 배치된다.

![withistio](/Users/hanjoo/github_blog/assets/image/istio/booinfo/withistio.svg)



모든 마이크로 서비스는 서비스의 수신 및 발신을 가로 채고, Istio control plane, routing, telemetry collection 및 애플리케이션 전체에 대한 정책 집행을 통해 외부 제어에 필요한 후크를 제공하는 엔보이 사이드카와 함께 포장된다.

— minikube를 사용하고 있다면, 최소 4GB RAM이 있어야 한다.



### 1. Istio 디렉토리로 이동



###2. 기본 Istio 설치는 자동 사이드카 인젝션을 사용한다. istio-inject = enabled 로 응용 프로그램을 호스팅 할 네임 스페이스에 레이블을 지정합니다.

~~~
$ kubectl label namespace default istio-injection=enabled
~~~

![Screenshot from 2019-06-22 14-59-28](/Users/hanjoo/github_blog/assets/image/istio/booinfo/Screenshot from 2019-06-22 14-59-28.png)



###3. kubectl 명령을 사용하여 애플리케이션 배포

~~~
$ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
~~~

![Screenshot from 2019-06-22 14-59-40](/Users/hanjoo/github_blog/assets/image/istio/booinfo/Screenshot from 2019-06-22 14-59-40.png)이 명령은 bookinfo 애플리케이션 아키텍처 다이어그램에 표시된 네 가지 서비스를 모두 시작합니다. 리뷰 서비스 v1, v2 및 v3의 3 가지 버전이 모두 시작됩니다.

> 실제 배포에서는 모든 버전을 동시에 배포하지 않고 시간이 지남에 따라 새로운 버전의 마이크로 서비스가 배포됩니다.



### 4. 모든 services 및 Pods 가 올바르게 정의되어 실행 중인지 확인

~~~
$ kubectl get services
~~~

![Screenshot from 2019-06-22 15-00-21](/Users/hanjoo/github_blog/assets/image/istio/booinfo/Screenshot from 2019-06-22 15-00-21.png)

~~~
$ kubectl get pods
~~~

![Screenshot from 2019-06-22 15-00-32](/Users/hanjoo/github_blog/assets/image/istio/booinfo/Screenshot from 2019-06-22 15-00-32.png)



### 5. curl 명령으로 요청을 보내 Bookinfo 프로그램이 실행중인지 확인

~~~
$ kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"
~~~

![Screenshot from 2019-06-22 15-00-48](/Users/hanjoo/github_blog/assets/image/istio/booinfo/Screenshot from 2019-06-22 15-00-48.png)



## Determining the ingress IP and port

이제 Bookinfo 서비스가 가동되고 실행되고 있으므로, 당신은 당신의 쿠버네티스 클러스터 외부(예: 브라우저)에서 그 애플리케이션에 접근할 수 있도록 해야 한다. Istio gateway는 이러한 목적으로 사용된다.

###6. 응용 프로그램의 진입 게이트웨이를 정의

~~~
$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
~~~

![Screenshot from 2019-06-22 15-01-07](/Users/hanjoo/github_blog/assets/image/istio/booinfo/Screenshot from 2019-06-22 15-01-07.png)



###7. 게이트웨이가 만들어졌는지 확인

~~~
$ kubectl get gateway
~~~

![Screenshot from 2019-06-22 15-01-27](/Users/hanjoo/github_blog/assets/image/istio/booinfo/Screenshot from 2019-06-22 15-01-27.png)



###8. 외부 로드 밸런서를 지원하는 환경에서 쿠버네티스 클러스터가 실행 중인지 확인

~~~
$ kubectl get svc istio-ingressgateway -n istio-system
~~~

![Screenshot from 2019-06-22 15-01-44](/Users/hanjoo/github_blog/assets/image/istio/booinfo/Screenshot from 2019-06-22 15-01-44.png)

EXTERNAL-IP 값을 설정한 경우 사용자 환경에는 수신 게이트웨이에 사용할 수 있는 외부 로드 밸런서가 있다. EXTERNAL-IP 값이 <none>(또는 영구 <pending>인 경우, 사용자 환경은 수신 게이트웨이에 외부 로드 밸런서를 제공하지 않는다. 이 경우 서비스의 노드 포트를 사용하여 게이트웨이에 액세스할 수 있다.

현재 컴퓨터에서는 외부 로드 밸런서를 제공하지 않으므로 서비스의 노드 포트를 사용하여 게이트웨이에 액세스한다.
외부 로드 밸런서를 제공하는 경우 <https://istio.io/docs/tasks/traffic-management/ingress/ingress-control/#determining-the-ingress-ip-and-ports> 다음 URL 참고한다.



### 9. ingress ports 설정

~~~
$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
~~~

![Screenshot from 2019-06-22 15-02-09](/Users/hanjoo/github_blog/assets/image/istio/booinfo/Screenshot from 2019-06-22 15-02-09.png)



### 10. ingress ip 설정

~~~
$ export INGRESS_HOST=127.0.0.1
~~~



###11. GATEWAY_URL 설정

~~~
$ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
~~~



## Confirm the app is accessible from outside the cluster



###12. 클러스터 외부에서 Bookminfo 애플리케이션에 액세스할 수 있는지 확인

~~~
$ curl -s http://${GATEWAY_URL}/productpage | grep -o "<title>.*</title>"
~~~

![Screenshot from 2019-06-22 15-02-32](/Users/hanjoo/github_blog/assets/image/istio/booinfo/Screenshot from 2019-06-22 15-02-32.png)

당신은 또한 Booinfo 웹 페이지를 보기 위해 당신의 브라우저를 http://$GATEWAY_URL/productpage으로 가리킬 수 있다. 페이지를 여러 번 새로 고치면, 아직 버전 라우팅을 제어하기 위해 Istio를 사용하지 않았기 때문에 라운드 로빈 스타일(빨간색 별, 검은 별, 별 없음)으로 표시된 다양한 버전의 리뷰를 제품 페이지에 볼 수 있을 것이다.



## Apply default destination rules

Istio를 사용하여 Booinfo 버전 라우팅을 제어하기 전에 대상 규칙에서 사용 가능한 버전, 즉 서브셋을 정의해야 한다.



###13. Bookinfo 서비스에 대한 기본 대상 규칙을 생성

~~~
$ kubectl apply -f samples/bookinfo/networking/destination-rule-all-mtls.yaml
~~~

![Screenshot from 2019-06-22 15-02-52](/Users/hanjoo/github_blog/assets/image/istio/booinfo/Screenshot from 2019-06-22 15-02-52.png)몇 초만 기다리면...

![Screenshot from 2019-06-22 15-00-10](/Users/hanjoo/github_blog/assets/image/istio/booinfo/Screenshot from 2019-06-22 15-00-10.png)

### 14. destination rules 확인

~~~
$ kubectl get destinationrules -o yaml
~~~



# Visualizing Your Mesh

이 작업은 Istio Mesh의 여러 가지 측면을 시각화하는 방법을 보여준다.

이 작업의 일부로 Kiali 추가 기능을 설치하고 웹 기반 그래픽 사용자 인터페이스를 사용하여 메시 및 Istio 구성 개체의 서비스 그래프를 볼 수 있다.

Bookinfo 샘플 애플리케이션을 예제로 사용한다.

##Generating a service graph

###15. 클러스터에서 서비스가 실행 중인지 확인

~~~
$ kubectl -n istio-system get svc kiali
~~~

![Screenshot from 2019-06-22 15-03-28](/Users/hanjoo/github_blog/assets/image/istio/booinfo/Screenshot from 2019-06-22 15-03-28.png)

###16. send traffic to the mesh

~~~
$ curl http://$GATEWAY_URL/productpage
~~~

~~~
http://$GATEWAY_URL/productpage
~~~

![1561183855610](/Users/hanjoo/github_blog/assets/image/istio/booinfo/1561183855610.png)

###17. open the Kiali UI

~~~
$ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001
~~~

~~~
http://localhost:20001/kiali/console 
~~~

![Screenshot from 2019-06-22 15-03-49](/Users/hanjoo/github_blog/assets/image/istio/booinfo/Screenshot from 2019-06-22 15-03-49.png)



http://localhost:20001/kiali/console](http://localhost:20001/kiali/console)





---

출처

https://istio.io/docs/examples/bookinfo/#determining-the-ingress-ip-and-port