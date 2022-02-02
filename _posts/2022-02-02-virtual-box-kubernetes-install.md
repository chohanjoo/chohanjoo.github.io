---
title: "VirtualBox Ubuntu 환경에 Kubernetes 구축하기"
description: VirtualBox Ubuntu 환경에 Kubernetes 구축하기.
categories:
 - kubernetes
tags:
- kubernetes
- virtualbox
---

_본 게시글은 멋쟁이사자처럼 강의를 들으며 정리를 한 내용이다._

# Virtual Box Ubuntu Server 고정 IP 할당하기

## 네트워크 설정

1) NAT 
2) 어댑터에 브리지

<br/>

## 네트워크 인터페이스 설정 

```bash
$ sudo vim /etc/network/interfaces
```

```
# /etc/network/interfaces

auto lo
iface lo inet loopback

auto enp0s3
iface enp0s3 inet dhcp

auto enp0s8
iface enp0s8 inet static
address 192.168.56.10 # 할당할 고정 IP
netmask 255.255.255.0
network 192.168.56.0
```

```bash
$ sudo systemctl restart networking
```

<br/>

> Failed to restart network.service: Unit network.service not found
<br/>
<br/>
해당 에러 발생시에 ifupdown 패키지를 설치한다.
<br/>
>> $ sudo apt install ifupdown

<br/>

### Network connection 확인

아래 두 명령어를 통해 네트워크가 정상적으로 연결되었는지 확인한다.

```bash
$ ping 8.8.8.8

$ ping 192.168.56.20 # 클러스터로 연결할 VM
```

### VM 복제

VirualBox 에서 VM 을 복제한다. (완전한 복제)

아래 명령어를 통해 hostanme, user 를 변겯한다.

```
# hostnamectl set-hostname slave01

# sudo adduser 임시유저
# sudo adduser 임시유저 sudo

### 임시유저로 로그인 

# sudo usermod -l 변경원하는유저이름 바꾸려는유저이름

# sudo usermod -d /home/바꾸려는유저이름 -m 바꾸려는유저이름


### 변경한 유저로 로그인 


# sudo deluser 임시유저
# sudo rm -r /home/임시유저
```


<br/>
<br/>

# Kubernetes 설치하기

설치는 `kubeadm` 을 통해 진해아하였다. 자세한 설명은 [documnet](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) 를 참고한다.

## VM 사양확인

cpu 가 2개가 limit 이므로 vm 프로세서에서 설정해준다.


<br/>

## iptable 설정

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

<br/>

이 때, 같이 swap 도 비활성화해주자. 
swap 이 설정되어 있으면, kubelet 이 올라오지 않는다.



```
sudo swapoff -a  
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

자세한 설명은 [stackoverflow](https://stackoverflow.com/questions/70229371/kubelet-service-is-not-running-it-seems-like-the-kubelet-isnt-running-or-healt) 를 참고한다.

<br/>

## docker 설치

```
# apt update
# apt install -y docker.io
```

<br/>

## kubelet, kubeadm, kubectl 설치

```
# cat <<EOF > kube_install.sh
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
EOF

# bash kube_install.sh

# cat <<EOF > /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

# service docker restart
```

<br/>

## kubeadm init 

```
# kubeadm init --apiserver-advertise-address=192.168.56.10
```

master node 의 ip 를 명시해준다. 

(명시하지 않으면, 10.0.2.15 로 클러스터링 해야해서 정상적으로 되지 않는다.)

<br/>

## network cni 설정

weave net 을 사용하여 구축하였다.

```
# kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```


> GET http://127.0.0.1:6784/status connection failed 에러가 발생하며 Weave net, core dns 가 안올라온다면... core dns 의 ip 범위를 iptable 에 추가해준다. 
자세한 설명은 [stackoverflow](https://stackoverflow.com/questions/54550285/readiness-probe-failed-error-in-weave-kubernetes) 를 참고한다.
<br/>
>> $ iptables -t nat -I KUBE-SERVICES -d 10.96.0.1/32 -p tcp -m comment --comment "default/kubernetes:https cluster IP" -m tcp --dport 443 -j KUBE-MARK-MASQ



