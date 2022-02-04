---
title: \[Nginx\]\(Config\)응답 헤더에서 Nginx 정보 제거 - remove response header server
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
응답헤더에 표시되는 nginx 정보 제거

# 버전 정보만 숨기기

nginx 1.20.1 → nginx

<center><img src="/assets/images/posts/nginx/config/response_header_1.png" width="150%" height="150%" ></center>
 

## nginx.conf 내용 추가

```bash
http {
server_tokens off;
}
```

# 버전 정보 변경 - 모듈 추가 필요

nginx 1.20.1 → JWS (원하는 내용으로 작성)

### 사용 모듈

[headers-more-nginx-module](https://github.com/openresty/headers-more-nginx-module)

## 소스컴파일 (docker)

```bash
## install headers-more-nginx-module
RUN wget https://github.com/openresty/headers-more-nginx-module/archive/refs/tags/v0.33.tar.gz
RUN tar zxvf v0.33.tar.gz

RUN cd nginx-1.20.1 && \
./configure \
--prefix=/etc/nginx \
--with-http_ssl_module \
--with-http_realip_module \
--with-http_sub_module \
--with-http_gzip_static_module \
--with-http_stub_status_module \
--with-http_v2_module \
--add-module=../headers-more-nginx-module-0.33

RUN cd nginx-1.20.1 && \
	make && \ 
	make install
```

## nginx.conf 내용 추가

<center><img src="/assets/images/posts/nginx/config/response_header_2.png" width="150%" height="150%" ></center>


```bash
http {
more_set_headers 'Server: JWS';
}
```