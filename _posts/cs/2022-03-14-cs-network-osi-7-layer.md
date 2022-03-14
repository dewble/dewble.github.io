---
title: \[CS\]\(Network\)OSI 7 계층
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- CS
- Concept
- Network
- OSI
categories:
- CS
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
- OSI 7 Layer
    - 7 (응용계층):
        - http, ftp, telnet
        - 사용자와 직접 상호작용하는 응용 프로그램들이 포함된 계층
    - 6 (표현계층):
        - 포맷 / 세션으로 통한 연결이 확인되면 데이터 요청에 의한 데이터 표현
        - 데이터의 형식(Format)을 정의하는 계층
    - 5 (세션계층):
        - 연결지향 / 노드간 연결이 잘되어있나?
        - 컴퓨터끼리 통신을 하기 위해 세션을 만드는 계층
    - 4 (전송계층):
        - tcp, udp / 오류검출, 패킷을 분할로 보내어 정리
        - 최종 수신 프로세스로 데이터의 전송을 담당하는 계층
    - 3 (네트워크):
        - 라우터 / ip를 가지고 출발 목적에 대해 데이터 전송
        - 패킷을 목적지까지 가장 빠른길로 전송하기 위한 계층
    - 2 (데이터링크):
        - 스위치 / 물리적으로 연결된 노드에 데이터를 주고받고 간단한 오류 검출, 맥어드레스
        - 데이터의 물리적인 전송과 에러 검출, 흐름 제어를 담당
    - 1 (물리계층):
        - 선 / 노드간 데이터를 전송하기 위함
        - 데이터를 전기 신호로 바꾸어주는 계층