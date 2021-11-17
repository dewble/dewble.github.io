---
title: \[Scouter\]\(Install\)Install scouter host and java agent
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Scouter
- Install
- Host Agent
- Java Agent
categories:
- Scouter
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
Host Agent: OS의 CPU, Memory, Disk등의 성능 정보 전송  
Java Agent: 실시간 서비스 성능 정보, Heap Memory, Thread 등 Java 성능 정보  

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

# 2.  Install host agent

### Scouter **Download link**

[Releases · scouter-project/scouter](https://github.com/scouter-project/scouter/releases)

## 2.1 Install host agent

```bash
wget https://github.com/scouter-project/scouter/releases/download/v2.10.2/scouter-all-2.10.2.tar.gz

root@ajtv006 [~/workspace/scouter/scouter/agent.host]tar xvf scouter-all-2.10.2.tar.gz
```

## 2.2 Edit host agent config

```bash
root@ajtv006 [~/workspace/scouter/scouter/agent.host/conf]cat scouter.conf
root@ajtv006 [~/workspace/scouter/scouter/agent.host/conf]cat scouter.conf
# Scouter Server IP Address (Default : 127.0.0.1)
net_collector_ip=127.0.0.1

# Scouter Server Port (Default : 6100)
net_collector_udp_port=6100
net_collector_tcp_port=6100

# Scouter Name(Default : tomcat1)
obj_name=myFirstTomcat1

cpu_alert_enabled=true
cpu_warning_pct=90
cpu_fatal_pct=95
cpu_check_period_ms=180000
cpu_warning_history=30
cpu_fatal_history=30
## do not send same alert in time
#cpu_alert_interval_ms=300000
disk_alert_enabled=true
disk_warning_pct=80
disk_fatal_pct=90

mem_alert_enabled=true
mem_warning_pct=80
mem_fatal_pct=90
```
전체 옵션 및 default 값은 scouter client의 Object > Configure 메뉴에서 확인이 가능하다.

## 2.3 Start host agent

```bash
root@ajtv006 [~/workspace/scouter/scouter/agent.host]./host.sh
nohup: redirecting stderr to stdout
  ____                  _
 / ___|  ___ ___  _   _| |_ ___ _ __
 \___ \ / __/   \| | | | __/ _ \ '__|
  ___) | (_| (+) | |_| | ||  __/ |
 |____/ \___\___/ \__,_|\__\___|_|
 Open Source S/W Performance Monitoring
 Scouter version 2.10.2

Configure -Dscouter.config=./conf/scouter.conf
Scouter Host Agent Version 2.10.2 2020-12-19 01:45 GMT
System JRE version : 1.8.0_275
```

---

# 3. Install java agent

## 3.1 Edit java agent config

scouter/agent.java/conf/scouter.conf

```bash
### scouter java agent configuration sample
obj_name=tomcat-1531
net_collector_ip=127.0.0.1
net_collector_udp_port=6100
net_collector_tcp_port=6100
#hook_method_patterns=sample.mybiz.*Biz.*,sample.service.*Service.*
#trace_http_client_ip_header_key=X-Forwarded-For
#profile_spring_controller_method_parameter_enabled=false
#hook_exception_class_patterns=my.exception.TypedException
#profile_fullstack_hooked_exception_enabled=true
#hook_exception_handler_method_patterns=my.AbstractAPIController.fallbackHandler,my.ApiExceptionLoggingFilter.handleNotFoundErrorResponse
#hook_exception_hanlder_exclude_class_patterns=exception.BizException
```

여러개의 WAS를 모니터링할 경우 scouter.conf를 WAS에 맞게 여러개 작성 한다.

## 3.1 Start tomcat with scouter agent

### was 기동 시키는 계정 권한 주의

catalina.bat 에서 setenv.bat 찾아 확인하고 

call 부분에 setenv.bat 없다면 새로 생성

```bash
rem Get standard environment variables
if not exist "%CATALINA_BASE%\bin\setenv.bat" goto checkSetenvHome
call "%CATALINA_BASE%\bin\setenv.bat"
```

tomcat/bin/setenv.bat

```bash
set "JAVA_OPTS=%JAVA_OPTS% -javaagent:C:\work\scouter\agent.java\scouter.agent.jar"
set "JAVA_OPTS=%JAVA_OPTS% -Dscouter.config=C:\work\scouter\agent.java\conf\scouter.conf"
```

/bin/start.bat

```bash
# start tomcat
startup.bat
```

<center><img src="/assets/images/posts/scouter/install/collector-6.png" width="150%" height="150%"></center>


## 3.2 Start Wildfly with Scouter agent

### was 기동 시키는 계정 권한 주의

Scouter 기동 시키는 두 가지 방법

### 3.2.1 Edit $JBOSS_HOME/bin/standalone.conf

```bash
JAVA_OPTS=" ${JAVA_OPTS} -javaagent:${SCOUTER_AGENT_DIR}/scouter.agent.jar"
JAVA_OPTS=" ${JAVA_OPTS} -Dscouter.config=${SCOUTER_AGENT_DIR}/conf/scouter1.conf"
JAVA_OPTS=" ${JAVA_OPTS} -Dobj_name=myFirstTomcat1"
```

```bash
JAVA_OPTS=" ${JAVA_OPTS} -javaagent:/home/asmanager/scouter/agent.java/scouter.agent.jar"
JAVA_OPTS=" ${JAVA_OPTS} -Dscouter.config=/home/asmanager/scouter/agent.java/conf/scouter.conf"
JAVA_OPTS=" ${JAVA_OPTS} -Dobj_name=wildfly-1531"
```

WAS가 여러개 뜨는 경우 -Dscouter.config 를 각각 작성하여 기동시킨다.

### 3.2.2 Edit $servername/bin/env.sh

```bash
SCOUTER_CONFIG=/home/asmanager/scouter/agent.java/conf/$NODE_NAME.conf

### scouter
JAVA_OPTS="$JAVA_OPTS -javaagent:/home/asmanager/scouter/agent.java/scouter.agent.jar"
JAVA_OPTS="$JAVA_OPTS -Dscouter.config=$SCOUTER_CONFIG"
```

## 3.3 Start Embedded tomcat with scouter agent

```bash
java -javaagent:scouter/scouter.agent.jar -Dscouter.config=scouter/scouter.conf -jar app.war -Dspring.profiles.active=%DEPLOY_MODE% -Dmaven.test.skip=true -Xms128m -Xmx2048m -XX:+UseG1GC 
```