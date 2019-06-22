# Kiali 소스 설치



준비사항

1. Go Programming Language 설치
2. git 설치
3. Docker 설치
4. Kubernetes 설치
5. Istio 설치



## Building

### 1. GOPATH 내에서 이 저장소를 복제

본 포스트는 "/source/kiali/kiali"의 GOPATH 예제를 사용하지만 원하는 경로로 수정해도 상관없다.
아래 명령어의 첫 줄을 변경하여 GOPATH를 사용하십시오.

```
$ export GOPATH=/source/kiali/kiali
$ mkdir -p $GOPATH
$ cd $GOPATH
$ mkdir -p src/github.com/kiali
$ cd src/github.com/kiali
$ git clone https://github.com/chohanjoo/kiali.git
$ export PATH=${PATH}:${GOPATH}/bin
```



### 2. Install Glide

Kiali가 자체적으로 구축하기 위해 사용하는 Go 종속성 관리 도구이다.

```
$ cd ${GOPATH}/src/github.com/kiali/kiali
$ make dep-install
```



![deb-install](/Users/hanjoo/github_blog/assets/image/Kiali/deb-install.png)



## Running

### 3. Building the Container Image

Kiali container Image, Kiali operator Image를 생성한다. 

```
$ cd ${GOPATH}/src/github.com/kiali/kiali
$ make docker-build
```



![Screenshot from 2019-06-21 10-56-26](/Users/hanjoo/github_blog/assets/image/Kiali/Screenshot from 2019-06-21 10-56-26.png)

![docker-build](/Users/hanjoo/github_blog/assets/image/Kiali/docker-build.png)



###4. Deploying Kiali Operator

Kiali operator를 먼저 설치해야한다.
이 작업은 한 번만 수행하면됩니다. **operator가 배포 된 후에는 Kiali를 여러 번 배포하고 제거할 수 있습니다.**

```
$ cd ${GOPATH}/src/github.com/kiali/kiali/operator
$ make operator-create
```



![operator-create](/Users/hanjoo/github_blog/assets/image/Kiali/operator-create.png)

![operator-create_2](/Users/hanjoo/github_blog/assets/image/Kiali/operator-create_2.png)



### 5. install Kiali

**Kiali를 설치하려면 [Kiali-operator] 네임스페이스에 Kiali 리소스를 생성하면 된다.**

아래 명령어를 통해 default 설정된 Kiali를 설치할 수 있다.

~~~
$ /usr/bin/kubectl apply -n kiali-operator -f https://raw.githubusercontent.com/kiali/kiali/master/operator/deploy/kiali/kiali_cr.yaml
~~~

![create-kiali-default](/Users/hanjoo/github_blog/assets/image/Kiali/create-kiali-default.png)

만약 리소스 파일을 커스텀하여 Kiali를 설치하려면 다음 명령어를 사용한다.

~~~
$ cd ${GOPATH}/src/github.com/kiali/kiali/operator
$ make kiali-create
~~~





### 6. create account

~~~
$ /usr/bin/kubectl create secret generic kiali -n istio-system --from-literal 'username=admin' --from-literal 'passphrase=admin'
~~~



### 7. open Kiali UI

~~~
$ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001
~~~



![kiali ui](/Users/hanjoo/Downloads/kiali install from source/kiali ui.png)



