---
title: \[Linux\]\(Command\)basename - strip directory and suffix from filenames
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Linux
- Command
- SCP
categories:
- Linux
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# **Purpose**
Print NAME with any leading directory components removed.  If specified, also remove a trailing SUFFIX.

# suffix

remove a trailing suffix

option: -s

```bash
# without -s 
root@jv0540 [/data/gitlab/backups]basename /data/gitlab/backups/1620209195_2021_05_05_13.2.0_gitlab_backup.tar
1620209195_2021_05_05_13.2.0_gitlab_backup.tar

# with -s
root@jv0540 [/data/gitlab/backups]basename -s "_gitlab_backup.tar" /data/gitlab/backups/1620209195_2021_05_05_13.2.0_gitlab_backup.tar
1620209195_2021_05_05_13.2.0

root@jv0540 [/data/gitlab/backups]basename -s "_gitlab_backup.tar" /data/gitlab/backups/*gitlab_backup.tar
1620209195_2021_05_03_13.2.0
1620209195_2021_05_05_13.2.0

```

# multiple

options: -a

```bash
root@jv0540 [~/scripts]basename -a ~/scripts/*.txt
test1.txt
test2.txt
test3.txt

root@jv0540 [~/scripts]basename -a ~/scripts/test1.txt ~/scripts/test2.txt
test1.txt
test2.txt
```