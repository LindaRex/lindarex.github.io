---
title: "우분투(Ubuntu) 환경에 HAProxy로 서브도메인(subdomain) 설정하기"
categories: 
  - haproxy
tags: 
  - "haproxy"
  - "load balancer"
  - "proxy"
  - "reverse proxy"
  - "high availability"
  - "subdomain"
  - "open source software"
  - "open source"
  - oss
  - ubuntu
---


이 포스트에서는 우분투(ubuntu) 환경에 HAProxy(haproxy)로 서브도메인(subdomain)을 설정하는 방법을 소개합니다.


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
1. (선택사항) haproxy 설치 확인
2. (선택사항) haproxy 설정 확인
3. haproxy subdomain 설정
4. haproxy subdomain 설정 확인


## 내용(CONTENTS)
### 1. (선택사항) haproxy 설치 확인
```console
# haproxy -v
HA-Proxy version 1.8.8-1ubuntu0.11 2020/06/22
Copyright 2000-2018 Willy Tarreau <willy@haproxy.org>
```

### 2. (선택사항) haproxy 설정 확인

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

### 3. haproxy subdomain 설정

```console
# vi /etc/haproxy/haproxy.cfg
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

#### 3.1. defaults 섹션(section)
```
# ADDED
option http-server-close
option forwardfor       except 127.0.0.0/8
option                  redispatch
retries                 3
timeout http-request    10s
timeout queue           1m
timeout http-keep-alive 10s
timeout check           10s
maxconn                 3000

# MODIFIED
timeout connect         10s # 5000
timeout client          1m  # 50000
timeout server          1m  # 50000
```

##### 3.1.1.
```
option http-server-close
```

- 설명

##### 3.1.1.
```
option forwardfor       except 127.0.0.0/8
```

- 설명

##### 3.1.1.
```
option                  redispatch
```

- 설명

##### 3.1.1.
```
retries                 3
```

- 설명

##### 3.1.1.
```
timeout http-request    10s
```

- 설명

##### 3.1.1.
```
timeout queue           1m
```

- 설명

##### 3.1.1.
```
timeout http-keep-alive 10s
```

- 설명

##### 3.1.1.
```
timeout check           10s
```

- 설명

##### 3.1.1.
```
maxconn                 3000
```

- 설명

##### 3.1.1.
```
timeout connect         10s # 5000
```

- 설명

##### 3.1.1.
```
timeout client          1m  # 50000
```

- 설명

##### 3.1.1.
```
timeout server          1m  # 50000
```

- 설명


- global 섹션(section)의 설정은 프로세스 전체에 적용되며, 운영체제(OS)에 따라 다를 수 있습니다.

- haproxy는 global section 외에 defaults, listen, frontend, backend 등의 proxy section으로 구성됩니다.




## 마무리(CONCLUSION)
ubuntu 환경에 haproxy 설정을 완료했습니다.
<br /><br />
haproxy 설정에 대한 더 자세한 내용은 아래 참고 페이지를 확인해 주시기 바랍니다.
<br /><br />
다음 포스트에서는 haproxy를 활용한 서브도메인(subdomain) 설정 방법을 소개하겠습니다.


## 참고(REFERENCES)
- [https://cbonte.github.io/haproxy-dconv/1.8/configuration.html](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html){: target="\_blank"}


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



