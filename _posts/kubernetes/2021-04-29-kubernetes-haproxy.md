---
title: \[Kubernetes\]\(Kubeadm\)HA Configuration2-HAProxy
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Kubeadm
- HAProxy
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
Load balance VIP

# Install haproxy on all master node

## Install

```bash
root@AJTV005 [/]yum install haproxy -y
root@AJTV005 [/]cd /etc/haproxy
root@AJTV005 [/etc/haproxy]cp haproxy.cfg haproxy.cfg.bak
```

## config example1
```bash
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     81920
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    #stats socket /var/lib/haproxy/stats
    # seamless reload 를 위한 status socket 운영
    stats socket /var/lib/haproxy/stats mode 777 level admin expose-fd listeners

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    #mode                    http
    log                     global
    #option                  httplog
    option                  dontlognull
    #option http-server-close
    #option forwardfor       except 127.0.0.0/8
    #option                  redispatch
    retries                 3
    timeout http-request    10s
    #timeout queue           1m
    timeout connect         5s
    timeout client          10s
    timeout server          10s
    timeout http-keep-alive 10s
    timeout check           5s
    maxconn                 10000

#---------------------------------------------------------------------
# mail server
mailers mta
        mailer smtp1 192.168.2.151:25

#---------------------------------------------------------------------
listen elk_192.168.171.159_9200
        mode tcp
        bind 192.168.171.159:9200
        maxconn 8000
        balance roundrobin
        server elk_1 192.168.171.155:9200 check inter 2s fall 3 rise 2
        server elk_2 192.168.171.156:9200 check inter 2s fall 3 rise 2
        server elk_3 192.168.171.157:9200 check inter 2s fall 3 rise 2
        email-alert mailers mta
        email-alert level notice
        email-alert from mail@mail.com
        email-alert to mail@mail.com

listen stats
        mode http
        bind 192.168.171.159:80
        maxconn 100
        stats enable
        stats hide-version
        stats uri /stats
        stats auth admin:admin199
#---------------------------------------------------------------------
```

## ControlPlane

```bash
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     81920
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    #stats socket /var/lib/haproxy/stats
    # seamless reload 를 위한 status socket 운영
    stats socket /var/lib/haproxy/stats #mode 777 level admin expose-fd listeners

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    #mode                    http
    log                     global
    #option                  httplog
    option                  dontlognull
    #option http-server-close
    #option forwardfor       except 127.0.0.0/8
    #option                  redispatch
    retries                 3
    timeout http-request    10s
    #timeout queue           1m
    timeout connect         5s
    timeout client          10s
    timeout server          10s
    timeout http-keep-alive 10s
    timeout check           5s
    maxconn                 10000

#---------------------------------------------------------------------
listen K8s_10.50.103.133_8443
        mode tcp
        bind 10.50.103.133:8443
        maxconn 8000
        balance roundrobin
        server CP1 10.50.103.134:6443 check inter 2s fall 3 rise 2
        server CP2 10.50.103.135:6443 check inter 2s fall 3 rise 2
        server CP3 10.50.103.136:6443 check inter 2s fall 3 rise 2

## admin page
listen stats
        mode http
        bind 10.50.103.133:80
        maxconn 100
        stats enable
        stats hide-version
        stats uri /stats
        stats auth admin:admin123!@
#---------------------------------------------------------------------
```

## 컨피그 검사

```bash
root@AJTV005 [/etc/haproxy]haproxy -f /etc/haproxy/haproxy.cfg
```

```bash
systemctl enable haproxy && systemctl restart haproxy && systemctl status haproxy
```

# HAproxy admin page
<center><img src="/assets/images/posts/kubernetes/kubeadm-haproxy.png" width="150%" height="150%"></center>
HAProxy 에서 제공하는 stats 페이지에서 통신 상태를 확인할 수 있다.


# slave 서버에서 VIP 확인이 안될 경우
## Edit sysctl.conf
```bash
cat << EOF >> /etc/sysctl.conf
# 양쪽 노드에서 /etc/sysctl.conf 파일에 IP 가 포워딩될 수 있도록
net.ipv4.ip_forward = 1

# 로컬 호스트 주소 이외의 다른 가상 IP에 바인딩할 수 있도록
net.ipv4.ip_nonlocal_bind = 1

# 자신의 네트워크가 스푸핑된 공격지의 소스로 사용되는것을 차단
net.ipv4.conf.default.rp_filter = 2

# 스푸핑을 막으려고 source route 패킹을 허용하지 않도록
net.ipv4.conf.default.accept_source_route = 0
EOF

sysctl -p /etc/sysctl.conf
```

## check service

```bash
root@AJTV005 [/etc/haproxy]systemctl restart haproxy
root@AJTV005 [/etc/haproxy]systemctl status haproxy
● haproxy.service - HAProxy Load Balancer
   Loaded: loaded (/usr/lib/systemd/system/haproxy.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2020-12-04 15:25:48 KST; 4s ago
 Main PID: 6612 (haproxy-systemd)
   CGroup: /system.slice/haproxy.service
           ├─6612 /usr/sbin/haproxy-systemd-wrapper -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid
           ├─6613 /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -Ds
           └─6614 /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -Ds

Dec 04 15:25:48 AJTV005 systemd[1]: Started HAProxy Load Balancer.
Dec 04 15:25:48 AJTV005 haproxy-systemd-wrapper[6612]: haproxy-systemd-wrapper: executing /usr/sbin/haproxy -f... -Ds
Hint: Some lines were ellipsized, use -l to show in full.
root@AJTV005 [/etc/haproxy]systemctl enable haproxy
Created symlink from /etc/systemd/system/multi-user.target.wants/haproxy.service to /usr/lib/systemd/system/haproxy.service.
root@AJTV005 [/etc/haproxy]
```