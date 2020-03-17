---
title: "우분투(Ubuntu) 환경에 방화벽(UFW) 설정하기"
categories: 
  - ubuntu
tags: 
  - ufw
  - ubuntu
---


방화벽(UFW, uncomplicated firewall)은 데비안(debian) 계열 및 다양한 리눅스(linux) 환경에서 작동되고, GPL(GNU General Public License)이 적용되며 파이썬(python)으로 개발되었습니다. <br />
사용하기 쉬운 CLI(command line interface, 명령줄 인터페이스)를 사용하고 프로그램 구성에는 iptables를 사용하는 netfilter 방화벽(firewall)을 관리하는 프로그램입니다. <br />
ufw는 기본적으로 ubuntu 18.04 LTS 이후 버전에서 사용할 수 있습니다. <br />
이 포스트에서는 ubuntu 환경에서 ufw를 설정하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- ubuntu 환경이 필요합니다.

> Ubuntu 설치 방법은 [VMware workstation에 ubuntu server 16.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1604-installation/){: target="\_blank"} 또는 [VMware workstation에 ubuntu server 18.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1804-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.1 build-15018445)
- Ubuntu 18.04.3 LTS (Bionic Beaver) Server (64-bit)


## 요약(SUMMARY)
1. ufw 활성화 또는 비활성화
2. ufw 기본 정책(default rules) 조회
3. ufw default rules 허용 또는 차단
4. ufw rule 허용 또는 차단
5. ufw rule 삭제
6. 서비스 명으로 ufw rule 허용 또는 차단
7. IP 주소로 ufw rule 허용 또는 차단
8. ufw ping(icmp) 허용 또는 차단
9. (선택사항) apt 명령어로 ufw 삭제


## 내용(CONTENTS)
### 1. ufw 활성화 또는 비활성화
#### 1.1. ufw 활성화
```console
$ sudo ufw enable
```

> ufw은 기본적으로 비활성화되어 있습니다.

#### 1.2. ufw 비활성화
```console
$ sudo ufw disable
```

### 2. ufw 상태 조회
```console
$ sudo ufw status verbose
Status: inactive
```

### 3. ufw default rules 조회
```console
$ sudo ufw show raw
```

> 아래 경로의 하위 파일을 조회하여 확인할 수 있습니다.

```console
$ sudo cat /etc/ufw/user.rules
*filter
:ufw-user-input - [0:0]
:ufw-user-output - [0:0]
:ufw-user-forward - [0:0]
:ufw-before-logging-input - [0:0]
:ufw-before-logging-output - [0:0]
:ufw-before-logging-forward - [0:0]
:ufw-user-logging-input - [0:0]
:ufw-user-logging-output - [0:0]
:ufw-user-logging-forward - [0:0]
:ufw-after-logging-input - [0:0]
:ufw-after-logging-output - [0:0]
:ufw-after-logging-forward - [0:0]
:ufw-logging-deny - [0:0]
:ufw-logging-allow - [0:0]
:ufw-user-limit - [0:0]
:ufw-user-limit-accept - [0:0]
### RULES ###

### END RULES ###

### LOGGING ###
-A ufw-after-logging-input -j LOG --log-prefix "[UFW BLOCK] " -m limit --limit 3/min --limit-burst 10
-A ufw-after-logging-forward -j LOG --log-prefix "[UFW BLOCK] " -m limit --limit 3/min --limit-burst 10
-I ufw-logging-deny -m conntrack --ctstate INVALID -j RETURN -m limit --limit 3/min --limit-burst 10
-A ufw-logging-deny -j LOG --log-prefix "[UFW BLOCK] " -m limit --limit 3/min --limit-burst 10
-A ufw-logging-allow -j LOG --log-prefix "[UFW ALLOW] " -m limit --limit 3/min --limit-burst 10
### END LOGGING ###

### RATE LIMITING ###
-A ufw-user-limit -m limit --limit 3/minute -j LOG --log-prefix "[UFW LIMIT BLOCK] "
-A ufw-user-limit -j REJECT
-A ufw-user-limit-accept -j ACCEPT
### END RATE LIMITING ###
COMMIT
```

### 4. ufw default rules 허용 또는 차단
#### 4.1. ufw default rules 허용
```console
$ sudo ufw default allow
```

#### 4.2. ufw default rules 차단
```console
$ sudo ufw default deny
```

### 5. ufw rule 허용 또는 차단
#### 5.1. TCP 8080 포트(port) 허용
```console
$ sudo ufw allow 8080/tcp
```

#### 5.2. TCP 8080 port 차단
```console
$ sudo ufw deny 8080/tcp
```

#### 5.3. UDP 22 port 허용
```console
$ sudo ufw allow 22/udp
```

