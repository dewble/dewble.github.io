---
title: \[Kubernetes\]\(Kubeadm\) HA Configuration1-keepalived
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Kubeadm
- Keepalived
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
Create VIP for HA configuration  


# What is keepalived ?
:VRRP 기능을 이용하는 서버 다중화 도구

마스터 서버와 나머지 백업 서버가 동일한 서비스 아이피를 가진다.  
평상 시에는 클라이언트는 마스터 서버로만 접근한다.
마스터 서버는 백업 서버보다 우선순위 값이 높고,  
마스터 서버가 다운되면 백업 서버 중 가장 우선순위 값이 높은 서버가 마스터로 전환 된다.  
마스터 서버가 되살아나면 다시 마스터 역할을 돌려 받게된다.

# Install Keepalived on all master node

```bash
root@AJTV005 [~]yum install -y keepalived
root@AJTV005 [/etc/keepalived]cp keepalived.conf keepalived.conf.bak
```

## config example

```bash
! Configuration File for keepalived

global_defs {
   router_id K8sCP2DEV
}

vrrp_instance VI_1 {
    state MASTER
    interface ens160            # 사용할 인터페이스 설정
    virtual_router_id 51        # Master Node 모두 같은 값
    priority 150                # 우선순위 설정 (max 255) 높은 곳에 IP 생성
    advert_int 1                # VRRP패킷 송신 간경 설정 (seconds)
    authentication {
        auth_type PASS          # 평문 인증 설정
        auth_pass joins         # 인증을 위한 키 (Master Node 모두 같은 값)
    }
    virtual_ipaddress {
       10.50.103.133             # VIP 설정
    }
}
```

ControlPlane 2 에서는  1 과 동일하게 구성하되 priority와 라우터 아이디만 다르게 설정 한다.

## master1

```bash
! Configuration File for keepalived

global_defs {
   router_id k8s-master2
}

vrrp_instance VI_1 {
    state MASTER
    interface ens192
    virtual_router_id 51
    priority 151
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass joins
    }
    virtual_ipaddress {
        10.50.107.23
    }
}
```

## master2

```bash
! Configuration File for keepalived

global_defs {
   router_id k8s-master1
}

vrrp_instance VI_1 {
    state MASTER
    interface ens192
    virtual_router_id 51
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass joins
    }
    virtual_ipaddress {
        10.50.107.23
    }
}
```

```bash
systemctl enable keepalived && systemctl restart keepalived && systemctl status keepalived && ip a
```

## service 실행 후 가상 IP 확인

```bash
root@AJTV005 [/etc/keepalived]systemctl restart keepalived.service
root@AJTV005 [/etc/keepalived]systemctl status keepalived
● keepalived.service - LVS and VRRP High Availability Monitor
   Loaded: loaded (/usr/lib/systemd/system/keepalived.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2020-12-04 15:07:47 KST; 6s ago
  Process: 6461 ExecStart=/usr/sbin/keepalived $KEEPALIVED_OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 6462 (keepalived)
   CGroup: /system.slice/keepalived.service
           ├─6462 /usr/sbin/keepalived -D
           ├─6463 /usr/sbin/keepalived -D
           └─6464 /usr/sbin/keepalived -D

Dec 04 15:07:49 AJTV005 Keepalived_vrrp[6464]: VRRP_Instance(VI_1) Transition to MASTER STATE
Dec 04 15:07:50 AJTV005 Keepalived_vrrp[6464]: VRRP_Instance(VI_1) Entering MASTER STATE
Dec 04 15:07:50 AJTV005 Keepalived_vrrp[6464]: VRRP_Instance(VI_1) setting protocol iptable drop rule
Dec 04 15:07:50 AJTV005 Keepalived_vrrp[6464]: VRRP_Instance(VI_1) setting protocol VIPs.
Dec 04 15:07:50 AJTV005 Keepalived_vrrp[6464]: Sending gratuitous ARP on ens192 for 10.50.107.23
Dec 04 15:07:50 AJTV005 Keepalived_vrrp[6464]: VRRP_Instance(VI_1) Sending/queueing gratuitous ARPs on ens192 for 10.50.107.23
Dec 04 15:07:50 AJTV005 Keepalived_vrrp[6464]: Sending gratuitous ARP on ens192 for 10.50.107.23
Dec 04 15:07:50 AJTV005 Keepalived_vrrp[6464]: Sending gratuitous ARP on ens192 for 10.50.107.23
Dec 04 15:07:50 AJTV005 Keepalived_vrrp[6464]: Sending gratuitous ARP on ens192 for 10.50.107.23
Dec 04 15:07:50 AJTV005 Keepalived_vrrp[6464]: Sending gratuitous ARP on ens192 for 10.50.107.23
root@AJTV005 [/etc/keepalived]systemctl enable keepalived
Created symlink from /etc/systemd/system/multi-user.target.wants/keepalived.service to /usr/lib/systemd/system/keepalived.service.
```

## IP 확인

```bash
root@AJTV005 [/etc/keepalived]ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:1d:81:a6 brd ff:ff:ff:ff:ff:ff
    inet 10.50.107.21/24 brd 10.50.107.255 scope global ens192
       valid_lft forever preferred_lft forever
    inet 10.50.107.23/32 scope global ens192
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe1d:81a6/64 scope link
       valid_lft forever preferred_lft forever
```