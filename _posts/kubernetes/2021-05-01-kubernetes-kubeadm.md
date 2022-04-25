---
title: \[Kubernetes\]\(Kubeadm\)HA Configuration3-Kubeadm in CentOS
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Kubeadm
- Cluster
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
kubernetes 고가용성 클러스터 구성

클러스터의 정합성을 유지하기 위해 최소 ControlPlane 노드 3개 이상의 홀수 개수가 권장 된다.

### Kubeadm Ha topology

<center><img src="/assets/images/posts/kubernetes/kubeadm/kubeadm-ha-topology.svg" width="150%" height="150%"></center>

---
# 공통

### vim /etc/hosts

```bash
10.50.107.21    master1
10.50.107.22    master2
10.50.107.23    VIP
10.50.107.24    worker1
10.50.107.25    worker2
10.50.107.26    master3
```

## 0. Installing runtime (docker version 확인)

### Install docker

[Install Docker Engine on CentOS](https://docs.docker.com/engine/install/centos/)

[컨테이너 런타임](https://kubernetes.io/ko/docs/setup/production-environment/container-runtimes/)

```bash
##
yum install -y yum-utils

##
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

## docker 설치 (최신버전의 경우 설치 안될수도 있음)
yum install docker-ce docker-ce-cli containerd.io -y

yum install -y \
  containerd.io-1.2.13 \
  docker-ce-19.03.11 \
  docker-ce-cli-19.03.11

## cgroup 변경
mkdir -pv /etc/docker
## 도커 데몬 설정
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "insecure-registries":["core.harbor.domain:32260"]
}
EOF

## /etc/systemd/system/docker.service.d 생성
mkdir -p /etc/systemd/system/docker.service.d

## 도커 데이터 경로 변경
mkdir -pv /data/docker

vim /lib/systemd/system/docker.service
#ExecStart=/usr/bin/dockerd -H fd://
ExecStart=/usr/bin/dockerd -g /data/docker -H fd:// --containerd=/run/containerd/containerd.sock

## daemon start
systemctl enable docker &&systemctl restart docker && systemctl status docker

## Checking install docker
docker version

##  Checking cgroup
docker info | grep -i "cgroup"

######## 전체 복사 ( docker data 경로 변경 제외 )
yum install -y yum-utils
yum-config-manager   --add-repo    https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io -y
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "insecure-registries":["core.harbor.domain:32260"]
}
EOF
mkdir -p /etc/systemd/system/docker.service.d
systemctl restart docker && systemctl enable docker
docker version
docker info | grep -i "cgroup"
```

### docker data 경로 설정

/lib/systemd/system/docker.service 파일을 열고 아래 내용을 수정한다. 

```bash
## 디렉토리 생성
mkdir -pv /data/docker

#ExecStart=/usr/bin/dockerd -H fd://
ExecStart=/usr/bin/dockerd -g /data/docker -H fd:// --containerd=/run/containerd/containerd.sock
```

기존에 ExecStart 부분에 docker 데이터가 저장되는 위치를 지정한다.

수정이 되면 도커를 중지시킨다.

### 그런 다음 기존에 docker 경로를 위 경로로 통째로 복제해준다.

```bash
## 변경 되는 경로에 데이터 복사
cp -aprv /var/lib/docker /data/
sudo rsync -aqxP /var/lib/docker /data/ 

## 이후 docker start
sudo systemctl start docker  

## 경로 확인
ps -ef |grep docker
```

### 사용하지 않는 볼륨 제거 (컨테이너, 네트워크, 이미지)

```bash
## delete images older than 6 months ago; 4320h = 24 hour/day * 30 days/month * 6 months
docker system prune --filter "until=4320h" -f
```

### 일반계정으로 docker 사용

```bash
# docker 그룹 생성 docker 설치시 자동 생성
sudo groupadd docker
# 사용자 추가
sudo usermod -aG docker asmanager
# 로그아웃 후 다시 로그인해서 그룹에 사용자로 추가 되었는지 확인.
```

## 1. Disable firewall

```bash
systemctl stop firewalld &* systemctl disable firewalld
```

## 2. Swap disabled (Must)

```bash
swapoff -a && sed -i '/swap/s/^/#/' /etc/fstab
```

## 3. Setting iptables see bridged traffic

CentOS 같은 리눅스 배포판은 net.bridge.bridge-no-call-iptables 값이 디폴트 0 (zero)다

이는 bridge 네트워크를 통해 송수신되는 패킷이 iptables 설정을 우회한다는 의미다

컨테이너의 네트워크 패킷이 호스트머신의 iptables 설정에 따라 제어되도록 하는 것이 바람직하며 이를 위해서는 이 값을 1로 설정해야 한다

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

## 4. Selinux setting

```bash
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

## 5. Installing kubeadm, kubelet and kubectl

### Adding repo

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
```

```bash
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable kubelet && systemctl start kubelet && systemctl status kubelet
```

---
# Only Master nodes

## 1. Allow root login When using ssh

```bash
## root 로그인 허용
sed -i 's/PermitRootLogin no/PermitRootLogin yes/g' /etc/ssh/sshd_config
systemctl restart sshd
```

## 2. Create ssh connection

[asmanager@JN0182 .ssh]$ ssh-copy-id -i ~/.ssh/id_rsa.pub [asmanager@10.50.107.22](mailto:asmanager@10.50.107.22)

```bash
ssh-keygen -t rsa

ssh-copy-id 10.50.107.22 -p 2211

Now try logging into the machine, with:   "ssh -p '2211' '10.50.107.22'"
and check to make sure that only the key(s) you wanted were added.

root@AJTV005 [~]ssh 10.50.107.22 -p 2211
Last login: Fri Dec  4 11:16:11 2020
root@AJTV006 [~]
```

## 3. Kubeadm 구성 (단일 Master or HA)

```bash
kubeadm init

kubeadm init --apiserver-advertise-address=<<control-plane node IP>>

## 단일 Master 구성
kubeadm init --apiserver-advertise-address=10.50.107.21 --upload-certs --pod-network-cidr=10.244.0.0/16

## HA 구성시 아래 명령어로 실행
kubeadm init --control-plane-endpoint "<노드1 DNS/IP or LoadBalancer DNS/IP>:26443" \
                --upload-certs \
                --pod-network-cidr "10.244.0.0/16"
kubeadm init --control-plane-endpoint "10.50.103.133:8443" \
                --upload-certs \
                --pod-network-cidr "10.254.0.0/16"
```

설치가 진행되고 master,worker join 을 위한 **토큰을 저장**해둔다.  

### option

- -apiserver-advertise-address string

The IP address the API Server will advertise it's listening on. If not set the default network interface will be used.

- -control-plane-endpoint string

Specify a stable IP address or DNS name for the control plane.

- -upload-certs

Upload control-plane certificates to the kubeadm-certs Secret. 

- -pod-network-cidr string

Specify range of IP addresses for the pod network. If set, the control plane will automatically allocate CIDRs for every node.

### 인증서를 가진 유저만 kubectl 명령어를 사용할 수 있다.

```bash
**To start using your cluster, you need to run the following as a regular user:**

**mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config**
```

./kube/config 안에 key 가 있다.

## 4. Install Weave on master1

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d 'n')&env.IPALLOC_RANGE=10.253.0.1/16"

## 삭제
kubectl delete -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

```

이후 

```bash
kubectl get nodes
```

명령어를 확인해보면 STATUS 가 READY 가 된다.

## 5. Join with other master node

```bash
kubeadm join 10.50.107.23:8443 --token 5yig3i.v60r5ecz56ufoucq     --discovery-token-ca-cert-hash sha256:790dbec7bba3afe9c6aec68b7dcc8dff700aa55c1fa4d390a55d23a8cc725349     --control-plane --certificate-key a27731afbc0c2dc206279921d7374e24f8999c6875e636548a9623dcf6945ca0
```

## 6. Join with other work node

```bash
kubeadm join 10.50.107.23:8443 --token 5yig3i.v60r5ecz56ufoucq \
    --discovery-token-ca-cert-hash sha256:790dbec7bba3afe9c6aec68b7dcc8dff700aa55c1fa4d390a55d23a8cc725349
```