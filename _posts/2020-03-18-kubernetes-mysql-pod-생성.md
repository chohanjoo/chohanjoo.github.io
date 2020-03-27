---
title: "[sns project] Kubernetes Mysql Pod 생성"
description: 쿠버네티스 클러스터 mysql pod를 생성한다.
categories:
 - project
 - cloud
tags:
- project
- sns
- kubernetes
- cloud
---

# Kubernetes Mysql Pod 생성하기

포스트의 내용은 [블로그]([https://velog.io/@pa324/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-mysql-%EC%84%A4%EC%B9%98-6bjxv4dcoa](https://velog.io/@pa324/쿠버네티스-mysql-설치-6bjxv4dcoa)) 를 요약한 것입니다.

## 개요

pod에 부하가 걸리거나, 이상 징후가 있을 경우 pod가 재시작되기 때문에 모든 데이터가 초기화 된다.
따라사, mysql 컨테이너를 kubernetes pod를 통해서 구동시킬 때, persistent volume을 설정해야 한다.



## persistent volume 이란?

쿠버네티스에는 volume이라는 저장공간이 있다. pod가 생성될 때 volume이 만들어지고, pod를 삭제할 때 volume도 삭제된다.
pod가 재시작될 때는 volume을 계속 참조하기 때문에 데이터 영속성을 제공한다.

### volume type

- emptyDir - 임시저장할 데이터를 보관하는 volume
- hostPath - worker node의 파일 시스템에 mounting 시키는 volume
- gitRepo - git repository에 mounting 시키는 volume



## persistent volume claim 이란?

Persistent storage에 동적으로 프로비젼 시키는 설정이다

(1). 사용자가 pod에서 사용할 persistent volume이 필요한 경우 kubernetes를 통해서 PersistentVolumeClaim을 생성한다.
(2). Kubernetes API server에게 PersistentVolumeClaim을 넘겨준다.
(3). Kubernetes는 적합한 PersistentVolume을 찾고 PersistentVolumeClaim과 바인딩 시킨다.
    그리고 나서 pod에서는 PersistentVolumeClaim을 통해서 volume을 설정할 수 있다.

![mysql](/assets/images/sns/mysql.png)



## persistentVolume, persistentVolumeClaim 설정하기

### persistentVolume, persistentVolumeClaim 생성

파일명은 `mysql-pv.yaml` 로 한다. 
volume 이름은 `mysql-pv-volume`으로 한다.
hostPath 타입으로 volume을 생성하고, worker node의 `/Users/hj/mnt/data` 경로에 mysql 파일이 저장되도록 설정한다.
** 프로젝트의 환경이 MAC OS 이기 때문에 home 디렉터리에 mnt/data 디렉터리를 생성하였다.



~~~yml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: mysql-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/Users/hj/mnt/data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
~~~



### persistent volume pod 생성

~~~shell
$ kubectl apply -f mysql-pv.yaml
~~~



## mysql pod 생성

### deployment, service yaml 파일 생성

containers 항목을 보면 volumeMounts를 확인할 수 잇다.
Mysql-persistent-storage라는 이름으로 volume으로 `/var/lib/mysql (컨테이너 내부에서 실제로 저장되는 경로)` 디렉토리를 마운트 시킨다.
파일명은 `mysql-deployment.yaml` 로 한다.

~~~yml
apiVersion: v1
kind: Service
metadata:
  name: mysql-cluster-ip-service
spec:
  clusterIP: 10.101.204.217
  ports:
  - port: 3306
  selector:
    app: mysql
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      nodeName: docker-desktop
      containers:
      - image: mysql:5.6
        name: mysql
        env:
          # Use secret in real usage
        - name: MYSQL_ROOT_PASSWORD
          value: root
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
~~~



### mysql pod 생성

 ~~~shell
$ kubectl apply -f mysql-deployment.yaml
 ~~~



## 정상적으로 mount 되는지 확인하기

### mysql pod에 bash를 이용해 접속하기

~~~shell
$ kubectl exec -it [mysql pod 이름] - bash
~~~

### mysql 접속하기

~~~shell
$ mysql -u root -p root
~~~

### 데이터베이스 test 생성

~~~shell
$ CREATE DATABASES test;
~~~



## MysqlWorkbench에 연결하기

### port forwarding

~~~shell
$ kubectl port-forward [mysql pod 이름] 3307:3306
~~~

Local 의 mysql 이 3306 포트를 사용하고 있으므로 mysql pod는 3307 포트를 사용한다.

### MysqlWorkbench에 연결



---

출처

[쿠버네티스 - mysql 설치]([https://velog.io/@pa324/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-mysql-%EC%84%A4%EC%B9%98-6bjxv4dcoa](https://velog.io/@pa324/쿠버네티스-mysql-설치-6bjxv4dcoa))

