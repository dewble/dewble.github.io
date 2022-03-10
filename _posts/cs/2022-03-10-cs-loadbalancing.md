---
title: \[CS\]\(Network\)로드밸런스싱 알고리즘과 종류
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- CS
- Concept
- Load balancing
categories:
- CS
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
- 로드 밸런싱 알고리즘
    - HASH
        - 새로운 연결 시 각 클라이언트에 hashingkey(IP+PORT/IP) 를 통해 해당 서버에 배정하는 방식, 연결 지속
    - RR
        - 서버에 들어온 요청을 순서대로 돌아가며 배정
    - Weighted RR
        - 서버의 처리 능력을 고려하여 가중치 부여 RR
    - Least connection
        - 서버들 중 가장 적게 연결된 서버에 배정
    - Weighted least-connection
- 로드 밸런싱 종류
    - L2(Data Link Layer)
        - MAC 주소를 바탕으로 LB
        - 브릿지, 허브
    - L3(Network Layer)
        - IP 주소를 바탕으로  LB
        - Router, ICMP, IP
    - L4(Transport Layer)
        - 전송계층(IP,PORT) Level에서 LB 진행
        - TCP, UDP
    - L7(application Layer)
        - 사용자의 Request Level에서 LB 진행
        - HTTP, HTTPS, FTP, SMTP