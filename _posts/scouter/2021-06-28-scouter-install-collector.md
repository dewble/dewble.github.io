---
title: \[Scouter\]\(Install\)Install scouter collector, client
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Scouter
- Install
- Collector
categories:
- Scouter
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# 1. Install JDK 1.8 or higher
## 1.1 Install openjdk

```bash
yum install java-1.8.0-openjdk java-1.8.0-openjdk-devel -y
```

## 1.2 Find java location

```bash
## 실제 경로 찾기
root@ajtv005 [~]readlink -f /usr/bin/java
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.275.b01-0.el7_9.x86_64/jre/bin/java
```

## 1.3 Set JAVA_HOME, PATH, CLASSPATH

```bash
## vim /etc/profile
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.275.b01-0.el7_9.x86_64
PATH=$PATH:$JAVA_HOME/bin
CLASSPATH=$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar

export JAVA_HOME PATH CLASSPATH

##
root@ajtv005 [~]source /etc/profile
```

## 1.4 Java test

```bash
## vim javatest.java
public class javatest{
   public static void main(String[] args){
        System.out.println("Hello World!!");
   }
}

## 컴파일
root@ajtv005 [~/scripts]javac javatest.java
root@ajtv005 [~/scripts]java -cp . javatest
Hello World!!
```

---

# 2. Install Scouter Collector
<center><img src="/assets/images/posts/scouter/install/collector-1.png" width="150%" height="150%"></center>

### Collector, client Download link
[Releases · scouter-project/scouter](https://github.com/scouter-project/scouter/releases)

## 2.1 Install Scouter Collector

```bash
wget https://github.com/scouter-project/scouter/releases/download/v2.10.2/scouter-all-2.10.2.tar.gz
root@ajtv005 [~/workspace/scouter]tar xvf scouter-all-2.10.2.tar.gz

## 시작
root@ajtv005 [~/workspace/scouter/scouter/server]./startup.sh
nohup: redirecting stderr to stdout
  ____                  _
 / ___|  ___ ___  _   _| |_ ___ _ __
 \___ \ / __/   \| | | | __/ _ \ '__|
  ___) | (_| (+) | |_| | ||  __/ |
 |____/ \___\___/ \__,_|\__\___|_|
 Open Source S/W Performance Monitoring
 Scouter version 2.10.2
```

## 2.2 Set Config, Network Port

- UDP Receive Port : 6100 (성능 정보 수집용)
- TCP Service Port : 6100 (Agent 및 Client 통신용)

server/conf/scouter.conf

```bash
# server node name
server_id=Scouter_Collector

# Agent Control and Service Port(Default : TCP 6100)
net_tcp_listen_port=6100

# UDP Receive Port(Default : 6100)
net_udp_listen_port=6100

# DB directory(Default : ./database)
# Repository 경로
db_dir=./database

# Log directory(Default : ./logs)
log_dir=./logs
```

전체 옵션 및 default 값은 scouter client의 Collector > Configure 메뉴에서 확인이 가능하다.

---
# 3. Install client - windows

<center><img src="/assets/images/posts/scouter/install/collector-2.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/install/collector-3.png" width="50%" height="50%"></center>


## 3.1 Add collector server

<center><img src="/assets/images/posts/scouter/install/collector-4.png" width="150%" height="150%"></center>


## 3.2 set General

<center><img src="/assets/images/posts/scouter/install/collector-5.png" width="150%" height="150%"></center>
