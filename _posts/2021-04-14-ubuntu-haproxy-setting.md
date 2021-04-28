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
  - "oss"
  - "ubuntu"
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
1. (선택사항) haproxy 설치 확인
2. haproxy 설정 확인


## 내용(CONTENTS)
### 1. (선택사항) haproxy 설치 확인
```console
# haproxy -v
HA-Proxy version 1.8.8-1ubuntu0.11 2020/06/22
Copyright 2000-2018 Willy Tarreau <willy@haproxy.org>
```

### 2. haproxy 설정 확인

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

#### 2.1. 'global'
```
'global'
```

- global 섹션(section)의 설정은 프로세스 전체에 적용되며, 운영체제(OS)에 따라 다를 수 있습니다.

- haproxy는 global section 외에 defaults, listen, frontend, backend 등의 proxy section으로 구성됩니다.

#### 2.2. 'log'
```
'log /dev/log    local0'
'log /dev/log    local1 notice'
```

- global syslog 서버(server) 설정입니다.

- 시작 및 종료에 대한 로그(log)와 "log global"로 구성된 proxy의 모든 log를 수신합니다.

- "/dev/log"는 주소(address) 설정이며, IPv4, IPv6, 유닉스(unix) 소켓(socket)의 경로(path) 등으로 설정할 수 있습니다.

- "local0", "local1"은 facility 설정이며, syslog facility로 설정합니다.

- "notice"는 level 설정이며, syslog severity level로 설정합니다.

