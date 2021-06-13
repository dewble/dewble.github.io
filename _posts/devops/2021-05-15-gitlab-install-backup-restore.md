---
title: \[DevOps\]\(Gitlab\)Install / Backup / Restore gitlab
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- DevOps
- Gitlab
- CI/CD
categories:
- DevOps
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
Install, backup, restore Gitlab CI/CD  

---
# 1. Install gitlab

Gitlab version: 13.2.0

**If you want to restore gitlab on other server you have to match gitlab version**

```bash
yum install -y curl policycoreutils-python openssh-server perl

## add repo 
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash

## install gitlab
## EXTERNAL_URL is option
EXTERNAL_URL="https://gitlab.dewble.com" yum install -y gitlab-ce-13.2.0-ce.0.el7
```

The version can be checked with command below

```bash
yum --showduplicates list gitlab-ce 
```    

---
# 2. Backup

## set backup config

vim /etc/gitlab/gitlab.rb

```bash
### Backup Settings
###! Docs: https://docs.gitlab.com/omnibus/settings/backups.html

gitlab_rails['manage_backup_path'] = true
gitlab_rails['backup_path'] = "/data/gitlab/backups"
```

## Backup command

```bash
gitlab-rake gitlab:backup:create
```  

---
# 3. restore
## File transfer

Transfer file from the original to the destination

```bash
# Required files
/etc/gitlab/gitlab-secrets.json
/etc/gitlab/gitlab.rb
gitlab_backup.tar
```

## Stop process

```bash
gitlab-ctl stop unicorn
gitlab-ctl stop puma
gitlab-ctl stop sidekiq

# Verify
gitlab-ctl status
```

## Restroe gitlab

```bash
## without gitlab-tar
gitlab-backup restore BACKUP=1620209195_2021_05_05_13.2.0

```

## Restart gitlab

```bash
gitlab-ctl reconfigure
gitlab-ctl restart

## command a few seconds later after restart
# check config
gitlab-rake gitlab:check SANITIZE=true
# check secrets
gitlab-rake gitlab:doctor:secrets
```