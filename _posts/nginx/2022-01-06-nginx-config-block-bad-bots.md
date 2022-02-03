---
title: \[Nginx\]\(Config\)웹 크롤링 차단(Block bad bots) - http_user_agent
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Nginx
- Config
categories:
- Nginx
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
부적절한 웹 크롤링을 차단  

더 많은 리스트들은 악성 봇 리스트를 검색하여 환경에 맞게 수정

### 주요 사이트 검색 로봇 User-Agent
naver: Yeti  
daum: Daum  
google: Googlebot  


# nginx.conf 에 내용 추가
목적에 맞게 http_user_agent에 내용을 추가한다.

```bash
server {
## 중략
  if ($http_user_agent ~* (^MJ12bot|^MJ12bot/v1.4.5|SemrushBot|SemrushBot-SA|DomainCrawler|MegaIndex.ru|AlphaBot) ) { return 403; }
    
  location / {
  ## 중략
  }
}
```