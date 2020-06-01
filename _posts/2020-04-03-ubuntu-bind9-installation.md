---
title: "우분투(Ubuntu) 환경에 패키지(Package)로 BIND(BIND9) 설치하기"
categories: 
  - bind9
tags: 
  - bind9
  - dns
  - "open source software"
  - "open source"
  - oss
  - ubuntu
---


BIND(bind9, berkeley internet name domain)는 안정성과 고품질로 유닉스(unix) 및 리눅스(linux)에서 널리 사용되며, 모든 기능을 갖춘 매우 유연한 DNS(dns, domain name server) 시스템(system)입니다. <br />
bind9은 MPL(mozilla public license) 2.0 라이선스(license)가 적용되는 오픈소스(open source) 소프트웨어이며, 인터넷에 dns 정보를 게시 할 수 있을 뿐만 아니라 사용자의 dns 쿼리를 처리할 수 있습니다.  <br />
bind9은 내부 dns 서버(server)를 설정하여, server가 개인 호스트(host) 이름과 개인 IP 주소(address)를 확인하는 데 사용합니다. <br />
이를 통해 내부 host 이름과 개인 IP address를 중앙에서 관리할 수 ​​있으며, 이는 사용자 환경이 몇 개 이상의 host로 확장될 때 필요합니다. <br />
이 포스트에서는 우분투(ubuntu) 환경에서 package로 bind9을 설치하는 방법을 소개합니다.

> server 구성 및 인프라(Infrastructure) 관리 시에 중요한 부분은 dns를 설정하여 네트워크(network) 인터페이스와 IP address를 쉽게 검색할 방법을 유지하는 것입니다. <br />
IP address 대신 FQDN(fully qualified domain name)을 사용하여 network address를 지정하면, 애플리케이션(application) 및 서비스(service) 구성이 쉬워지고 구성 파일의 유지 관리가 향상됩니다.


## 선행조건(PREREQUISITE)
- ubuntu 환경이 필요합니다.

> ubuntu 설치 방법은 [VMware workstation에 ubuntu server 16.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1604-installation/){: target="\_blank"} 또는 [VMware workstation에 ubuntu server 18.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1804-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.2 build-15785246)
- Ubuntu 18.04.4 LTS (Bionic Beaver) Server (64-bit)
- BIND 9.11.3


## 요약(SUMMARY)
1. apt 명령어로 bind9 설치
2. bind9 설치 확인
3. systemctl 명령어로 bind9 서비스 관리
4. (선택사항) apt 명령어로 bind9 삭제


## 내용(CONTENTS)
### 1. apt 명령어로 bind9 설치
```console
$ sudo apt update && sudo apt install bind9 bind9utils bind9-doc -y
```

### 2. bind9 설치 확인
```console
$ named -V
BIND 9.11.3-1ubuntu1.11-Ubuntu (Extended Support Version) <id:a375815>
running on Linux x86_64 4.15.0-91-generic #92-Ubuntu SMP Fri Feb 28 11:09:48 UTC 2020
built by make with '--build=x86_64-linux-gnu' '--prefix=/usr' '--includedir=/usr/include' '--mandir=/usr/share/man' '--infodir=/usr/share/info' '--sysconfdir=/etc' '--localstatedir=/var' '--disable-silent-rules' '--libdir=/usr/lib/x86_64-linux-gnu' '--libexecdir=/usr/lib/x86_64-linux-gnu' '--disable-maintainer-mode' '--disable-dependency-tracking' '--libdir=/usr/lib/x86_64-linux-gnu' '--sysconfdir=/etc/bind' '--with-python=python3' '--localstatedir=/' '--enable-threads' '--enable-largefile' '--with-libtool' '--enable-shared' '--enable-static' '--with-gost=no' '--with-openssl=/usr' '--with-gssapi=/usr' '--with-libjson=/usr' '--without-lmdb' '--with-gnu-ld' '--with-geoip=/usr' '--with-atf=no' '--enable-ipv6' '--enable-rrl' '--enable-filter-aaaa' '--enable-native-pkcs11' '--with-pkcs11=/usr/lib/softhsm/libsofthsm2.so' '--with-randomdev=/dev/urandom' '--with-eddsa=no' 'build_alias=x86_64-linux-gnu''CFLAGS=-g -O2 -fdebug-prefix-map=/build/bind9-uW3Pyl/bind9-9.11.3+dfsg=. -fstack-protector-strong -Wformat -Werror=format-security -fno-strict-aliasing -fno-delete-null-pointer-checks -DNO_VERSION_DATE -DDIG_SIGCHASE' 'LDFLAGS=-Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,-z,now' 'CPPFLAGS=-Wdate-time -D_FORTIFY_SOURCE=2'
compiled by GCC 7.4.0
compiled with OpenSSL version: OpenSSL 1.1.1  11 Sep 2018
linked to OpenSSL version: OpenSSL 1.1.1  11 Sep 2018
compiled with libxml2 version: 2.9.4
linked to libxml2 version: 20904
compiled with libjson-c version: 0.12.1
linked to libjson-c version: 0.12.1
compiled with zlib version: 1.2.11
linked to zlib version: 1.2.11
threads support is enabled
```

