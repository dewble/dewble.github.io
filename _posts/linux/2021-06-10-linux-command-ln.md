---
title: \[Linux\]\(Command\)ln - make links between files
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Linux
- Command
- ln
categories:
- Linux
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
The **ln** utility creates a new directory entry (linked file) which has the same modes as the original file.

<center><img src="/assets/images/posts/linux/command/ln/ln1.png" width="150%" height="150%"></center>

---
# hardlink
하드 링크를 생성하면 하드링크 파일이 생성되고 원본 파일과 같은 inode를 사용한다.

### 사용법

```bash
ln basefile.txt hardlink.txt
```
### inode 보는 명령어

```bash
ls -il [filename]
```

1. Create hardlink
2. Check inode  
<center><img src="/assets/images/posts/linux/command/ln/ln2.png" width="150%" height="150%"></center>  


--- 
# softlink (symbolic link)

소프트 링크를 생성하면 새로운 inode를 생성하고, 데이터는 원본 파일에 연결한다.

### 사용법

```bash
ln -s basefile.txt softlink.txt
```

1. Create softlink
2. Check inode  
<center><img src="/assets/images/posts/linux/command/ln/ln3.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/linux/command/ln/ln4.png" width="150%" height="150%"></center>
원본 파일이 변경되면 softlink로 찾을 수 없다.

<center><img src="/assets/images/posts/linux/command/ln/ln5.png" width="150%" height="150%"></center>

### option

-s : 소프트 심볼릭링크  
-f : 강제 처리 옵션 (기존에 링크가 존재할 경우)