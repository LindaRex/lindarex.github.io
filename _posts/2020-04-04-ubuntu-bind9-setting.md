---
title: "우분투(Ubuntu) 환경에 BIND(BIND9) 설정하기"
categories: 
  - bind9
tags: 
  - bind9
  - ubuntu
---


BIND(bind9, berkeley internet name domain)는 DNS(domain name system)를 제공하는 서버(server), 즉 네임 서버(name server) 기능을 갖춘 오픈소스(open source) 소프트웨어입니다. <br />
bind9 설정은 인프라(infrastructure) 환경과 용도에 따라 다양하지만, 이 포스트에서는 간단한 구성으로 ubuntu 환경에서 bind9을 설정하는 방법을 소개합니다.

포스트 작성 전에 DNS server와 name server, DNS에 대해 간략히 설명합니다. <br />
DNS server는 도메인(domain) 이름과 관련 기록을 관리, 유지, 처리하는 name server의 일종이며, DNS 쿼리(query)에 응답하는 server입니다.  <br />
name server는 네이밍(naming) 서비스를 제공하는 모든 server인데, 일반적으로 로컬 DNS server로 알려져 있으며, DNS server를 찾는 데 사용됩니다.  <br />
name server는 domain 이름을 IP 주소(address)로 변환합니다. 이를 통해 사용자는 웹 사이트의 실제 IP address 대신 domain 이름을 입력하여 웹 사이트에 액세스할 수 있습니다.  <br />
DNS는 호스트(host) 이름을 IP address로 변환하는 분산 시스템입니다.  <br />
DNS는 주로 TCP/IP 네트워킹 프로토콜(protocol) 제품군과 함께 domain 이름을 확인하는 데 사용되지만, 다른 용도로도 사용되기도 합니다.  <br />
즉, DNS server는 기본적으로 name server 유형이지만, 모든 name server가 DNS server인 것은 아닙니다. <br />
하지만 DNS server가 가장 일반적인 name server이므로, name server와 DNS server는 서로 바꿔 사용할 수 있습니다.