> syslog에 대한 자세한 사항은 [https://en.wikipedia.org/wiki/Syslog](https://en.wikipedia.org/wiki/Syslog){: target="\_blank"} 페이지를 참고하시기 바랍니다.

#### 2.3. 'chroot'
```
'chroot /var/lib/haproxy'
```

- 보안 강화를 위한 chroot 설정이며, 슈퍼 유저(superuser) 권한으로 시작될 때만 작동합니다.

- "/var/lib/haproxy"는 chroot 감옥(jail) 디렉터리(directory) path 또는 샌드박스 path이며, chroot로 만들어진 격리된 공간입니다.

> chroot에 대한 자세한 사항은 [https://ko.wikipedia.org/wiki/Chroot](https://ko.wikipedia.org/wiki/Chroot){: target="\_blank"} 페이지를 참고하시기 바랍니다.

#### 2.4. 'stats socket'
```
'stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners'
```

- socket 바인딩(bind) 설정입니다.
    + unix socket을 "path"에 bind 하거나 TCP v4/v6 address를 "address:port" 형식으로 bind 합니다.

- "/run/haproxy/admin.sock"는 "stats socket" 설정의 path입니다.

- "mode 660"는 unix socket 접근(access) 권한 설정입니다.
    + 8진 모드로 설정하며, global section의 "unix-bind"로도 설정할 수 있습니다.

- "level admin"은 socket 실행 권한 제한 설정입니다.
    + "user", "operator", "admin" 중 하나로 설정할 수 있으며, 아시다시피 "admin"은 모든 권한을 갖습니다.

- "expose-fd listeners"은 seamless reload(zero downtime reload)를 위한 설정으로, haproxy 1.8 버전부터 사용할 수 있습니다.

    + 위 설정은 리스너(listeners) FD(file descriptor)를 다른 haproxy 프로세스(process)로 전달합니다.
    + 이전 process에서 file descriptor를 통해 unix socket을 사용해서 연결(connection)을 유지하고, 이를 이전 process와 새로운 process에 전달해서 connection 유실을 방지할 수 있는 설정입니다.

#### 2.5. 'stats timeout'
```
'stats timeout 30s'
```

- socket timeout 설정입니다.
    + 기본 시간제한은 10초이며, 밀리초(millisecond) 단위 또는 "us", "ms", "s", "m", "h", "d" 단위로 설정합니다.

#### 2.6. 'user'
```
'user haproxy'
```

- 시스템(system) 사용자를 unix socket 소유자로 설정합니다.

#### 2.7. 'group'
```
'group haproxy'
```

- system 그룹을 unix socket 그룹으로 설정합니다.

#### 2.8. 'daemon'
```
'daemon'
```

- process fork를 백그라운드로 만드는 권장 설정입니다.

- "-D" 인수(argument)와 동일하며, "-db" argument로 비활성화할 수 있습니다.

> fork에 대한 자세한 사항은 [https://ko.wikipedia.org/wiki/포크_(시스템_호출)](https://ko.wikipedia.org/wiki/%ED%8F%AC%ED%81%AC_(%EC%8B%9C%EC%8A%A4%ED%85%9C_%ED%98%B8%EC%B6%9C)){: target="\_blank"} 페이지를 참고하시기 바랍니다.

#### 2.9. 'ca-base'
```
'ca-base /etc/ssl/certs'
```

- "ca-file" 또는 "crl-file"의 상대 경로(path) 사용 시, SSL CA 인증서(certificates) 및 CRLs의 기본 directory를 설정합니다.

#### 2.10. 'crt-base'
```
'crt-base /etc/ssl/private'
```

- "crtfile"의 상대 경로(path) 사용 시, SSL certificates의 기본 directory를 설정합니다.

#### 2.11. 'ssl-default-bind-ciphers'
```
'ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:RSA+AESGCM:RSA+AES:!aNULL:!MD5:!DSS'
```

- SSL/TLS cipher 설정입니다.

- TLS v1.2까지 설정할 수 있으며, TLS v1.3 구성은 "ssl-default-bind-ciphersuites" 설정이 필요합니다.

> SSL/TLS에 대한 자세한 사항은 [https://ko.wikipedia.org/wiki/전송_계층_보안](https://ko.wikipedia.org/wiki/%EC%A0%84%EC%86%A1_%EA%B3%84%EC%B8%B5_%EB%B3%B4%EC%95%88){: target="\_blank"} 페이지를 참고하시기 바랍니다.

> cipher에 대한 자세한 사항은 [https://ko.wikipedia.org/wiki/암호_(암호학)](https://ko.wikipedia.org/wiki/%EC%95%94%ED%98%B8_(%EC%95%94%ED%98%B8%ED%95%99)){: target="\_blank"} 페이지를 참고하시기 바랍니다.

> cipher suite에 대한 자세한 사항은 [https://ko.wikipedia.org/wiki/암호화_스위트](https://ko.wikipedia.org/wiki/%EC%95%94%ED%98%B8%ED%99%94_%EC%8A%A4%EC%9C%84%ED%8A%B8){: target="\_blank"} 페이지와 [https://en.wikipedia.org/wiki/Cipher_suite](https://en.wikipedia.org/wiki/Cipher_suite){: target="\_blank"} 페이지를 참고하시기 바랍니다.


#### 2.12. 'ssl-default-bind-options'
```
'ssl-default-bind-options no-sslv3'
```

- 기본 SSL 설정입니다.

- "no-sslv3" 설정은 SSLv3 사용을 비활성화합니다.

#### 2.13. 'defaults'
```
'defaults'
```

- defaults section의 설정은 모든 section에 적용됩니다.

- section 명은 선택 사항이지만, 설정하는 것을 권장합니다.

#### 2.14. 'log global'
```
'log global'
```

- global log 설정입니다.

- haproxy 인스턴스(instance) 당 하나만 설정할 수 있으며, 매개 변수(parameter)가 없습니다.

#### 2.15. 'mode'
```
'mode    http'
```

- haproxy 실행 mode 또는 프로토콜(protocol) 설정입니다.

- tcp, http, health mode로 설정할 수 있으며, 기본값은 tcp mode입니다.
    + tcp mode는 SSL, SSSH, SMTP 등에 사용합니다.
    + http mode는 RFC를 준수하지 않는 요청은 거부되며, 서버 연결(connection) 전에 분석됩니다.

#### 2.16. 'option'
```
'option  httplog'
```

- HTTP 요청(request), 세션(session) 상태 및 타이머(timer) 등의 로깅(logging)을 활성화하는 설정입니다.

- backend section에서는 사용할 수 없습니다.

```
'option  dontlognull'
```

- null connection logging 설정입니다.

- 이 설정은 데이터(data)가 전송되지 않는 connection을 기록하지 않습니다.

- "option" 앞에 "no" 키워드 추가로 설정을 비활성화 할 수 있습니다.

- backend section에서는 사용할 수 없습니다.

#### 2.17. 'timeout'
```
'timeout connect 5000'
```

- server의 연결 제한(지연) 시간 설정입니다.

- millisecond로 설정하며, frontend section에서는 사용할 수 없습니다.

```
'timeout client  50000'
```

- 클라이언트(client)의 최대 비활성 시간 설정입니다.

- millisecond로 설정하며, backend section에서는 사용할 수 없습니다.

```
'timeout server  50000'
```

- server의 최대 비활성 시간 설정입니다.

- millisecond로 설정하며, frontend section에서는 사용할 수 없습니다.

#### 2.18. 'errorfile'
```
'errorfile 400 /etc/haproxy/errors/400.http'
'errorfile 403 /etc/haproxy/errors/403.http'
'errorfile 408 /etc/haproxy/errors/408.http'
'errorfile 500 /etc/haproxy/errors/500.http'
'errorfile 502 /etc/haproxy/errors/502.http'
'errorfile 503 /etc/haproxy/errors/503.http'
'errorfile 504 /etc/haproxy/errors/504.http'
```

- haproxy의 오류(error) file 설정입니다.

- HTTP 상태(status) 코드(code)에 맞춰서 error file을 연결합니다.


## 마무리(CONCLUSION)
ubuntu 환경에 haproxy 설정을 완료했습니다.
<br /><br />
haproxy 설정에 대한 더 자세한 내용은 아래 참고 페이지를 확인해 주시기 바랍니다.
<br /><br />
다음 포스트에서는 haproxy를 활용한 서브도메인(subdomain) 설정 방법을 소개하겠습니다.


## 참고(REFERENCES)
- [https://cbonte.github.io/haproxy-dconv/1.8/configuration.html](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html){: target="\_blank"}
