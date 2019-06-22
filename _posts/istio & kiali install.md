# Istio & Kiali 설치



> 쿠버네티스 설치 실패시 다음의 초기화 과정을 수행한다.
>
> ## sudo kubeadm reset
>
> ####$ vim script.sh
>
> \#!/bin/sh
>
> sudo systemctl stop kubelet
>
> sudo systemctl stop docker
>
> sudo iptables --flush
>
> sudo iptables -tnat --flush
>
> sudo rm -rf /var/lib/cni/
>
> sudo rm -rf /var/lib/kubelet/*
>
> sudo rm -rf /etc/cni/
>
> sudo ifconfig cni0 down
>
> sudo ifconfig flannel.1 down
>
> sudo ifconfig docker0 down
>
> 
>
> 
>
> sudo ip link delete cni0
>
> sudo ip link delete flannel.1
>
> 
>
> sudo systemctl daemon-reload
>
> sudo systemctl start kubelet
>
> sudo systemctl start docker
>
> 
>
> \# cannot remove /var/lib/kubelet/pods 
>
> \# umount $(df -HT | grep '/var/lib/kubelet/pods' | awk '{print $7}')
>
> ####$ sudo chmod +x script.sh
>
> #####$ sudo su
>
> #####$ ./script.sh



## Downloading the release

Istio는 자체 istio-system 네임 스페이스에 설치되며 다른 모든 네임 스페이스의 서비스를 관리 할 수 있다.

### 1. 최신 릴리스 다운로드

~~~
$ curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.2.0 sh -
~~~



###2. istio 디랙토리로 이동

~~~
$ cd istio-1.2.0
~~~

- install/kubernetes 디렉토리 안에 쿠버네티스에서의 설치 yaml파일이 있다. 
- samples/ 디렉토리 안에 예제 애플리케이션이 있다.
- bin/ 디렉토리에 istioctl client binary가 있다. 
  istioctl은 사이드카 프록시로 Envoy를 수동으로 주입 할 때 사용됩니다.



###3. istioctl client를 환경 변수에 추가

~~~
export PATH=$PWD/bin:$PATH
~~~



##Installing the Chart

차트는 리소스 구성 매개 변수에 지정된 대로 최소 리소스를 사용하는 포드를 배포한다.

###4. Tiller 용 서비스 계정 설치

~~~
$ kubectl apply -f install/kubernetes/helm/helm-service-account.yaml
~~~

![tiller account install](/Users/hanjoo/github_blog/assets/image/istio/tiller account install.png)



###5. 서비스 계정으로 클러스터에 Tiller 설치

~~~
$ helm init --service-account tiller
~~~

![tiller install](/Users/hanjoo/github_blog/assets/image/istio/tiller install.png)

잠시 기달렸다가… 

~~~
$ kubectl get pods --all-namespaces
~~~

![tiller 확인](/Users/hanjoo/github_blog/assets/image/istio/tiller 확인.png)

입력 후 tiller-deploy의 상태가 Running이 되었음을 확인한 후에...



###6. Istio가 설치된 네임 스페이스 설정 및 만들기

~~~
$ NAMESPACE=istio-system
$ kubectl create ns $NAMESPACE
~~~

![tiller namespace](/Users/hanjoo/github_blog/assets/image/istio/tiller namespace.png)



###7. kiali를 사용하는 경우, kiali 대시 보드의 사용자 이름과 암호 생성

```
$ echo -n 'admin' | base64
YWRtaW4=
$ echo -n '1f2d1e2e67df' | base64
MWYyZDFlMmU2N2Rm
```

~~~
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: kiali
  namespace: $NAMESPACE
  labels:
    app: kiali
type: Opaque
data:
  username: YWRtaW4=
  passphrase: MWYyZDFlMmU2N2Rm
EOF
~~~

![kiali account](/Users/hanjoo/github_blog/assets/image/istio/kiali account.png)



###8. Grafana에 보안 모드를 사용하는 경우,  계정 생성

```
$ echo -n 'admin' | base64
YWRtaW4=
$ echo -n '1f2d1e2e67df' | base64
MWYyZDFlMmU2N2Rm
```

~~~
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: grafana
  namespace: $NAMESPACE
  labels:
    app: grafana
type: Opaque
data:
  username: YWRtaW4=
  passphrase: MWYyZDFlMmU2N2Rm
EOF
~~~

![grafana account](/Users/hanjoo/github_blog/assets/image/istio/grafana account.png)



###9. Istio 설치시 같이 설치할 패키지 선택

~~~
$ vim install/kubernetes/helm/istio/values.yaml
~~~

grafana, prometheus, tracing, kiali -> enabled : **True**



###10.  Istio initializer chart 설치

istio-init을 다른 Istio 차트와 동일한 네임 스페이스 (istio-system)에 설치하는 것이 좋습니다.

```
$ helm install install/kubernetes/helm/istio-init --name istio-init --namespace istio-system
```

![istio init](/Users/hanjoo/github_blog/assets/image/istio/istio init.png)

istio-init 둘다 Staus가 Completed인지 확인한다.

~~~
$ kubectl gets pods --all-namespaces
~~~

![istio-init status](/Users/hanjoo/github_blog/assets/image/istio/istio-init status.png)



### 11. $NAMESPACE에 릴리스 이름 istio가 있는 차트를 설치

~~~
$ helm install install/kubernetes/helm/istio --name istio --namespace istio-system
~~~

![istio init](/Users/hanjoo/github_blog/assets/image/istio/istio init.png)

![isito install_2](/Users/hanjoo/github_blog/assets/image/istio/isito install_2.png)

![istio install_3](/Users/hanjoo/github_blog/assets/image/istio/istio install_3.png)

![istio installl_4](/Users/hanjoo/github_blog/assets/image/istio/istio installl_4.png)

![istio install_5](/Users/hanjoo/github_blog/assets/image/istio/istio install_5.png)



###12. 모든 Istio CRD가 Kubernetes api-server에 커밋되었는지 확인

~~~
$  kubectl get crds | grep 'istio.io\|certmanager.k8s.io' | wc -l
~~~

53 이 나오면 정상!!

![istio crd 확인](/Users/hanjoo/github_blog/assets/image/istio/istio crd 확인.png)



###13. 정상적으로 동작하는지 확인

명령어 입력 후 istio-init 만 completed고 나머지 Running인거 확인

~~~
$ kubectl get pods --all-namespaces
~~~



pod들이 어느 slave-node에 할당이 되었는지 확인 가능하다.

~~~
$ kubectl get pods --all-namespaces -o wide
~~~



pod들이 update 될때마다 모니터링할 수 있다.

~~~
$ kubectl get pods --all-namespaces -o wide --watch
~~~

모든 pod들이 Running 상태이면 성공이다.



---

출처

https://istio.io/docs/setup/kubernetes/#downloading-the-release

https://github.com/istio/istio/tree/master/install/kubernetes/helm/istio

https://github.com/istio/istio/tree/master/install/kubernetes/helm/istio-init