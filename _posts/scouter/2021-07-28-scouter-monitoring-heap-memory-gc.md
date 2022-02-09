---
title: \[Scouter\]\(Monitoring\)What are Heap Memory and Garbage Collection(GC)
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Scouter
- Monitoring
- What is
- Heap Memory
- Garbage Collection
categories:
- Scouter
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Java Heap Memory
<center><img src="/assets/images/posts/scouter/monitoring/monitoring-heap-gc-1.png" width="150%" height="150%"></center>

java 8 이상 부터는 Permanet Generation → Metaspace로 표현한다.

운영을 위해
Xms(initial heap size), Xmx(maximum heap size), PermSize(Metaspace)를 설정하여 사용한다.

---

# Garbage Collection

## Minor GC

Young Generation 이 차게되면 Minor GC가 동작한다.

1\. 새롭게 생성된 object들이 Eden 영역에 저장된다.  
2\. Eden 영역이 Full이 되면 live object의 리스트를 확인한다.  
3\. 아직 살아있는(다른곳에서 참조가 있는) live object는 HeapSpace.Survivor(To)로 옮긴다.  

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-heap-gc-2.png" width="150%" height="150%"></center>

4\. live object는 To로 이동되었으며 Eden 영역은 garbage만 남게 된다.  
5\. Eden의 garbage를 scanvenge하여 깨끗하게 만든다(minor collection). 이 작업 후에는 Eden영역이 항상 깨끗하게 비어지게 된다(Completely Empty). 그리고 To에 live object TeunuringThreshold 값은 1이된다.  

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-heap-gc-3.png" width="150%" height="150%"></center>

6\. 다른 object들이 새롭게 생성되면서 Eden 영역이 가득차게 되면 다시 Eden과 To 영역의 live object 리스트를 확인한다.  
7\. Eden과 To의 live object를 From으로 copy. Eden과 To에는 garbage만 남게 된다.  

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-heap-gc-4.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-heap-gc-5.png" width="150%" height="150%"></center>

8\. Eden과 To의 garbage를 scavenge하여 깨끗하게 만든다. 그리고 기존에 To 영역과 From 영역의 이름을 바꾼다. 그렇게 되면 To 영역(기존 From)에는 TenuringThreshold값이 1(기존 Eden), 2(기존 To)인 live object만 남게된다.  

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-heap-gc-6.png" width="150%" height="150%"></center>

9\. 다시 다른 object들이 새롭게 생성되면서 Eden 영역이 가득차게 되면 live object의 TenuringThreshold 값을 확인한다.  
10\. 이때 To 영역의 TenuringThreshold값이 2인 object들이 처음으로 promotion 된다 (New → Old)    
11\. promotion이 일어나면 live object는 Old 영역에 저장되며 TenuringThreshold 값이 3이 된다.  

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-heap-gc-7.png" width="150%" height="150%"></center>

12\. 그리고 가득찬 Eden 영역을 해소하기 위해 Eden과 To 영역의 live object를 확인하여 From으로 이동시킨다.  

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-heap-gc-8.png" width="150%" height="150%"></center>

13\. live object의 이동이 완료되면 From에는 object들의 TenuringThreshold값이 Eden에서 옮겨온 object는 1로 To에서 옮겨온 object는 2로 설정된다.  

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-heap-gc-9.png" width="150%" height="150%"></center>

14\. copy가 완료되면 Eden과 To 영역을 scavenger하여 깨끗하게 만든다. 그리고 To 영역과 From 영역의 이름을 바꾼다.  
15\. MaxTenuringThreshold 값 만큼 위와 같은 Minor GC가 수행되면서 object들이 promotion 된다.  

> [https://m.blog.naver.com/PostView.nhn?isHttpsRedirect=true&blogId=bumsukoh&logNo=110118615120](https://m.blog.naver.com/PostView.nhn?isHttpsRedirect=true&blogId=bumsukoh&logNo=110118615120)

## Major GC

1\. Minor GC가 지속적으로 수행되면 promotion이 발생하게되어 Old 영역이 가득차게 된다.  

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-heap-gc-10.png" width="150%" height="150%"></center>

2\. Old 영역이 가득차게 되면 Major GC가 발생하며 Old 영역의 Object에 대해 더 이상 사용하지 않는 Object를 삭제하고 메모리의 연속된 여유공간을 확보하기 위해 앞쪽으로 정렬하는 작업을 수행한다.  

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-heap-gc-11.png" width="150%" height="150%"></center>

3\. 위와 같이 Major GC는 Old 영역의 모든 object에 대해 참조정보를 확인하여 Mark하고 해당 object를 삭제하기 위해 Sweep작업을 수행한다. 그리고 마지막에 할당가능한 메모리 공간을 확보하기 위한 Compact 작업을 수행한다.  

> [https://m.blog.naver.com/PostView.nhn?isHttpsRedirect=true&blogId=bumsukoh&logNo=110118615120](https://m.blog.naver.com/PostView.nhn?isHttpsRedirect=true&blogId=bumsukoh&logNo=110118615120)

- object의 수가 많을 수록 GC 시간을 길어진다.
- GC Time이 길어질수록 사용자의 응답시간이 느려진다.
- GC Time이 1초가 안넘도록 노력해야 한다.

> [JVM Option 및 용어 참고](https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html)

---

# Example

## 예시 - Heap memory

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-heap-gc-12.png" width="150%" height="150%"></center>

Heap 메모리가 올라가면 GC Time과 Count가 올라가고 이때문에 CPU 또한 상승하게 된다.

## 예시 -  Perm memory

보통 하루 이틀 운영하면 호출될 class가 전부 호출되기 때문에 더 이상 상승하지 않는다.

프레임워크 설계가 잘못되면 상승할 수 있음.


<center><img src="/assets/images/posts/scouter/monitoring/monitoring-heap-gc-13.png" width="150%" height="150%"></center>