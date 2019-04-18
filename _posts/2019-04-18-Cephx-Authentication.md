---
layout: post
title:  "Cephx Authentication"
date:   2019-04-18 18:15:59
author: Hanjoo Cho
categories: Cloud
tags:    Cloud Ceph
cover:  "/assets/instacode.png"
---

# Cephx Authentication

Ceph는 두 가지 인증 모드를 제공

- None : Any user can access data without authentication
- Cephx : Ceph는 Kerberos와 유사한 방식으로 사용자 인증을 요구

cephx를 비활성화 하면 여기에 설명 된 절차를 사용하여 키를 생성 할 필요가 없다.
cephx를 다시 사용하고 이미 키를 생성 한 경우, 키를 다시 생설 할 필요가 없다.



## Configuring Cephx (Cephx 구성하기)

Ceph 클러스터와 그 데몬을 위해 `cephx` 프로토콜 을 활성화하기 위해 따라야하는 몇 가지 중요한 절차가 있습니다 :

1. 관리자가 Ceph 명령을 실행할 수 있도록 기본 `client.admin` 사용자에 대한 비밀 키를 생성해야 합니다.
2. 모니터 비밀 키를 생성하여 클러스터의 모든 모니터에 분배해야합니다.
3. 인증 을 사용하려면 [Enabling Cephx](http://docs.ceph.com/docs/emperor/rados/operations/authentication/#enabling-cephx) 의 나머지 단계를 따라야 합니다.



### The CLIENT.ADMIN KEY

Ceph를 처음 설치할 때 명령 줄에서 실행하는 각 Ceph 명령은 사용자 가 `client.admin` 기본 사용자 라고 가정합니다 . `cephx` 를 사용 하여 Ceph 를 실행하는 경우 `ceph` 명령을 관리자로 실행 하려면 `client.admin` 사용자 의 키가 있어야합니다 .

> cephx가 활성화 된 명령 행에서 Ceph 명령을 실행하려면`client.admin` 사용자에 대한 키를 작성하고 `/etc/ceph`에 secret file 을 작성해야합니다 .



다음 명령은 admin 기능을 사용하여 모니터 에서 `client.admin` 키를 생성하고 등록하고 이를 로컬 파일 시스템의 키링에 기록합니다. 키가 이미 존재하면, 현재 값이 리턴됩니다.

~~~
sudo ceph auth get-or-create client.admin mds 'allow' osd 'allow *' mon 'allow *' > /etc/ceph/keyring
~~~



### MONITOR KEYRING

Ceph는 a keyring for the monitors 필요합니다. 
Use the [ceph-authtool](http://docs.ceph.com/docs/emperor/man/8/ceph-authtool) command to generate a secret monitor key and keyring.

```
sudo ceph-authtool {keyring} --create-keyring --gen-key -n mon.
```

여러 모니터가 있는 클러스터는 모든 모니터에 대해 동일한 키링을 가져야합니다. 따라서 다음 디렉토리 아래의 각 모니터 호스트에 키링을 복사해야합니다.

~~~
/var/lib/ceph/mon/$cluster-$id
~~~



### ENABLING CEPHX (Cephx 활성화)

When `cephx` is enabled, Ceph will look for the keyring in the default search path, which includes `/etc/ceph/keyring`. You can override this location by adding a `keyring` option in the `[global]` section of your [Ceph configuration](http://docs.ceph.com/docs/emperor/rados/configuration/ceph-conf) file, but this is not recommended.

Execute the following procedures to enable `cephx` on a cluster with `cephx` disabled. **If you (or your deployment utility) have already generated the keys, you may skip the steps related to generating keys.** 

1. `client.admin` 키를 작성하고 클라이언트 호스트의 키 사본을 저장하십시오.

~~~
ceph auth get-or-create client.admin mon 'allow *' mds 'allow *' osd 'allow *' -o /etc/ceph/keyring

**Warning:** This will clobber any existing ``/etc/ceph/keyring`` file. Be careful!
~~~

2. Generate a secret monitor `mon.` key

```
ceph-authtool --create --gen-key -n mon. /tmp/monitor-key
```

3. Copy the mon keyring into a `keyring` file in every monitor’s `mon data` directory

~~~
cp /tmp/monitor-key /var/lib/ceph/mon/ceph-a/keyring
~~~

4. Generate a secret key for every MDS, where `{$id}` is the MDS letter:

```
ceph auth get-or-create mds.{$id} mon 'allow rwx' osd 'allow *' mds 'allow *' -o /var/lib/ceph/mds/ceph-{$id}/keyring
```

6. Enable `cephx` authentication for versions `0.51` and above by **setting the following options in the `[global]` section of your [Ceph configuration](http://docs.ceph.com/docs/emperor/rados/configuration/ceph-conf) file**

```
auth cluster required = cephx
auth service required = cephx
auth client required = cephx
```

7. Or, enable `cephx` authentication for Ceph versions `0.50` and below by setting the following option in the `[global]` section of your [Ceph configuration](http://docs.ceph.com/docs/emperor/rados/configuration/ceph-conf) file. **NOTE:** Deprecated as of version `0.50`.

```
auth supported = cephx
```

8. Start or restart the Ceph cluster. See [Operating a Cluster](http://docs.ceph.com/docs/emperor/rados/operations/operating) for details.

---

참고문헌

<http://docs.ceph.com/docs/emperor/rados/operations/authentication/>

