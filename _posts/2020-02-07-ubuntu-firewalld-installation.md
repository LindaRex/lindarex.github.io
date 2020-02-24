---
title: "우분투(Ubuntu) 환경에 방화벽(Firewalld) 설치 및 설정하기"
categories: 
  - ubuntu
tags: 
  - firewalld
  - ubuntu
---


방화벽(이하 firewalld)은 IPv4, IPv6, 이더넷 브리지 및 IPSet의 방화벽 설정을 지원하는 리눅스(이하 linux) 방화벽 관리 도구입니다. <br />
firewalld는 linux 커널 netfilter 프레임워크의 프런트 엔드 역할을 하며, RHEL 7 제품군의 기본 방화벽 관리 소프트웨어입니다. <br />
우분투(이하 Ubuntu)의 기본 방화벽 시스템은 UFW(Uncomplicated Firewall)이지만, firewalld를 설치하고 사용할 수 있습니다. <br />
이 포스트에서는 Ubuntu 환경에서 패키지로 firewalld를 설치하고 설정하는 방법을 소개합니다.

> UFW 설정 방법은 [우분투(Ubuntu) 환경에 방화벽(UFW) 설정하기](https://lindarex.github.io/ubuntu/ubuntu-ufw-setting/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 선행조건(PREREQUISITE)
- Ubuntu 환경이 필요합니다.

> Ubuntu 설치 방법은 [VMware workstation에 Ubuntu 16.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1604-installation/){: target="\_blank"} 또는 [VMware workstation에 Ubuntu 18.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1804-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.1 build-15018445)
- Ubuntu 16.04.4 LTS (Xenial Xerus) Server (64-bit)


## 요약(SUMMARY)
1. apt 명령어로 firewalld 설치
2. firewalld 설치 확인
3. firewalld 설정


## 내용(CONTENTS)
### 1. apt 명령어로 firewalld 설치
```shell
$ sudo apt update -y && sudo apt install firewalld -y
```

### 2. firewalld 설치 확인
```shell
$ sudo firewall-cmd --version
0.4.4.5
```

### 3. firewalld 설정
#### 3.1. firewalld에  rule 추가
> 아래 예제는 영구적으로 public zone에 TCP 8080 포트를 추가하는 명령어입니다.

```shell
$ sudo firewall-cmd --permanent --zone=public --add-port=8080/tcp
```

#### 3.2. firewalld에 rule 적용
> 아래 명령어를 실행하기 전에는 추가한 rule이 적용되지 않습니다.

```shell
$ sudo firewall-cmd --reload
```

#### 3.3. firewalld에 설정된 모든 값 조회
```shell
$ sudo firewall-cmd --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: ssh dhcpv6-client
  ports: 8080/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```


## 마무리(CONCLUSION)
Ubuntu 환경에 패키지로 firewalld 설치 및 설정을 완료했습니다. <br />
firewalld는 CentOS 7부터 이전의 Iptables를 대체해 새롭게 선보인 패킷 필터링 방화벽 프로그램입니다. <br />
다음 포스트에서는 Iptables, UFW 등의 방화벽 특징과 설정 방법을 소개하겠습니다.


## 참고(REFERENCES)
- [http://manpages.ubuntu.com/manpages/bionic/man1/firewall-cmd.1.html](http://manpages.ubuntu.com/manpages/bionic/man1/firewall-cmd.1.html){: target="\_blank"}
- [https://computingforgeeks.com/install-and-use-firewalld-on-ubuntu-18-04-ubuntu-16-04/](https://computingforgeeks.com/install-and-use-firewalld-on-ubuntu-18-04-ubuntu-16-04/){: target="\_blank"}
