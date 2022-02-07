---
title: \[Nginx\]\(Config\)Customizing 301 pages to hide nginx information
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
curl 명령어로 사이트를 조회했을때 301 code page와 Nginx 정보를 같이 반환한다.  
외부 사용자가 WEB 정보를 알 수 없도록 Nginx 정보를 변경한다.  

# 소스 변경 후 컴파일

### 아래 파일에서 nginx → [원하는 내용] 으로 변경 후 컴파일

소스 수정

```bash
/nginx-1.20.2/src/http/ngx_http_special_response.c
static u_char ngx_http_error_tail[] =
"<hr><center>nginx</center>" CRLF
"</body>" CRLF
"</html>" CRLF
```

컴파일

```bash
## configure
cd nginx-1.20.2 && \
./configure \
--prefix=/etc/nginx \
--with-http_ssl_module \
--with-http_realip_module \
--with-http_sub_module \
--with-http_gzip_static_module \
--with-http_stub_status_module \
--with-http_v2_module \

## install
make && \
make install

```

# Docker 사용하는 경우

Dockerfile에서 아래와 같이 작성하여 custom image를 만든 후 사용한다.

```bash
## edit src for 301 code edit
RUN sed -i 's,<center>nginx,<center>JWS,g' /nginx-1.20.2/src/http/ngx_http_special_response.c

## configure, make, make install
RUN cd nginx-1.20.2 && \
./configure \
--prefix=/etc/nginx \
--with-http_ssl_module \
--with-http_realip_module \
--with-http_sub_module \
--with-http_gzip_static_module \
--with-http_stub_status_module \
--with-http_v2_module \
--add-module=../headers-more-nginx-module-0.33

RUN cd nginx-1.20.2 && \
        make && \
        make install
```

# 결과
적용 전
<center><img src="/assets/images/posts/nginx/config/301-custom-page_1.png" width="150%" height="150%" ></center>

적용 후
<center><img src="/assets/images/posts/nginx/config/301-custom-page_2.png" width="150%" height="150%" ></center>