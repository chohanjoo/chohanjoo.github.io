# Kiali 아키텍처



## Description (기술)

What microservices are part of my Istio service mesh amd how are they connected?
내 Istio 서비스 메쉬의 일부인 마이크로서비스는 무엇이며 어떻게 연결되어 있는가?



Kiali works with Istio to visualise the service mesh topology, features like circuit breakers or request rates.
Kiali는 Istio와 협력하여 회로 차단기 또는 요청 속도와 같은 기능 메쉬 토폴로지를 시각화합니다.



Kiali also includes Jaeger Tracing to provide distributed tracing out of the box
Kiali는 Jaeger Tracing을 포함하여 분산 형 트레이싱을 제공합니다.



## Architecture

키알리는 컨테이너 애플리케이션 플랫폼에서 실행되는 백엔드 애플리케이션과 사용자 대면 프런트엔드 애플리케이션이라는 두 가지 요소로 구성된다. 
더욱이 키알리는 컨테이너 애플리케이션 플랫폼과 Istio에서 제공하는 외부 서비스와 구성요소에 의존한다.

![kiali_architecture](/Users/hanjoo/github_blog/assets/image/Kiali/kiali_architecture.png)



### Kiali back-end

백엔드는 컨테이너 애플리케이션 플랫폼에서 실행되는 애플리케이션이다. 고(Go)로 되어 있어. 

Istio와 통신하고, 데이터를 검색 및 처리하며, 이 데이터를 프런트 엔드에 공개하는 부품이다.

백엔드에는 스토리지가 필요 없다. 백엔드를 로컬로 실행할 때 파일 및 환경 변수(인증 시 사용되는 사용자 및 암호 포함)에 모든 구성이 설정된다. 도커 이미지를 사용하여 백엔드를 클러스터에 배포할 때 구성 맵과 암호를 사용하여 구성을 설정한다.



### Kiali front-end

프론트 엔드는 React로 구축되고 Typescript로 작성된 단일 페이지 웹 애플리케이션이다..

표준 배포에서 백엔드는 프런트엔드에 서비스를 제공한다. 그런 다음, 프런트 엔드에서는 데이터를 입수하여 사용자에게 제시하기 위해 키알리 백엔드를 조회한다.

Since, currently, there are no options for personalizations, it's mainly stateless. 세션 자격 증명과 같은 일부 데이터는 유지될 수 있지만 이 데이터는 브라우저에 저장되며 다른 브라우저나 다른 장치에서는 사용할 수 없다.



### Istio

Istio는 Kiali 요구사항이다. 서비스 메쉬를 제공하고 제어하는 구성 요소 입니다. Kiali와 Istio는 별도로 설치할 수 있지만, Kiali는 Istio에게 의존하고 있으며, 없으면 작동하지 않는다.

Kiali는 Prometheus와 Cluster API를 통해 노출되는 Istio 데이터와 구성을 검색해야 한다. 이것이 다이어그램에 점선으로 표시되어 간접 의존을 나타내는 종속성을 보여주는 이유다.



### Promethus

Promethus는 Istio 종속성입니다. Istio 원격 측정 기능이 활성화되면 메트릭 데이터가 Prometheus에 저장됩니다. Kiali는 Prometheus에 저장된 데이터를 사용하여 메쉬 토폴로지를 파악하고 메트릭을 표시하며 상태를 계산하고 가능한 문제(지표)를 표시합니다.

Kiali는 Prometheus와 직접 통신하고 Istio Telemetery에서 사용하는 데이터 스키마를 사용합니다. 그것은 Kiali에 대한 의존성이 크며, 대부분의 기능은 Kiali 없이는 작동하지 않습니다.

현재 Kiali는 Istio의 기본 메트릭 세트를 사용합니다. 이러한 기본 메트릭이 항상 존재하는지 확인하십시오. 맞춤 측정 항목은 Kiali에서 지원하지 않습니다. 사용자 지정 메트릭 또는 기본 메트릭의 변형이 필요한 경우 사용자 지정 이름을 사용하여 새 Istio 메트릭을 만듭니다.



### Cluster API

Kiali는 서비스 메쉬 구성을 가져오고 해결하기 위해 컨테이너 애플리케이션 플랫폼 (클러스터 API)의 API를 사용합니다.

Kiali가 작동하는 컨테이너 애플리케이션 플랫폼은 OKD와 Kubernetes입니다. Kiali는 이러한 플랫폼의 파생 모델에 대해서도 작업하고 있습니다. 클러스터 API를 배우려면 OKD REST API 참조 및 Kubernetes API 참조를 확인하십시오.

Kiali는 클러스터 API를 쿼리하여 예를 들어 네임 스페이스, 서비스, 배포, 포드 및 기타 엔터티에 대한 정의를 검색합니다. 또한 Kiali는 다른 클러스터 엔티티 간의 관계를 해결하기 위해 쿼리를 작성합니다.
가상 서비스, 대상 규칙, 경로 규칙, 게이트웨이, 할당량 등과 같은 Istio 구성을 검색하기 위해 클러스터 API도 쿼리됩니다.



### Jaeger

예거는 선택 사항입니다. 가능한 경우 Kiali는 사용자를 Jaeger의 추적 데이터로 안내 할 수 있습니다. 
이 기능이 필요한 경우, Kiali가 Jaeger 통합을 위해 올바르게 구성되었는지 확인하십시오.

추적 데이터는 Istio의 분산 추적이 사용 가능한 경우에만 사용할 수 있습니다.



### Grafana

Grafana는 선택 사항입니다. 가능한 경우 Kiali의 측정 항목 페이지에 Grafana의 동일한 측정 항목으로 연결되는 링크가 표시됩니다. 이 기능이 필요한 경우 Kiali가 Grafana 통합을 위해 올바르게 구성되었는지 확인하십시오.

Kiali는 기본 메트릭 기능을 제공합니다. 작업 부하, 응용 프로그램 및 서비스에 대한 기본 Istio 메트릭을 표시 할 수 있습니다. 제공된 메트릭에 몇 가지 그룹을 적용하고 다른 시간 범위에 대한 메트릭을 가져올 수 있습니다. However, Kiali doesn't allow to customize the views nor customize the Prometheus queries. 이러한 기능이 필요한 경우 Grafana를 설치해야합니다. Grafana를 설치하려면 Istio 문서를 참조하십시오.