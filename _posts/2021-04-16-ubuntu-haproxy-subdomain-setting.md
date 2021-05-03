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

frontend http-rex
        bind :::80 v4v6   # bind *:80
        option        http-server-close
 
        acl domain_net_scm hdr(host) -i scm.rex-domain.net
        acl domain_net_ci hdr(host) -i ci.rex-domain.net
        acl domain_net_sonar hdr(host) -i sonar.rex-domain.net

        use_backend domain_net_app_scm        if domain_net_scm
        use_backend domain_net_app_ci        if domain_net_ci
        use_backend domain_net_app_sonar        if domain_net_sonar

backend domain_net_app_scm
        balance roundrobin
        server host1 10.0.06.6:8801
 
backend domain_net_app_ci
        balance roundrobin
        server host1 10.0.06.6:8802
 
backend domain_net_app_sonar
        balance roundrobin
        server host1 10.0.06.6:8803
```

#### 3.1. global 섹션(section)

- global section 설정은 [우분투(Ubuntu) 환경에 HAProxy 설정하기](https://lindarex.github.io/haproxy/ubuntu-haproxy-setting/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

#### 3.2. defaults section
- 아래 항목 외의 defaults section 설정은 [우분투(Ubuntu) 환경에 HAProxy 설정하기](https://lindarex.github.io/haproxy/ubuntu-haproxy-setting/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

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

##### 3.2.1. 'option http-server-close'
```
option http-server-close
```

- 클라이언트(client)에서 HTTP 연결(connection) 유지(keep-alive) 및 파이프 라이닝 기능을 유지하면서 서버(server)에서 HTTP connection을 닫을 수 있는 설정입니다.

- 이 설정은 client에서 가장 낮은 대기 시간(latency)을 제공하고 server에서 가장 빠른 세션(session) 재사용을 제공하여 리소스(resource)를 절약합니다.

- "option" 앞에 "no" 키워드 추가로 설정을 비활성화 할 수 있습니다.

> haproxy는 기본적으로 keep-alive 모드(mode)로 작동합니다.

##### 3.2.2. 'option forwardfor'
```
option forwardfor       except 127.0.0.0/8
```

- server로 전송된 요청(request)에 "X-Forwarded-For" 헤더(header) 삽입에 대한 설정입니다.

- "except" 키워드(keyword)를 적용하여 주소(address) 또는 네트워크(network)를 header에 추가하지 않을 수 있습니다.

> haproxy는 리버스(reverse) 프록시(proxy) mode로 작동하므로, haproxy server는 해당(server 자신) IP address를 client address로 간주합니다.

##### 3.2.3. 'option redispatch'
```
option                  redispatch
```

- connection 실패 시, session 재전송(redistribution)에 대한 설정입니다.

- "option" 앞에 "no" 키워드 추가로 설정을 비활성화 할 수 있습니다.

- frontend section에서는 사용할 수 없습니다.

##### 3.2.4. 'retries'
```
retries                 3
```

- connection 실패 후, server에서 수행 할 재시도 횟수 설정입니다.

- 이 설정은 전체 request가 아닌 connection 시도 횟수에 적용됩니다.

- frontend section에서는 사용할 수 없습니다.

##### 3.2.5. 'timeout http-request'
```
timeout http-request    10s
```

- 전체 HTTP request를 대기할 최대 허용 시간 설정입니다.

- 이 설정을 활성화하면, client 유형(type)에 관계없이 request가 제 시간에 완료되지 않으면 request는 중단됩니다.

- 이 설정에서 제한 시간이 만료(expire)되면, client에게 HTTP 408 응답(response)을 전송하고 connection을 닫습니다.

##### 3.2.6. 'timeout queue'
```
timeout queue           1m
```

- queue에서 보류(pending) 중인 request의 제한 시간 설정입니다.

- 이 설정을 지정하지 않으면, "timeout connect" 설정과 같은 값을 적용합니다.

- frontend section에서는 사용할 수 없습니다.

##### 3.2.7. 'timeout http-keep-alive'
```
timeout http-keep-alive 10s
```

- 새로운 HTTP request를 대기할 최대 허용 시간 설정입니다.

- "keep-alive"의 새 request를 대기 시간은 "timeout http-request"로 설정합니다.

##### 3.2.8. 'timeout check'
```
timeout check           10s
```

- "timeout check" 설정은 "timeout connect" 설정의 최대 허용 연결 시간을 사용한 후, "timeout check" 설정을 추가로 최대 허용 읽기 시간으로 적용합니다.

- 이 설정을 지정하지 않으면, "timeout connect" 설정으로 적용합니다.

- frontend section에서는 사용할 수 없습니다.

##### 3.2.9. 'maxconn'
```
maxconn                 3000
```

- frontend의 최대 동시 connection 수를 설정합니다.

- 기본값은 2000이며, global maxconn을 초과하지 않아야 합니다.

- backend section에서는 사용할 수 없습니다.


##### 3.2.10. 'timeout'
```
timeout connect         10s # 5000
timeout client          1m  # 50000
timeout server          1m  # 50000
```

- 위 설정은 [우분투(Ubuntu) 환경에 HAProxy 설정하기](https://lindarex.github.io/haproxy/ubuntu-haproxy-setting/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

#### 3.3. frontend section
##### 3.3.1. 'frontend'
```
frontend http-rex
```

- "frontend section"은 client connection에 대한 IP address와 포트(port)를 설정합니다.

- 필요에 따라 "frontend section"을 추가 할 수 있으며, "frontend" 키워드 뒤에 이름을 설정하여 구분할 수 있습니다.

> 모든 proxy section 이름은 대소문자, 숫자, '\-'(대시), '\_'(밑줄), '.'(점), ':'(콜론)으로 구성되어야 합니다.

##### 3.3.2. 'bind'
```
bind :::80 v4v6   # bind *:80
```

- ":::80"은 frontend가 수신할 address 설정이며, 선택 사항입니다.

- address 값은 호스트 이름, IPv4 주소, IPv6 주소, '\*'로 설정할 수 있으며, 설정하지 않으면 IPv4 address가 수신됩니다.

- "v4v6" 설정은 기본 address 사용 시, IPv4 및 IPv6 모두 socket에 바인딩(bind)하며, 리눅스(linux) 커널(kernel) 2.4.21 이상에서 지원합니다.

- defaults section과 backend section에서는 사용할 수 없습니다.



// todo
##### 3.3.1. ''
```
option        http-server-close
```

-


##### 3.3.1. ''
```
acl domain_net_scm hdr(host) -i scm.rex-domain.net
acl domain_net_ci hdr(host) -i ci.rex-domain.net
acl domain_net_sonar hdr(host) -i sonar.rex-domain.net
```

-


##### 3.3.1. ''
```
use_backend domain_net_app_scm        if domain_net_scm
use_backend domain_net_app_ci        if domain_net_ci
use_backend domain_net_app_sonar        if domain_net_sonar
```

-


##### 3.3.1. ''
```
backend domain_net_app_scm
        balance roundrobin
        server host1 10.0.06.6:8801
```
 
```
backend domain_net_app_ci
        balance roundrobin
        server host1 10.0.06.6:8802
```
 
```
backend domain_net_app_sonar
        balance roundrobin
        server host1 10.0.06.6:8803
```





## 마무리(CONCLUSION)
ubuntu 환경에 haproxy 설정을 완료했습니다.
<br /><br />
haproxy 설정에 대한 더 자세한 내용은 아래 참고 페이지를 확인해 주시기 바랍니다.
<br /><br />
다음 포스트에서는 haproxy를 활용한 서브도메인(subdomain) 설정 방법을 소개하겠습니다.


## 참고(REFERENCES)
- [https://cbonte.github.io/haproxy-dconv/1.8/configuration.html](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html){: target="\_blank"}

