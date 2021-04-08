---
title: "우분투(Ubuntu) 환경에 패키지(Package)로 HAProxy 설치하기"
categories: 
  - haproxy
tags: 
  - "haproxy"
  - "load balancer"
  - "open source software"
  - "open source"
  - oss
  - ubuntu
---




```
# apt update
# apt install haproxy -y

root@withrex-inception:/home/rex# haproxy -v
HA-Proxy version 1.8.8-1ubuntu0.11 2020/06/22
Copyright 2000-2018 Willy Tarreau <willy@haproxy.org>


root@withrex-inception:/home/rex# vi /etc/haproxy/haproxy.cfg
---
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
        log     global
        mode    http
        option  httplog
        option  dontlognull

        # ADDED
        option http-server-close
        option forwardfor       except 127.0.0.0/8
        option                  redispatch
        retries                 3
        timeout http-request    10s
        timeout queue           1m
        timeout connect         10s # 5000
        timeout client          1m  # 50000
        timeout server          1m  # 50000
        timeout http-keep-alive 10s
        timeout check           10s
        maxconn                 3000

        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

frontend http
        bind :::80 v4v6   # bind *:80
        option        http-server-close
 
        # acl domain_com hdr(host) -i withrex.com
        # acl domain_net hdr(host) -i withrex.net
        # acl domain_fun hdr(host) -i withrex.fun
        acl domain_space hdr(host) -i withrex.space
 
        acl domain_com_scm hdr(host) -i scm.withrex.com
        acl domain_com_ci hdr(host) -i ci.withrex.com
        acl domain_com_sonar hdr(host) -i sonar.withrex.com

        acl domain_net_scm hdr(host) -i scm.withrex.net
        acl domain_net_ci hdr(host) -i ci.withrex.net
        acl domain_net_sonar hdr(host) -i sonar.withrex.net

        acl domain_fun_scm hdr(host) -i scm.withrex.fun
        acl domain_fun_ci hdr(host) -i ci.withrex.fun
        acl domain_fun_sonar hdr(host) -i sonar.withrex.fun
 
        # acl is_root path -i /
        # acl is_domain hdr(host) -i withrex.com

        # use_backend domain_com_app        if domain_com
        # use_backend domain_net_app        if domain_net
        # use_backend domain_fun_app        if domain_fun

        use_backend domain_com_app_scm        if domain_com_scm
        use_backend domain_com_app_ci        if domain_com_ci
        use_backend domain_com_app_sonar        if domain_com_sonar

        use_backend domain_net_app_scm        if domain_net_scm
        use_backend domain_net_app_ci        if domain_net_ci
        use_backend domain_net_app_sonar        if domain_net_sonar

        use_backend domain_fun_app_scm        if domain_fun_scm
        use_backend domain_fun_app_ci        if domain_fun_ci
        use_backend domain_fun_app_sonar        if domain_fun_sonar

        http-request redirect code 301 location https://lindarex.synology.me:5111 if domain_space
 
# backend domain_com_app
#         balance roundrobin
#         server host1 141.164.61.7:8801
#  
# backend domain_net_app
#         balance roundrobin
#         server host1 141.164.61.7:8802
#  
# backend domain_fun_app
#         balance roundrobin
#         server host1 141.164.61.7:8803

# COM
backend domain_com_app_scm
        balance roundrobin
        server host1 141.164.61.7:8801
 
backend domain_com_app_ci
        balance roundrobin
        server host1 141.164.61.7:8802
 
backend domain_com_app_sonar
        balance roundrobin
        server host1 141.164.61.7:8803

# NET
backend domain_net_app_scm
        balance roundrobin
        server host1 141.164.61.7:8801
 
backend domain_net_app_ci
        balance roundrobin
        server host1 141.164.61.7:8802
 
backend domain_net_app_sonar
        balance roundrobin
        server host1 141.164.61.7:8803

# FUN
backend domain_fun_app_scm
        balance roundrobin
        server host1 141.164.61.7:8801
 
backend domain_fun_app_ci
        balance roundrobin
        server host1 141.164.61.7:8802
 
backend domain_fun_app_sonar
        balance roundrobin
        server host1 141.164.61.7:8803

```

```
systemctl status haproxy
```

```
withrex.fun
CNAME Record
www.withrex.fun   lindarex.synology.me
```

# TODO
- haproxy :: http -> https