#### 5.4. UDP 22 port 차단
```console
$ sudo ufw deny 22/udp
```

#### 5.5. TCP/UDP 53 port 허용
```console
$ sudo ufw allow 53
```

#### 5.6. TCP/UDP 53 port 차단
```console
$ sudo ufw deny 53
```

> 53 port는 DNS 사용 port입니다.

### 6. ufw rule 삭제
#### 6.1. TCP 8080 port 차단 rule 삭제
```console
$ sudo ufw delete deny 8080/tcp
```

#### 6.2. UDP 22 port 차단 rule 삭제
```console
$ sudo ufw delete deny 22/udp
```

#### 6.3. TCP/UDP 53 port 차단 rule 삭제
```console
$ sudo ufw delete deny 53
```

### 7. 서비스(service) 명으로 ufw rule 허용 또는 차단

> 아래 명령어로 service 목록을 조회할 수 있습니다.
```console
$ cat /etc/services
```

#### 7.1. SSH service 허용
```console
$ sudo ufw allow ssh
```

#### 7.2. SSH service 차단
```console
$ sudo ufw deny ssh
```

### 8. IP 주소(address)로 ufw rule 허용 또는 차단
#### 8.1. IP address 허용
```console
$ sudo ufw allow from 192.168.10.20
```

#### 8.2. IP address 차단
```console
$ sudo ufw deny from 192.168.10.20
```

#### 8.3. IP address subnet(net mask) 허용
```console
$ sudo ufw allow from 192.168.10.0/24
```

#### 8.4. IP address subnet(net mask) 차단
```console
$ sudo ufw deny from 192.168.10.0/24
```

#### 8.5. IP address와 port 허용
```console
$ sudo ufw allow from 192.168.10.20 to any port 22
```

#### 8.6. IP address와 port 차단
```console
$ sudo ufw deny from 192.168.10.20 to any port 22
```

#### 8.7. IP address와 port, protocol 허용
```console
$ sudo ufw allow from 192.168.10.20 to any port 22 proto tcp
```

#### 8.8. IP address와 port, protocol 차단
```console
$ sudo ufw deny from 192.168.10.20 to any port 22 proto tcp
```

### 9. ufw ping(icmp) 허용 또는 차단

> ufw는 기본적으로 ping 요청을 허용합니다.

#### 9.1. ufw ping(icmp) 허용
```console
$ sudo vi /etc/ufw/before.rules
```

```shell
...
# ok icmp codes
-A ufw-before-input -p icmp --icmp-type destination-unreachable -j ACCEPT
-A ufw-before-input -p icmp --icmp-type source-quench -j ACCEPT
-A ufw-before-input -p icmp --icmp-type time-exceeded -j ACCEPT
-A ufw-before-input -p icmp --icmp-type parameter-problem -j ACCEPT
-A ufw-before-input -p icmp --icmp-type echo-request -j ACCEPT
...
```

#### 9.2. ufw ping(icmp) 차단
```console
$ sudo vi /etc/ufw/before.rules
```

```shell
...
# ok icmp codes
-A ufw-before-input -p icmp --icmp-type destination-unreachable -j DROP
-A ufw-before-input -p icmp --icmp-type source-quench -j DROP
-A ufw-before-input -p icmp --icmp-type time-exceeded -j DROP
-A ufw-before-input -p icmp --icmp-type parameter-problem -j DROP
-A ufw-before-input -p icmp --icmp-type echo-request -j DROP
...
```

> 'ACCEPT'를 'DROP'으로 변경합니다.

### 10. (선택사항) apt 명령어로 ufw 삭제
- 설정 파일을 유지하며 ufw를 삭제합니다.

```console
$ sudo apt remove ufw
$ sudo apt remove --auto-remove ufw
```

- 설정 파일과 함께 ufw를 삭제합니다. (단, 사용자 홈 디렉터리의 설정 파일은 유지됩니다.)
```console
$ sudo apt purge ufw
$ sudo apt purge --auto-remove ufw
```


## 마무리(CONCLUSION)
ubuntu 환경에 ufw 설정을 완료했습니다. <br />
다음 포스트에서는 [우분투(Ubuntu) 환경에 iptables 설정하기](https://lindarex.github.io/ubuntu/ubuntu-iptables-setting/){: target="\_blank"}를 소개하겠습니다.


## 참고(REFERENCES)
- [https://help.ubuntu.com/community/UFW](https://help.ubuntu.com/community/UFW){: target="\_blank"}
- [http://manpages.ubuntu.com/manpages/bionic/en/man8/ufw.8.html](http://manpages.ubuntu.com/manpages/bionic/en/man8/ufw.8.html){: target="\_blank"}
