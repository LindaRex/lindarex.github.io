---
title: "우분투(Ubuntu) 환경에 패키지(Package)로 HAProxy 설치하기"
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


HAProxy(이하 haproxy)는 TCP 및 HTTP 기반의 애플리케이션(application)을 위한 로드 밸런싱(load balancing) 및 프락시(proxy)를 제공하는 GNU General Public License Version 2 라이선스(license)가 적용된 오픈소스(open source) 소프트웨어입니다.
<br /><br />
haproxy는 C 언어로 개발되었으며, 네트워크(network) 스위치(switch)에서 제공하는 L4, L7 기능을 제공합니다.
<br /><br />
haproxy는 reverse proxy 형태로 동작하며, Keepalived를 사용하여 haproxy를 이중화하여 HA(high availability) 구성을 합니다.
<br /><br />
이 포스트에서는 우분투(ubuntu) 환경에 package로 haproxy를 설치하는 방법을 소개합니다.


> proxy에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/프록시_서버](https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9D%EC%8B%9C_%EC%84%9C%EB%B2%84){: target="\_blank"} 페이지를 참고하시기 바랍니다.

> L4, L7은 각각 OSI 모형(Open Systems Interconnection Reference Model, OSI 7 계층)의 전송 계층(Transport layer), 응용 계층(Application layer)을 의미합니다. 자세한 정보는 [https://ko.wikipedia.org/wiki/OSI_모형](https://ko.wikipedia.org/wiki/OSI_%EB%AA%A8%ED%98%95){: target="\_blank"} 페이지를 참고하시기 바랍니다.

> reverse proxy에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/리버스_프록시](https://ko.wikipedia.org/wiki/%EB%A6%AC%EB%B2%84%EC%8A%A4_%ED%94%84%EB%A1%9D%EC%8B%9C){: target="\_blank"} 페이지를 참고하시기 바랍니다.

> Keepalived에 대한 자세한 정보는 [https://www.keepalived.org/](https://www.keepalived.org/){: target="\_blank"} 페이지를 참고하시기 바랍니다.

> high availability에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/고가용성](https://ko.wikipedia.org/wiki/%EA%B3%A0%EA%B0%80%EC%9A%A9%EC%84%B1){: target="\_blank"} 페이지를 참고하시기 바랍니다.


## 선행조건(PREREQUISITE)
- ubuntu 환경이 필요합니다.
- 방화벽 설정이 필요합니다.

> ubuntu 설치 방법은 [우분투(Ubuntu) 서버(Server) 18.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1804-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

> 방화벽 설정 방법은 [우분투(Ubuntu) 환경에 방화벽(Firewalld) 설치 및 설정하기](https://lindarex.github.io/ubuntu/ubuntu-firewalld-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- AWS EC2 instance
    + Ubuntu Server 18.04 LTS (HVM), EBS General Purpose (SSD) Volume Type (64비트 Arm)
    + m5.large (ECU, 2 vCPUs, 3.1 GHz, 8 GiB 메모리, EBS 전용)
- Ubuntu 18.04.5 LTS (GNU/Linux 5.4.0-1043-aws x86_64)
- haproxy 1.8.8


## 요약(SUMMARY)
1. apt install 명령어로 haproxy 설치
2. (선택사항) 특정 버전 haproxy 설치
3. haproxy 설정 확인
4. systemctl 명령어로 haproxy 관리


## 내용(CONTENTS)
### 1. apt install 명령어로 haproxy 설치
#### 1.1. package index 업데이트
```console
# apt update
```

#### 1.2. haproxy 설치
```console
# apt install haproxy -y
```

#### 1.3. haproxy 설치 확인

> 2021년 4월 13일 기준 1.8.8 버전이 설치됩니다.

```console
# haproxy -v
HA-Proxy version 1.8.8-1ubuntu0.11 2020/06/22
Copyright 2000-2018 Willy Tarreau <willy@haproxy.org>
```


### 2. (선택사항) 특정 버전 haproxy 설치

> 2.2 LTS (Long Term Support) 버전을 설치합니다.

#### 2.1. (선택사항) 권장 package 설치 중지 설정
```console
# apt install --no-install-recommends software-properties-common
```

#### 2.2. dedicated PPA 활성

> 다른 버전의 haproxy PPA 정보는 [https://haproxy.debian.net/](https://haproxy.debian.net/){: target="\_blank"}를 확인해 주시기 바랍니다.

```console
# add-apt-repository ppa:vbernat/haproxy-2.2
 HAProxy is a free, very fast and reliable solution offering high availability, load balancing, and proxying for TCP and HTTP-based applications. It is particularly suited for web sites crawling under very high loads while needing persistence or Layer7 processing. Supporting tens of thousands of connections is clearly realistic with todays hardware. Its mode of operation makes its integration into existing architectures very easy and riskless, while still offering the possibility not to expose fragile web servers to the Net.

This PPA contains packages for HAProxy 2.2.
 More info: https://launchpad.net/~vbernat/+archive/ubuntu/haproxy-2.2
Press [ENTER] to continue or Ctrl-c to cancel adding it.

Hit:1 http://ap-northeast-2.ec2.archive.ubuntu.com/ubuntu bionic InRelease
Hit:2 http://ap-northeast-2.ec2.archive.ubuntu.com/ubuntu bionic-updates InRelease
Hit:3 http://ap-northeast-2.ec2.archive.ubuntu.com/ubuntu bionic-backports InRelease
Get:4 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]
Get:5 http://ppa.launchpad.net/vbernat/haproxy-2.2/ubuntu bionic InRelease [20.8 kB]
Get:6 http://ppa.launchpad.net/vbernat/haproxy-2.2/ubuntu bionic/main amd64 Packages [992 B]
Get:7 http://ppa.launchpad.net/vbernat/haproxy-2.2/ubuntu bionic/main Translation-en [704 B]
Fetched 111 kB in 2s (67.2 kB/s)
Reading package lists... Done
```

#### 2.3. haproxy 설치
```console
# apt install haproxy=2.2.\* -y
```

#### 2.4. haproxy 설치 확인
```console
# haproxy -v
HA-Proxy version 2.2.13-1ppa1~bionic 2021/04/02 - https://haproxy.org/
Status: long-term supported branch - will stop receiving fixes around Q2 2025.
Known bugs: http://www.haproxy.org/bugs/bugs-2.2.13.html
Running on: Linux 5.4.0-1043-aws #45~18.04.1-Ubuntu SMP Fri Apr 9 23:32:25 UTC 2021 x86_64
```


### 3. haproxy 설정 확인

- 설정 파일의 기본 위치는 아래와 같으며, 다음 포스트에서 haproxy 설정 방법을 소개하겠습니다.

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


### 4. systemctl 명령어로 haproxy 관리
#### 4.1. haproxy 설정 반영
```console
$ sudo systemctl daemon-reload
```

#### 4.2. haproxy 시작
```console
$ sudo systemctl start haproxy
```

#### 4.3. haproxy 중지
```console
$ sudo systemctl stop haproxy
```

#### 4.4. haproxy 재시작
```console
$ sudo systemctl restart haproxy
```

#### 4.5. haproxy 설정 재적용
```console
$ sudo systemctl reload haproxy
```

#### 4.6. haproxy 상태 조회
```console
$ sudo systemctl status haproxy
```

#### 4.7. haproxy 활성화(부팅 시 자동 시작)
```console
$ sudo systemctl enable haproxy
```

#### 4.8. haproxy 비활성화
```console
$ sudo systemctl disable haproxy
```

#### 4.9. haproxy 및 관련 프로세스 모두 중지
```console
$ sudo systemctl kill haproxy
```


## 마무리(CONCLUSION)
ubuntu 환경에 package로 haproxy 설치를 완료했습니다.
<br />
다음 포스트에서 haproxy 설정 방법을 소개하겠습니다.


## 참고(REFERENCES)
- [https://www.haproxy.org/](https://www.haproxy.org/){: target="\_blank"}





```
# apt update
# apt install haproxy -y
```



```
You need to enable a dedicated PPA with the following command:

# apt-get install --no-install-recommends software-properties-common
# add-apt-repository ppa:vbernat/haproxy-2.2
Then, use the following command:

# apt-get install haproxy=2.2.\*
```

```
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


```