> DNS server와 name server, DNS에 대한 자세한 정보는 [https://www.quora.com/What-is-the-difference-if-any-between-DNS-server-and-name-server](https://www.quora.com/What-is-the-difference-if-any-between-DNS-server-and-name-server){: target="\_blank"}를 확인해 주시기 바랍니다.


## 선행조건(PREREQUISITE)
- ubuntu 환경에 bind9이 설치되어 있어야 합니다.
- 방화벽 설정이 필요합니다.
    + TCP 및 UDP 53 포트가 개방되어 있어야 합니다.

> bind9 설치 방법은 [우분투(Ubuntu) 환경에 패키지(Package)로 BIND(BIND9) 설치하기](https://lindarex.github.io/bind9/ubuntu-bind9-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

> 방화벽 설정 방법은 [우분투(Ubuntu) 환경에 방화벽(Firewalld) 설치 및 설정하기](https://lindarex.github.io/ubuntu/ubuntu-firewalld-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.2 build-15785246)
- Ubuntu 18.04.4 LTS (Bionic Beaver) Server (64-bit)
- BIND 9.11.3

> 아래 VM(virtual machine) 정보는 예시입니다. 고정 IP 설정은 이 포스트에서 다루지 않습니다.

- bind9을 설치한 VM 정보
    + Host :: ns
    + Private FQDN :: ns.lindarex.local
    + Private IP address :: 10.0.1.32

- bind9 테스트를 위한 외부 VM 정보
    + Host :: test
    + Private FQDN :: test.lindarex.local
    + Private IP address :: 10.0.1.31

> FQDN(fully qualified domain name)이란 전체 주소 도메인 이름으로 host 이름과 domain 이름을 포함합니다.

## 요약(SUMMARY)
1. bind9 DNS 서버 VM 호스트 설정
2. bind9 DNS 서버 설정
3. bind9 로컬 DNS 서버 및 영역(ZONE) 설정
4. bind9 설정 확인
5. 네트워크 명령어로 bind9 DNS 서버 VM에서 DNS 확인
6. 네트워크 명령어로 외부 VM에서 DNS 확인


## 내용(CONTENTS)
### 1. DNS 서버 VM 호스트 설정
#### 1.1. /etc/hosts 설정
```console
$ sudo vi /etc/hosts
```

```shell
--------------------------------------------------------------------------------
127.0.0.1 localhost
127.0.1.1 ns.lindarex.local
10.0.1.32 ns.lindarex.local

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
--------------------------------------------------------------------------------
```

#### 1.2. /etc/hostname 설정
```console
$ sudo vi /etc/hostname
```

```shell
--------------------------------------------------------------------------------
ns.lindarex.local
--------------------------------------------------------------------------------
```

> host 설정 후 VM 재시작이 필요합니다.

### 2. bind9 DNS 서버 설정
```console
$ sudo vi /etc/bind/named.conf.options
```

```shell
--------------------------------------------------------------------------------
options {
	directory "/var/cache/bind";
	dnssec-validation auto;
	auth-nxdomain no;
    listen-on port 53 { localhost; 10.0.1.0/24; };
    allow-query { any; };
    forwarders { 8.8.8.8; };
    recursion yes;
};
--------------------------------------------------------------------------------
```

- 위 설정을 설명합니다.
    + 'directory "/var/cache/bind";' :: 기본값은 '/var/cache/bind'입니다. server의 작업 디렉터리(directory)를 정의하며 절대 경로(path)입니다.
    + 'dnssec-validation auto;' :: 기본값은 'auto'입니다. 'auto'로 지정하면, DNSSEC 유효성 검사가 활성화되고 DNS root zone에 기본 trust anchor가 사용됩니다.
    + 'auth-nxdomain no;' :: 기본값은 'no'입니다. 오래된 DNS 소프트웨어를 사용한다면 'yes'로 설정합니다.
    + 'listen-on port 53 { localhost; 10.0.1.0/24; };' :: server가 query에 응답할 인터페이스(interface)와 포트(port)를 지정합니다.
    + 'allow-query { any; };' :: DNS query를 할 수 있는 host를 지정합니다.
    + 'forwarders { 8.8.8.8; };' :: 포워딩(forwarding)에 사용할 IP address를 지정합니다.
    + 'recursion yes;' :: 기본값은 'yes'이며, 재귀(recursive) query를 활성화합니다.

> DNSSEC(domain name system security extensions)에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/DNSSEC](https://ko.wikipedia.org/wiki/DNSSEC){: target="\_blank"}와 [https://한국인터넷정보센터.한국/jsp/resources/dns/dnssecInfo/dnssecInfo.jsp](https://xn--3e0bx5euxnjje69i70af08bea817g.xn--3e0b707e/jsp/resources/dns/dnssecInfo/dnssecInfo.jsp){: target="\_blank"}를 확인해 주시기 바랍니다.

### 3. bind9 로컬 DNS 서버 및 영역(ZONE) 설정
#### 3.1. /etc/bind/named.conf.local 설정
- 정방향(forward) 및 역방향(reverse) 영역(zone)을 지정합니다.

> forward zone(forward lookup zone)은 host 이름 또는 FQDN에 대한 IP address를 관리하는 DNS zone이고, reverse zone(reverse lookup zone)은 IP address에 대응하는 host 이름 또는 FQDN을 관리하는 DNS zone입니다.

```console
$ sudo vi /etc/bind/named.conf.local
```

```shell
--------------------------------------------------------------------------------
// FORWARD ZONE
zone "lindarex.local" IN {
  type master;
  file "lindarex.local.zone";
};

// REVERSE ZONE
zone "1.0.10.in-addr.arpa" IN {
  type master;
  file "lindarex.local.zone.rev";
};
--------------------------------------------------------------------------------
```

- 위 설정을 설명합니다.
    + 'zone "lindarex.local" IN {' :: forward zone의 domain 이름을 지정합니다. 'IN'은 클래스(class)를 명시한 것으로 인터넷(internet)을 지정한 것입니다. 지정하지 않으면 기본값은 internet이지만, 예제를 위해 지정했습니다. 'hesiod', 'CHAOS' class 등이 존재합니다.
    + 'type master;' :: zone type을 'master'로 지정합니다. 'primary'와 같은 의미이며, slave (또는 secondary), mirror, delegation-only, forward, hint, redirect, static-stub, stub 등이 존재합니다.
    + 'file "lindarex.local.zone";' :: zone 파일(file)의 path와 이름을 지정합니다. 절대 path로 지정하지 않으면, '/etc/bind/named.conf.options'에서 설정한 directory 설정을 루트(root) directory로 사용합니다.
    + 'zone "1.0.10.in-addr.arpa" IN {' :: reverse zone의 IP 영역을 지정합니다. 역순으로 기재하는 것을 주의합니다.

    > 위 예제는 서브넷 마스크(subnet mask)가 '10.0.1.0/24'이기 때문에 '1.0.10.in-addr.arpa'으로 설정합니다. 만약 subnet mask가 '10.0.1.0/16' 이라면, '0.10.in-addr.arpa'으로 설정합니다.

#### 3.2. /var/cache/bind/lindarex.local.zone 생성
- forward zone file을 생성합니다.

```console
$ sudo vi /var/cache/bind/lindarex.local.zone
```

```shell
--------------------------------------------------------------------------------
$TTL	86400
@	IN	SOA	ns.lindarex.local. root.ns.lindarex.local. (
			      1		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			  86400 )	; Negative Cache TTL
;
@	IN	NS	ns.lindarex.local.
ns	IN	A	10.0.1.32
--------------------------------------------------------------------------------
```

- 위 설정을 설명합니다.
    + '$TTL	86400' :: resolver가 record를 캐시(cache) 할 시간(초, seconds)을 정의합니다. TTL(time-to-live)을 '0'으로 설정하면 record를 cache 하지 않습니다.
    + '@	IN	SOA	ns.lindarex.local. root.ns.lindarex.local. (' :: SOA(start of authority)는 zone(domain)에 대한 전역(global) 매개변수(parameters)와 zone 이름을 정의합니다. zone file에는 하나의 SOA RR(resource records)만 허용됩니다.
    + '1		; Serial' :: serial number를 정의합니다. 1에서 4294967295 사이의 부호 없는 32비트(bit) 값(최대 2147483647)이며, 10자리 필드로 정의됩니다. 이 값은 zone file의 RR이 업데이트될 때 증가해야 합니다.
    + '604800		; Refresh' :: refresh seconds를 정의합니다. 32 bit 값이며, 슬레이브(slave)가 마스터(master) DNS SOA RR을 읽어서 master에서 zone을 새로 고치려고 시도하는 시간을 설정합니다.
    + '86400		; Retry' :: update retry seconds를 정의합니다. 32 bit 값이며, 'refresh' 만료 또는 NOTIFY 메시지(message) 수신, slave(secondary)가 master에 연결하지 못하는 경우의 재시도 간격을 지정합니다.
    + '2419200		; Expire' :: expire seconds를 정의합니다. 32 bit 값이며, zone 데이터(data)가 더는 권한이 없는 경우를 나타냅니다. slave(secondary) server에서만 사용됩니다.
    + '86400 )	; Negative Cache TTL' :: negative cache TTL seconds를 정의합니다. 32 bit 값이며, 이전에는 'nx = nxdomain ttl'로 정의했었습니다. 'NAME ERROR = NXDOMAIN' 결과는 resolver에 의해 cache 되고, 허용하는 최댓값은 3시간(10800 seconds)입니다.
    + '@	IN	NS	ns.lindarex.local.' :: DNS generic record format(textual format)입니다. 아래 설명을 기재합니다.
        - 일반적으로 'owner-name', 'class', 'type', 'type-specific-data'로 구성됩니다.
        - 'owner-name' :: record가 속한 zone file에서 노드(node)의 소유자 이름 또는 레이블(label)입니다. 위 예제에서는 '@'이며, '$ORIGIN'을 대체합니다.
        - 'class' :: protocol 패밀리 또는 protocol 인스턴스(instance)를 정의합니다. 16 bit 값이며, 위 예제에서는 'IN'이며, internet protocol입니다.
        - 'type' :: 'type-specific-data'의 값을 결정하는 RR 유형이며, 위 예제에서는 'NS'입니다. 'NS'는 name server를 의미하며, SOA record로 정의된 domain 또는 하위(sub) domain의 권한 있는 name server를 지정합니다.
        - 'type-specific-data' :: 'class'와 'type' 값에 의해 정의됩니다. 위 예제에서는 'ns.lindarex.local.'입니다.
    + 'ns	IN	A	10.0.1.32' :: 아래 설명을 기재합니다.
        - 'owner-name' :: 위 예제에서는 'ns'이며, host 이름을 지정합니다.
        - 'class' :: 위 예제에서는 'IN'이며, internet protocol입니다.
        - 'type' :: 위 예제에서는 'A'입니다. 'A'는 IPv4 address record, host의 IPv4 address입니다.
        - 'type-specific-data' :: 위 예제에서는 '10.0.1.32'입니다.

#### 3.3. /var/cache/bind/lindarex.local.zone.rev 생성
- reverse zone file을 생성합니다.

```console
$ sudo vi /var/cache/bind/lindarex.local.zone.rev
```

```shell
--------------------------------------------------------------------------------
$TTL	86400
@	IN	SOA	ns.lindarex.local. root.ns.lindarex.local. (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			  86400 )	; Negative Cache TTL
;
@	IN	NS	ns.
32	IN	PTR	ns.lindarex.local.
--------------------------------------------------------------------------------
```

- forward zone file 설정과 다른 부분만 설명합니다.
    + '@	IN	NS	ns.' :: 아래 설명을 기재합니다.
        - 'owner-name' :: 위 예제에서는 '@'이며, '$ORIGIN'을 대체합니다.
        - 'class' :: 위 예제에서는 'IN'이며, internet protocol입니다.
        - 'type' :: 위 예제에서는 'NS'이며, name server입니다.
        - 'type-specific-data' :: 위 예제에서는 'ns.'이며, host 이름을 지정합니다.
    + '32	IN	PTR	ns.lindarex.local.' :: 아래 설명을 기재합니다.
        - 'owner-name' :: 위 예제에서는 '32'이며, bind9 DNS server VM의 IP address가 '10.0.1.32'이고, subnet mask가 '10.0.1.0/24'이기 때문에 역순으로 '32'를 지정합니다. 만약 subnet mask가 '10.0.1.0/16'이면, '32.1'로 설정해야 합니다.
        - 'class' :: 위 예제에서는 'IN'이며, internet protocol입니다.
        - 'type' :: 위 예제에서는 'PTR'입니다. 'PTR'은 pointer, reverse DNS입니다
        - 'type-specific-data' :: 위 예제에서는 'ns.lindarex.local.'입니다.

### 4. bind9 설정 확인
#### 4.1. bind9 설정 file 문법 확인
```console
$ named-checkconf
```

> 아무 결과가 나오지 않아야 문법(syntax) 오류가 없는 것입니다.

#### 4.2. bind9 zone file syntax 확인
- forward zone file의 syntax를 확인합니다.

```console
$ named-checkzone lindarex.local /var/cache/bind/lindarex.local.zone
zone lindarex.local/IN: loaded serial 1
OK
```

- reverse zone file의 syntax를 확인합니다.

```console
$ named-checkzone 10.0.1.32 /var/cache/bind/lindarex.local.zone.rev
zone 10.0.1.32/IN: loaded serial 2
OK
```

- bind9 설정에 syntax 오류가 없다면, bind9을 재시작합니다.

```console
$ sudo systemctl restart bind9.service
```

### 5. 네트워크 명령어로 bind9 DNS 서버 VM에서 DNS 확인

- 아래 명령어는 bind9을 설치한 VM(10.0.1.32)에서 실행합니다.

#### 5.1. nslookup 명령어(command)로 확인
```console
$ nslookup ns.lindarex.local
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
Name:	ns.lindarex.local
Address: 127.0.1.1
Name:	ns.lindarex.local
Address: 10.0.1.32
```

```console
$ nslookup ns.lindarex.local localhost.localdomain
Server:		localhost.localdomain
Address:	::1#53

Name:	ns.lindarex.local
Address: 10.0.1.32
```

> nslookup command에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/Nslookup](https://ko.wikipedia.org/wiki/Nslookup){: target="\_blank"}를 확인해 주시기 바랍니다.

#### 5.2. dig command로 확인
```console
$ dig ns.lindarex.local

; <<>> DiG 9.11.3-1ubuntu1.11-Ubuntu <<>> ns.lindarex.local
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 36078
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;ns.lindarex.local.		IN	A

;; ANSWER SECTION:
ns.lindarex.local.	0	IN	A	127.0.1.1
ns.lindarex.local.	0	IN	A	10.0.1.32

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Sat Apr 04 05:17:52 UTC 2020
;; MSG SIZE  rcvd: 78
```

```console
$ dig -x 10.0.1.32

; <<>> DiG 9.11.3-1ubuntu1.11-Ubuntu <<>> -x 10.0.1.32
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 14862
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;32.1.0.10.in-addr.arpa.		IN	PTR

;; ANSWER SECTION:
32.1.0.10.in-addr.arpa.	0	IN	PTR	ns.lindarex.local.

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Sat Apr 04 05:18:04 UTC 2020
;; MSG SIZE  rcvd: 100
```

> dig command에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/Dig](https://ko.wikipedia.org/wiki/Dig){: target="\_blank"}를 확인해 주시기 바랍니다.

### 6. 네트워크 명령어로 외부 VM에서 DNS 확인

- 아래 command는 외부 VM(10.0.1.0/24)에서 실행합니다.
- 외부 VM에 name server 설정이 필요합니다.

<!-- > name server 설정 방법은 [우분투(Ubuntu) 환경에 네임 서버(name server) 설정하기](https://lindarex.github.io/ubuntu/ubuntu-nameserver-setting/){: target="\_blank"} 포스트를 참고하시기 바랍니다. -->

#### 6.1. nslookup command로 확인
```console
$ nslookup ns.lindarex.local
Server:		10.0.1.32
Address:	10.0.1.32#53

Name:	ns.lindarex.local
Address: 10.0.1.32
```

```console
$ nslookup 10.0.1.32
32.1.0.10.in-addr.arpa	name = ns.lindarex.local.

```

#### 6.2. dig command로 확인
```console
$ dig ns.lindarex.local

; <<>> DiG 9.11.3-1ubuntu1.11-Ubuntu <<>> ns.lindarex.local
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37938
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 4d8ecf3a8c01a1e3c63e18595e8ac359738e244f6d3d1ec2 (good)
;; QUESTION SECTION:
;ns.lindarex.local.		IN	A

;; ANSWER SECTION:
ns.lindarex.local.	86400	IN	A	10.0.1.32

;; AUTHORITY SECTION:
lindarex.local.		86400	IN	NS	ns.lindarex.local.

;; Query time: 0 msec
;; SERVER: 10.0.1.32#53(10.0.1.32)
;; WHEN: Sat Apr 04 05:51:22 UTC 2020
;; MSG SIZE  rcvd: 104
```

```console
$ dig -x 10.0.1.32

; <<>> DiG 9.11.3-1ubuntu1.11-Ubuntu <<>> -x 10.0.1.32
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18625
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 37c4348db0ce59742913765d5e8ac375010a08556676faca (good)
;; QUESTION SECTION:
;32.1.0.10.in-addr.arpa.		IN	PTR

;; ANSWER SECTION:
32.1.0.10.in-addr.arpa.	86400	IN	PTR	ns.lindarex.local.

;; AUTHORITY SECTION:
1.0.10.in-addr.arpa.	86400	IN	NS	ns.

;; Query time: 0 msec
;; SERVER: 10.0.1.32#53(10.0.1.32)
;; WHEN: Sat Apr 04 05:51:49 UTC 2020
;; MSG SIZE  rcvd: 144
```


## 마무리(CONCLUSION)
ubuntu 환경에 bind9 설정을 완료했습니다. <br />
일반적으로 엔지니어가 아닌 이상 DNS 설정을 할 기회는 드물지만, bind9 설정을 통해 네트워크 용어와 개념을 조금은 익숙해지길 바라며 이 포스트를 작성했습니다. <br />
bind9 설정에 대한 더 자세한 내용은 아래 참고 페이지를 확인해 주시기 바랍니다.


## 참고(REFERENCES)
- [https://wiki.debian.org/Bind9](https://wiki.debian.org/Bind9){: target="\_blank"}
- [https://bind9.readthedocs.io/en/latest/index.html](https://bind9.readthedocs.io/en/latest/index.html){: target="\_blank"}
- [https://www.zytrax.com/books/dns/ch8/](https://www.zytrax.com/books/dns/ch8/){: target="\_blank"}