### 3. systemctl 명령어로 bind9 서비스(service) 관리
#### 3.1. bind9 service 설정 반영
```console
$ sudo systemctl daemon-reload
```

#### 3.2. bind9 service 시작
```console
$ sudo systemctl start bind9.service
```

#### 3.3. bind9 service 중지
```console
$ sudo systemctl stop bind9.service
```

#### 3.4. bind9 service 재시작
```console
$ sudo systemctl restart bind9.service
```

#### 3.5. bind9 service 설정 재적용
```console
$ sudo systemctl reload bind9.service
```

#### 3.6. bind9 service 상태 조회
```console
$ sudo systemctl status bind9.service
```

#### 3.7. bind9 service 활성화(부팅 시 자동 시작)
```console
$ sudo systemctl enable bind9.service
```

#### 3.8. bind9 service 비활성화
```console
$ sudo systemctl disable bind9.service
```

#### 3.9. bind9 service 및 관련 프로세스 모두 중지
```console
$ sudo systemctl kill bind9.service
```

### 4. (선택사항) apt 명령어로 bind9 삭제
- '--auto-remove' 옵션을 추가하면, 사용하지 않는 관련 package를 모두 삭제합니다.

#### 4.1. apt remove 명령어로 bind9 삭제
- 설정 파일을 유지하며 bind9을 삭제합니다.

```console
$ sudo apt remove bind9*
$ sudo apt remove --auto-remove bind9*
```

#### 4.2. apt purge 명령어로 bind9 삭제
- 설정 파일과 함께 bind9을 삭제합니다. (단, 사용자 홈 디렉터리의 설정 파일은 유지됩니다.)

```console
$ sudo apt purge bind9*
$ sudo apt purge --auto-remove bind9*
```


## 마무리(CONCLUSION)
ubuntu 환경에 package로 bind9 설치를 완료했습니다. <br />
bind9 설정은 [우분투(Ubuntu) 환경에 BIND(BIND9) 설정하기](https://lindarex.github.io/bind9/ubuntu-bind9-setting/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 참고(REFERENCES)
- [https://www.isc.org/bind/](https://www.isc.org/bind/){: target="\_blank"}
- [https://www.linuxbabe.com/ubuntu/set-up-authoritative-dns-server-ubuntu-18-04-bind9](https://www.linuxbabe.com/ubuntu/set-up-authoritative-dns-server-ubuntu-18-04-bind9){: target="\_blank"}
- [https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-ubuntu-18-04){: target="\_blank"}
- [https://www.linuxtechi.com/install-configure-bind-9-dns-server-ubuntu-debian/](https://www.linuxtechi.com/install-configure-bind-9-dns-server-ubuntu-debian/){: target="\_blank"}
