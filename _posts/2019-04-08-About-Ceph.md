    ---
    layout: post
    title:  "Ceph란?"
    date:   2019-04-08 15:54:59
    author: Hanjoo Cho
    categories: Cloud
    tags:    Cloud
    cover:  "/assets/instacode.png"
    ---

# Ceph



## 정의

- SDS(software Defined Storage) 서비스를 제공하는 솔루션 중 하나

- PB 규모의 리눅스 분산 파일 시스템

- block, file, object를 저장하고 워크로드에 따라 클러스터 노드를 계속 확장 할 수 있는 scale out 형태로 제공

Ceph Object Storage 서비스, Ceph Block Device 서비스, Ceph FileSystem 서비스를 제공

이 서비스를 사용하기 위해서는 Ceph라는 Storage Cluster를 구축해야 한다.

Ceph Cluster는 OSD(OSD Daemon), Monitor, MDS(Metadata Server)의 3가지 서비스로 구성되어있다.

*클러스터 구성에는 최소 1개의 Ceph Monitor와 최소 2개의 OSD 데몬이 필요하다.

(디폴트는 3개의 OSD이다 - 3개의 복제본을 저장해야 하기 때문에)



1) Ceph OSDs

* Object Storage Daemon
* OSD는 데이터를 저장하고, 복제, 부하분산 등의 역할을 한다.
* 간단하게 Ceph 데이터를 저장하는 저장소이다.
* OSD 디스크는 1TB당 메모리 1G이상 구성해야 한다.
* disk 하나당 OSD 하나가 할당되는 개념
* 무결성과 성늘을 위한 journal을 사용한다,
* Cluster내에 10 ~ 10000s OSD를 사용할 수 있다.
* 각 object에 대한 저장소 연결을 직접 수행한다.
* osd daemon은 data 재분배의 역할을 수행한다.

2) MON (Monitors)

- 모니터는 클러스터의 상태를 체크하고, PG(Placement Group) map, OSD map 등을 관리한다.

- Ceph와 state history를 저장하고 관리한다.

- ceph cluster당 1개의 monitor가 존재해도 되지만 3개 이상의 monitor를 권장한다.

- crush map 및 cluster map을 client에게 전달한다.

- monitor는 저장되는 데이터는 존재하지 않고 cluster map을 소유하고 client로부터의 request가 발생되면

  cluster map을 전달하는 역할을 수행한다. 또한 주기적으로 lightweight 한 monitor daemon과 정보를 교환한다.

> ## OSD heartheat
>
> 각 OSD는 서로간에 heartbeat을 주고(default 6초) 받는다. 20초간 heartbeat를 받지 못하는 경우 해당 OSD에 대해서 monitor에게 report한다. 
>
> 다수의 OSD가 down되었다고 report를 하는 경우 monitor는 down되었다고 report된 OSD를 down으로 mark

3) MDS

- Ceph Metadata Server는 일반 사용자가 Ceph 데이터를 검색 및 체크 하기 위해 metadata를 저장하는 서버

  (Ceph Block Device, Ceph Object Storage는 MDS를 사용하지 않는다.)

4) Journal

* 각 OSD는 각각의 journal을 가지고 있다.
* journal을 두는 이유는 속도와 일관성 측면이다.
* ceph에 write operation이 발생되면 최초로 journal에 먼저 기록되게 된다.
* 실제 사용은 write에 대해서만 관여하게 된다.



## 개념도

![img](https://thebook.io/img/006881/053_01.jpg)



## RADOS

- Reliable : 복제를 통해 데이터 분실을 회피한다.
- Autonomic : 서로 통신하여 failure를 감지하고 투명한 복제를 수행
- Distributed
- Object Store

Ceph의 기반이며 모든 것들은 RADOS에 저장된다.
Ceph의 중요한 기능을 제공한다.
( 분산 오브젝트 저장, HA, 신뢰성, no SPOF, self healing, self managing)
CRUSH 알고리즘을 포함한다.
모든 Data는 RADOS를 거쳐 object로 저장하게 된다.
데이터 타입을 구분하지 않는다. (즉 object, block, file 등을 구분하지 않고 최종 object로 저장한다.)
object chunk는 4MB이다.



-------

아래 방식들로 접근하기 위해서는 다음과 같은 파일이 필요하다.

- /etc/ceph/ceph.conf
- /etc/ceph/ceph,client,admin.keyring

해당 파일을 복사해 client로 사용할 서버에 두어야 한다.



1. librados

   c/c++/python/ruby 등이 사용가능

   TCP/IP 통신을 기반한 raw socket를 사용하여 통신

   HTTP를 사용하지 않고 direct로 연결이 되기 때문에 HTTP overhead를 가지지 X

2. RadosGW

   Rest (swift 등) interface를 통해 Rados cluster에 object를 저장한다.

3. RBD 

   Block Storage

   RADOS에서 disk image의 저장소

   Host로부터 VM들을 분리

   object block size는 기본이 4M이고 4k부터 32M까지 설정 가능

4. CephFS

   RADOS내에 존재하는 metadata 서버를 연결하여 file정보를 확인한 후 연결을 수행



-----

참고문헌

https://www.jacobbaek.com/594

https://m.blog.naver.com/PostView.nhn?blogId=pap5608&logNo=220569478476&proxyReferer=https%3A%2F%2Fwww.google.com%2F



