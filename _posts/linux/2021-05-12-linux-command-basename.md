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
# Purpose
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
0 0 * * * (find /box/log/*/*.log.* -mtime +7 -exec rm -rf {} \;)
30 0 * * * (find /box/log/*/*.log.* ! -name '*.gz' -mtime +1 -exec gzip {} \;)
```

### 매일 4시, 14일 지난 로그 삭제 (script 실행)

```bash
00 04 * * * /root/scheduled_job/log_delete.sh
```

log_delete.sh

```bash
#!/bin/sh

LOG=/box/java_logs

find $LOG -type f -mtime +14 |xargs rm -f
```