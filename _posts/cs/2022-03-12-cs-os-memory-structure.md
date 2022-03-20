---
title: \[CS\]\(OS\)메모리 구조
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- CS
- Concept
- OS
- Memory
categories:
- CS
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
- 메모리 구조
    - 코드 영역:
        - ***실행할 프로그램의 코드가 저장***되는 영역
        - 사용자가 프로그램 실행 명령을 내리면 OS가 HDD에서 메모리로 실행 코드를 올리게 되고, CPU는 코드 영역에 저장된 명령어를 하나씩 처리하게 된다.
    - 데이터 영역:
        - 프로그램의 전역(global), 정적(static)가 저장되는 영역
        - 데이터 영역은 프로그램의 시작과 함께 할당되며, 프로그램이 종료되면 소멸한다.
    - 힙 영역:
        - ***프로그래머가 직접 관리할 수 있는 메모리영역***
        - 이 공간에 메모리를 할당하는 것을 동적할당이라고 한다. Java에서는 GC가 자동으로 해제해준다. 스택영역과 달리 낮은 주소에서 높은 주소로 메모리가 할당 된다.
    - 스택 영역:
        - 함수의 호출과 함께 할당되며 지역 변수와 매개 변수가 저장되는 영역
        - 스택 영역에 저장되는 함수의 ***호출 정보***를 스택프레임이라고 한다.
        - 스택 영역은 ***함수의 호출이 완료되면 소멸***한다.
        - 높은 주소에서 낮은 주소로 메모리가 할당 된다.

<center><img src="/assets/images/posts/cs/os/memory-structure.png" width="100%" height="100%" ></center>  