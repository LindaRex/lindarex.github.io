---
title: "우분투(Ubuntu) 환경에 HAProxy 설정하기"
categories: 
  - haproxy
tags: 
  - "haproxy"
  - "load balancer"
  - "proxy"
  - "reverse proxy"
  - "high availability"
  - "open source software"
  - "open source"
  - oss
  - ubuntu
---


HAProxy(이하 haproxy)는 로드 밸런싱(load balancing) 및 프락시(proxy) 기능을 제공하는 GPL 2 라이선스(license)가 적용된 오픈소스(open source) 소프트웨어입니다.
<br /><br />
이 포스트에서는 우분투(ubuntu) 환경에 haproxy를 설정하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- ubuntu 환경에 haproxy가 설치되어 있어야 합니다.
- 방화벽 설정이 필요합니다.

> haproxy 설치 방법은 [우분투(Ubuntu) 환경에 패키지(Package)로 HAProxy 설치하기](https://lindarex.github.io/haproxy/ubuntu-haproxy-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

> 방화벽 설정 방법은 [우분투(Ubuntu) 환경에 방화벽(Firewalld) 설치 및 설정하기](https://lindarex.github.io/ubuntu/ubuntu-firewalld-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- AWS EC2 instance
    + Ubuntu Server 18.04 LTS (HVM), EBS General Purpose (SSD) Volume Type (64비트 Arm)
    + m5.large (ECU, 2 vCPUs, 3.1 GHz, 8 GiB 메모리, EBS 전용)
- Ubuntu 18.04.5 LTS (GNU/Linux 5.4.0-1043-aws x86_64)
- haproxy 1.8.8


## 요약(SUMMARY)
1. haproxy 설정 조회
1. apt install 명령어로 haproxy 설치
2. (선택사항) 특정 버전 haproxy 설치
4. systemctl 명령어로 haproxy 관리


## 내용(CONTENTS)
### 1. haproxy 설정 조회
#### 1.1. (선택사항) haproxy 설치 확인
```console
# haproxy -v
HA-Proxy version 1.8.8-1ubuntu0.11 2020/06/22
Copyright 2000-2018 Willy Tarreau <willy@haproxy.org>
```

#### 1.2. haproxy 설정 확인

```console
# cat /etc/haproxy/haproxy.cfg
```

```shell
global
    log /dev/log    local0
    log /dev/log    local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

    # Default SSL material locations
    ca-base /etc/ssl/certs
    crt-base /etc/ssl/private

    # Default ciphers to use on SSL-enabled listening sockets.
    # For more information, see ciphers(1SSL). This list is from:
    #  https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/
    # An alternative list with additional directives can be obtained from
    #  https://mozilla.github.io/server-side-tls/ssl-config-generator/?server=haproxy
    ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:RSA+AESGCM:RSA+AES:!aNULL:!MD5:!DSS
    ssl-default-bind-options no-sslv3

defaults
    log global
    mode    http
    option  httplog
    option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
    errorfile 400 /etc/haproxy/errors/400.http
    errorfile 403 /etc/haproxy/errors/403.http
    errorfile 408 /etc/haproxy/errors/408.http
    errorfile 500 /etc/haproxy/errors/500.http
    errorfile 502 /etc/haproxy/errors/502.http
    errorfile 503 /etc/haproxy/errors/503.http
    errorfile 504 /etc/haproxy/errors/504.http
```

- 위 설정을 설명합니다.

    ```
    'directory "/var/cache/bind";'
    // 기본값은 '/var/cache/bind'입니다.
    // server의 작업 디렉터리(directory)를 정의하며 절대 경로(path)입니다.
    ```

    ```
    'global'
    // 설명
    ```

    ```
    'log /dev/log    local0'
    // 설명
    ```

    ```
    'log /dev/log    local1 notice'
    // 설명
    ```

    ```
    'chroot /var/lib/haproxy'
    // 설명
    ```

    ```
    'stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners'
    // 설명
    ```

    ```
    'stats timeout 30s'
    // 설명
    ```

    ```
    'user haproxy'
    // 설명
    ```

    ```
    'group haproxy'
    // 설명
    ```

    ```
    'daemon'
    // 설명
    ```

    ```
    'ca-base /etc/ssl/certs'
    // 설명
    ```

    ```
    'crt-base /etc/ssl/private'
    // 설명
    ```

    ```
    'ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:RSA+AESGCM:RSA+AES:!aNULL:!MD5:!DSS'
    // 설명
    ```

    ```
    'ssl-default-bind-options no-sslv3'
    // 설명
    ```

    ```
    'defaults'
    // 설명
    ```

    ```
    'log global'
    // 설명
    ```

    ```
    'mode    http'
    // 설명
    ```

    ```
    'option  httplog'
    // 설명
    ```

    ```
    'option  dontlognull'
    // 설명
    ```

    ```
    'timeout connect 5000'
    // 설명
    ```

    ```
    'timeout client  50000'
    // 설명
    ```

    ```
    'timeout server  50000'
    // 설명
    ```

    ```
    'errorfile 400 /etc/haproxy/errors/400.http'
    // 설명
    ```

    ```
    'errorfile 403 /etc/haproxy/errors/403.http'
    // 설명
    ```

    ```
    'errorfile 408 /etc/haproxy/errors/408.http'
    // 설명
    ```

    ```
    'errorfile 500 /etc/haproxy/errors/500.http'
    // 설명
    ```

    ```
    'errorfile 502 /etc/haproxy/errors/502.http'
    // 설명
    ```

    ```
    'errorfile 503 /etc/haproxy/errors/503.http'
    // 설명
    ```

    ```
    'errorfile 504 /etc/haproxy/errors/504.http'
    // 설명
    ```



## 마무리(CONCLUSION)
ubuntu 환경에 haproxy 설정을 완료했습니다.
<br /><br />
haproxy 설정에 대한 더 자세한 내용은 아래 참고 페이지를 확인해 주시기 바랍니다.
<br /><br />
다음 포스트에서는 haproxy를 활용한 서브도메인(subdomain) 설정 방법을 소개하겠습니다.


## 참고(REFERENCES)
- [https://www.haproxy.org/](https://www.haproxy.org/){: target="\_blank"}
