---
layout: post
title:  "Ceph Authentication"
date:   2019-04-09 14:46:59
tags:    [Cloud, Ceph]
comments: true
---

# Ceph Authentication



Ceph -> distributed storage system

   - quorum of monitors
   - scores of MDSs
   - many thousands of OSD daemons (operating across many hosts/nodes-representig the server portion of the Ceph object store)



Ceph client (CephFS, Ceph block device, Ceph Gateway ) interact with the Ceph object store

All Ceph object store clients use the librados library to interact with the Ceph object store.



![img](http://docs.ceph.com/docs/firefly/_images/ditaa-d4b739f8889e03e5d72abddb26ce74425e540539.png)





## Ceph Authentication



A key scalability feature of Ceph 

​	-> to avoid a centralized interface to Ceph object store

​		-> Ceph clients must be able to interact with OSDs directly


cephx authentication system

​	-> authenticates users operating Ceph clients



user/actor 가 Ceph 클라이언트를 호출하여 모니터에 접속합니다. Kerberos와 달리 각 모니터는 사용자를 인증하고 키를 배포 할 수 있으므로 `cephx를` 사용할 때 단일 장애 지점이나 병목 현상이 없습니다. 모니터는 Ceph 서비스를 얻는 데 사용할 세션 키가 들어있는 Kerberos 티켓과 유사한 인증 데이터 구조를 리턴합니다. 이 세션 키는 그 자체로 사용자의 영구 비밀 키로 암호화되므로 사용자 만이 Ceph 모니터에서 서비스를 요청할 수 있습니다.
그런 다음 클라이언트는 세션 키를 사용하여 모니터에서 원하는 서비스를 요청하고 모니터는 실제로 데이터를 처리하는 OSD에 대해 클라이언트를 인증하는 티켓을 클라이언트에 제공합니다. Ceph 모니터와 OSD는 비밀을 공유하므로 클라이언트는 모니터에 제공된 티켓을 클러스터의 OSD 또는 메타 데이터 서버와 함께 사용할 수 있습니다. Kerberos와 마찬가지로, `cephx`티켓이 만료되므로 공격자는 만료 된 티켓이나 세션 키를 은밀하게 사용할 수 없습니다. 이러한 형태의 인증은 사용자의 비밀 키가 만료되기 전에 누설되지 않는 한 통신 매체에 대한 액세스 권한이 있는 공격자가 다른 사용자의 신원하에 가짜 메시지를 작성하거나 다른 사용자의 합법적 인 메시지를 변경하지 못하게합니다.

`cephx` 를 사용하려면 관리자가 먼저 사용자를 설정해야합니다. 다음 다이어그램에서 `client.admin` 사용자는 사용자 이름과 비밀 키를 생성하기 위해 명령 행에서 `ceph auth get-or-create-key` 를 호출합니다 . Ceph의 `인증` 하위 시스템은 사용자 이름과 키를 생성하고 모니터에 사본을 저장 한 다음 사용자의 비밀을 다시 `client.admin` 사용자 에게 전송합니다 . 이는 클라이언트와 모니터가 비밀 키를 공유 함을 의미합니다.



![img](http://docs.ceph.com/docs/firefly/_images/ditaa-6b1dafb6d8f177ab2beb3325857f1e98e4593ec6.png)



모니터로 인증하기 위해, 클라이언트는 사용자 이름을 모니터에 전달하고, 모니터는 세션 키를 생성하고 사용자 이름과 연관된 보안 키로 이를 암호화합니다. 그런 다음 모니터는 암호화 된 티켓을 클라이언트로 다시 전송합니다. 
그런 다음 클라이언트는 세션 키를 검색하기 위해 공유 비밀 키로 페이로드를 암호 해독합니다. 
세션 키는 현재 세션의 사용자를 식별합니다. 그런 다음 클라이언트는 세션 키로 서명 한 사용자를 대신하여 티켓을 요청합니다. 모니터는 티켓을 생성하고 사용자의 비밀 키로 암호화 한 다음 클라이언트로 다시 전송합니다.
클라이언트는 티켓을 암호 해독하고 이를 사용하여 클러스터 전체의 OSD 및 메타 데이터 서버에 대한 요청에 서명합니다.



![img](http://docs.ceph.com/docs/firefly/_images/ditaa-56e3a72e085f9070289331d64453b84ab1e9510b.png)



`cephx의` 프로토콜은 클라이언트 시스템과 Ceph 서버 사이의 지속적인 통신을 인증합니다. 초기 인증 이후에 클라이언트와 서버간에 전송 된 각 메시지는 모니터, OSD 및 메타 데이터 서버가 공유 암호로 확인할 수있는 티켓을 사용하여 서명됩니다.



![img](http://docs.ceph.com/docs/firefly/_images/ditaa-f97566f2e17ba6de07951872d259d25ae061027f.png)





## Ceph Limitations

`cephx의` 프로토콜은 서로 Ceph 클라이언트와 서버를 인증합니다. 사용자를 대신하여 실행되는 사용자 또는 응용 프로그램의 인증을 처리하기위한 것이 아닙니다. 그 효과가 액세스 제어 요구 사항을 처리하는 데 필요한 경우 Ceph 객체 저장소에 액세스하는 데 사용되는 프런트 엔드에만 해당되는 다른 메커니즘이 있어야합니다. 이 다른 메커니즘은 Ceph가 객체 저장소에 액세스하도록 허용 할 시스템에서 허용되는 사용자와 프로그램 만 실행할 수 있도록 보장하는 역할을합니다.

Ceph 클라이언트 및 서버를 인증하는 데 사용되는 키는 일반적으로 신뢰할 수있는 호스트에서 적절한 권한을 가진 일반 텍스트 파일에 저장됩니다. 

<pre>일반 텍스트 파일에 키를 저장하는 것은 보안상의 단점이 있지만 Ceph가 백그라운드에서 사용하는 기본 인증 방법을 고려할 때 피하기가 어렵습니다. Ceph 시스템을 설치하는 사람들은 이러한 단점을 알고 있어야합니다.</pre>

특히, 임의의 사용자 컴퓨터, 특히 휴대용 컴퓨터는 안전하지 않은 컴퓨터에 일반 텍스트 인증 키를 저장해야하기 때문에 Ceph와 직접 상호 작용하도록 구성하면 안됩니다. 그 기계를 훔쳐 갔거나 그것에 대한 은밀한 접근을 얻은 사람은 자신의 기계를 Ceph에게 인증 할 수있는 열쇠를 얻을 수 있습니다.

잠재적으로 안전하지 않은 컴퓨터가 Ceph 개체 저장소에 직접 액세스하는 것을 허용하는 대신 사용자는 자신의 환경에서 충분한 보안을 제공하는 방법을 사용하여 신뢰할 수있는 시스템에 로그인해야합니다. 그 신뢰할 수있는 기계는 인간 사용자를위한 평문 Ceph 키를 저장할 것입니다. Ceph의 향후 버전에서는 이러한 특정 인증 문제를보다 완벽하게 해결할 수 있습니다.

현재 Ceph 인증 프로토콜 중 어떤 것도 전송중인 메시지에 대한 비밀을 제공하지 않습니다. 따라서 와이어의 도청 자 (eavesdropper)는 Ceph의 클라이언트와 서버간에 전송 된 모든 데이터를 듣고 이해할 수 있습니다. 또한 Ceph에는 객체 저장소의 사용자 데이터를 암호화하는 옵션이 포함되어 있지 않습니다. 물론 사용자는 직접 데이터를 손으로 암호화하고 저장할 수 있지만 Ceph는 객체 암호화 자체를 수행 할 수있는 기능을 제공하지 않습니다. Ceph에 민감한 데이터를 저장하는 사람들은 Ceph 시스템에 데이터를 제공하기 전에 데이터를 암호화하는 것을 고려해야합니다.



---

참고문헌

http://docs.ceph.com/docs/firefly/rados/operations/auth-intro/



