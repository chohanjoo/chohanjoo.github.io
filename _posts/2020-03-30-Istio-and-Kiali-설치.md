---
title: "[sns project] Istio 및 Kiali 설치하기"
description: 마이크로서비스간의 네트워크 플로우를 보기 위해, 관리하기 위해 Kiali 라는 오픈소스를 사용할 것이다.
categories:
 - project
 - cloud
tags:
- project
- sns
- kubernetes
- cloud
---

# Istio & Kiali 설치

마이크로서비스간의 네트워크 플로우를 보기 위해, 관리하기 위해 Kiali 라는 오픈소스를 사용할 것이다.

설치방법에는 여러 가지가 있지만, Istio를 통해 설치할 것이다. Istio 공식 홈페이지를 참고하였다.

본 설치는 MacOS에서 진행되었다.



## Download Istio

### Istio 소스를 다운 받는다.

~~~shell
$ curl -L https://istio.io/downloadIstio | sh -
~~~

### 다운받은 디렉토리로 이동한다.

~~~shell
$ cd istio-1.5.1
~~~

디렉토리에는 3가지 내용을 포함하고 있다.

1. Kubernetes를 위한 YAML 파일이 있다. [ install/kubernetes ]
2. 샘플 애플리케이션이 있다. [ samples/ ]
3. `istioctl` client binary가 있다. [ bin/ ]

### 환경변수를 설정한다. (Linux , macOS)

~~~shell
$ export PATH=$PWD/bin:$PATH
~~~



## Install Istio

### demo configuration profile 로 설치한다.

설치하는 기능에 따라 다양한 configuration profile이 있다. 하지만 필자는 `prometheus` ,`grafana` 등 다양한 프로젝트를 사용해 볼 예정이므로 가장 기능이 많은 `demo` 버전으로 설치했다. 다른 profile은 [여기](https://istio.io/docs/setup/additional-setup/config-profiles/) 를 참고한다.

~~~shell
$ istioctl manifest apply --set profile=demo

Detected that your cluster does not support third party JWT authentication. Falling back to less secure first party JWT
- Applying manifest for component Base...
✔ Finished applying manifest for component Base.
- Applying manifest for component Pilot...
✔ Finished applying manifest for component Pilot.
Waiting for resources to become ready...
- Applying manifest for component EgressGateways...
- Applying manifest for component IngressGateways...
- Applying manifest for component AddonComponents...
✔ Finished applying manifest for component EgressGateways.
✔ Finished applying manifest for component IngressGateways.
✔ Finished applying manifest for component AddonComponents.

✔ Installation complete
~~~

** 설치 중간에 에러가 발생해서 고생했다. 다양한 시도 끝에 리부팅으로 해결되었다.

### 사이드카 프록시 주입

네임스페이스 레이블을 추가하여 나중에 애플리케이션을 배포할 때 Istio가 자동으로 특수 사이드카 프록시를 주입하도록 한다.

~~~shell
$ kubectl label namespace default istio-injection=enabled
~~~

모든 설치가 끝난 후 `kubectl get pod -A` 명령어를 통해 모든 파드를 확인해 보면 `Running` 임을 확인할 수 있다. 모든 파드가 `Running` 상태가 될 때까지 꽤 오랜시간이 걸렸다.

![after_kiali](/assets/images/sns/after_kiali.png)

## View the dashboard

네트워크 플로우를 보기 위해 `kiali` 대시보드를 실행한다.

~~~shell
$ istioctl dashboard kiali
~~~



---

출처

[Istio](https://istio.io/docs/setup/getting-started/)