HAProxy(이하 haproxy)는 개별적으로 확장 가능한 가벼운 소스 코드 관리 도구로, MIT 라이선스(license)가 적용된 오픈소스(open source) 소프트웨어입니다.
<br /><br />
haproxy는 다양한 플러그인과 Level 3 RESTful WebService를 제공하고, Git과 Mercurial, Subversion 저장소(repository)를 지원합니다.
<br /><br />
haproxy는 웹 서버(web server), 데이터베이스(database) 등의 종속성이 없고, 다양한 환경(Ubuntu, CentOS, Fedora, Windows, MacOS X, Docker, Kubernetes)에 설치할 수 있습니다.
<br /><br />
이 포스트에서는 우분투(ubuntu) 환경에 package로 haproxy를 설치하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- ubuntu 환경이 필요합니다.
- 방화벽 설정이 필요합니다.
    + TCP 8080 포트가 개방되어 있어야 합니다.

> ubuntu 설치 방법은 [우분투(Ubuntu) 서버(Server) 16.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1604-installation/){: target="\_blank"} 또는 [우분투(Ubuntu) 서버(Server) 18.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1804-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

> 방화벽 설정 방법은 [우분투(Ubuntu) 환경에 방화벽(Firewalld) 설치 및 설정하기](https://lindarex.github.io/ubuntu/ubuntu-firewalld-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- VULTR High Frequency Cloud Compute (256 GB NVMe, 3 CPU, 8192MB Memory, 4000GB Bandwidth)
- Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-136-generic x86_64)
- haproxy 2.14.1


## 요약(SUMMARY)
1. haproxy debian packages repository 설정
2. apt install 명령어로 haproxy 설치
3. (선택사항) haproxy 설정
4. systemctl 명령어로 haproxy 관리
5. 웹브라우저로 haproxy 접속


## 내용(CONTENTS)
### 1. haproxy debian packages repository key 추가
```console
$ sudo apt-key adv --recv-keys --keyserver hkps://keys.openpgp.org 0x975922F193B07D6E
Executing: /tmp/apt-key-gpghome.mDvF5MxIa1/gpg.1.sh --recv-keys --keyserver hkps://keys.openpgp.org 0x975922F193B07D6E
gpg: key 975922F193B07D6E: "SCM Packages (signing key for packages.haproxy.org) <scm-team@cloudogu.com>" not changed
gpg: Total number processed: 1
gpg:              unchanged: 1
```

### 2. haproxy debian packages repository 추가
```console
$ echo 'deb [arch=all] https://packages.haproxy.org/repository/apt-v2-releases/ stable main' | sudo tee /etc/apt/sources.list.d/haproxy.list
```

### 3. apt install 명령어로 haproxy 설치
#### 3.1. package 업데이트 
```console
$ sudo apt-get update
```

#### 3.2. haproxy 설치
```console
$ sudo apt-get install scm-server -y
```

### 4. (선택사항) haproxy 설정

- haproxy UI의 포트를 설정합니다.

```console
$ sudo vi /etc/default/scm-server
```

```shell
----------------------------------------------------------------------------------------------------
PORT=8080
----------------------------------------------------------------------------------------------------
```

### 5. systemctl 명령어로 haproxy 관리
#### 5.1. haproxy 설정 반영
```console
$ sudo systemctl daemon-reload
```

#### 5.2. haproxy 시작
```console
$ sudo systemctl start scm-server
```

#### 5.3. haproxy 중지
```console
$ sudo systemctl stop scm-server
```

#### 5.4. haproxy 재시작
```console
$ sudo systemctl restart scm-server
```

#### 5.5. haproxy 설정 재적용
```console
$ sudo systemctl reload scm-server
```

#### 5.6. haproxy 상태 조회
```console
$ sudo systemctl status scm-server
```

#### 5.7. haproxy 활성화(부팅 시 자동 시작)
```console
$ sudo systemctl enable scm-server
```

#### 5.8. haproxy 비활성화
```console
$ sudo systemctl disable scm-server
```

#### 5.9. haproxy 및 관련 프로세스 모두 중지
```console
$ sudo systemctl kill scm-server
```

### 6. 웹브라우저로 haproxy 접속
- http://[MY-IP]:8080/scm

> haproxy의 기본 계정은 'scmadmin'이며, 비밀번호는 'scmadmin' 입니다.


## 마무리(CONCLUSION)
ubuntu 환경에 package로 haproxy 설치를 완료했습니다.
<br /><br />
haproxy의 자세한 내용은 아래 페이지를 확인해 주시기 바랍니다.


## 참고(REFERENCES)
- [https://www.haproxy.org/](https://www.haproxy.org/){: target="\_blank"}
