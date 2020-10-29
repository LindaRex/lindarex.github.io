---
title: "우분투(Ubuntu) 환경에 패키지(Package)로 fail2ban 설치하기"
categories: 
  - fail2ban
tags: 
  - fail2ban
  - firewall
  - ubuntu
---


Fail2ban은 침입 차단 소프트웨어 프레임워크(framework)로, 파이썬(python) 언어로 개발되었습니다. <br />
로그(log) 파일과 iptables를 이용하여 접속 시도를 확인하고 차단하여 무차별 대입 공격으로부터 시스템을 보호합니다. <br />
패킷 제어 시스템이나 로컬 방화벽(iptables 또는 TCP wrapper)과의 인터페이스를 갖는 POSIX 시스템에서 사용할 수 있으며, GNU General Public License(GNU GPL 또는 GPL) v2+가 적용된 오픈소스(open source) 소프트웨어로 배포(release)됩니다. <br />
이 포스트에서는 우분투(ubuntu) 환경에서 package로 fail2ban을 설치하는 방법을 소개합니다.

> 침입 차단 소프트웨어(Intrusion Prevention Systems (IPS), Intrusion Detection and Prevention Systems (IDPS))에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/침입_차단_시스템](https://ko.wikipedia.org/wiki/%EC%B9%A8%EC%9E%85_%EC%B0%A8%EB%8B%A8_%EC%8B%9C%EC%8A%A4%ED%85%9C){: target="\_blank"}을 확인해 주시기 바랍니다.

> 무차별 대입 공격(brute-force attack)에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/무차별_대입_공격](https://ko.wikipedia.org/wiki/%EB%AC%B4%EC%B0%A8%EB%B3%84_%EB%8C%80%EC%9E%85_%EA%B3%B5%EA%B2%A9){: target="\_blank"}을 확인해 주시기 바랍니다.

> iptables에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/Iptables](https://ko.wikipedia.org/wiki/Iptables){: target="\_blank"}를 확인해 주시기 바랍니다.

> TCP wrapper에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/TCP_래퍼](https://ko.wikipedia.org/wiki/TCP_%EB%9E%98%ED%8D%BC){: target="\_blank"}를 확인해 주시기 바랍니다.

> POSIX 시스템에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/POSIX](https://ko.wikipedia.org/wiki/POSIX){: target="\_blank"}를 확인해 주시기 바랍니다.


## 선행조건(PREREQUISITE)
- ubuntu 환경이 필요합니다.

> ubuntu 설치 방법은 [VMware workstation에 ubuntu server 16.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1604-installation/){: target="\_blank"} 또는 [VMware workstation에 ubuntu server 18.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1804-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- AWS EC2 Instance Ubuntu Server 18.04 LTS (HVM), SSD Volume Type
- Ubuntu 18.04.5 LTS (GNU/Linux 5.4.0-1029-aws x86_64)
- Fail2Ban v0.10.2


## 요약(SUMMARY)
1. apt 명령어로 fail2ban 설치
2. fail2ban 설치 확인
3. systemctl 명령어로 fail2ban 서비스 관리
4. (선택사항) apt 명령어로 fail2ban 삭제
5. fail2ban 설정
6. fail2ban 사용 방법


## 내용(CONTENTS)
### 1. apt 명령어로 fail2ban 설치
```console
$ sudo apt update && sudo apt install fail2ban -y
```

### 2. fail2ban 설치 확인
```console
$ fail2ban-client -V
Fail2Ban v0.10.2

Copyright (c) 2004-2008 Cyril Jaquier, 2008- Fail2Ban Contributors
Copyright of modifications held by their respective authors.
Licensed under the GNU General Public License v2 (GPL).
```

### 3. systemctl 명령어로 fail2ban 서비스(service) 관리
#### 3.1. fail2ban service 설정 반영
```console
$ sudo systemctl daemon-reload
```

#### 3.2. fail2ban service 시작
```console
$ sudo systemctl start fail2ban.service
```

#### 3.3. fail2ban service 중지
```console
$ sudo systemctl stop fail2ban.service
```

#### 3.4. fail2ban service 재시작
```console
$ sudo systemctl restart fail2ban.service
```

#### 3.5. fail2ban service 설정 재적용
```console
$ sudo systemctl reload fail2ban.service
```

#### 3.6. fail2ban service 상태 조회
```console
$ sudo systemctl status fail2ban.service
```

#### 3.7. fail2ban service 활성화(부팅 시 자동 시작)
```console
$ sudo systemctl enable fail2ban.service
```

#### 3.8. fail2ban service 비활성화
```console
$ sudo systemctl disable fail2ban.service
```

#### 3.9. fail2ban service 및 관련 프로세스 모두 중지
```console
$ sudo systemctl kill fail2ban.service
```

### 4. (선택사항) apt 명령어로 fail2ban 삭제
- '--auto-remove' 옵션을 추가하면, 사용하지 않는 관련 package를 모두 삭제합니다.

#### 4.1. apt remove 명령어로 fail2ban 삭제
- 설정 파일을 유지하며 fail2ban을 삭제합니다.

```console
$ sudo apt remove fail2ban*
$ sudo apt remove --auto-remove fail2ban*
```

#### 4.2. apt purge 명령어로 fail2ban 삭제
- 설정 파일과 함께 fail2ban을 삭제합니다. (단, 사용자 홈 디렉터리의 설정 파일은 유지됩니다.)

```console
$ sudo apt purge fail2ban*
$ sudo apt purge --auto-remove fail2ban*
```

### 5. fail2ban 설정
```console
$ sudo vi /etc/fail2ban/jail.conf
```

```console
--------------------------------------------------------------------------------
[DEFAULT]
# 차단 예외 IPs, 추가 시 스페이스로 구분
ignoreip = 127.0.0.1/8

# 접속 차단 시간, 1분 = 60, 아래 예제는 1일, -1 설정 시 영구 차단
bantime = 86400

# 설정 시간 내 maxretry 설정만큼 접속 실패 시 차단
findtime = 86400

# 최대 허용 횟수
maxretry = 3

# 차단 방법, firewalld 사용 시 'firewallcmd-new', iptables 사용 시 'iptables-multiport'
banaction = iptables-multiport


[sshd]
enabled = true
--------------------------------------------------------------------------------
```

> '/etc/fail2ban/jail.conf' 파일은 업데이트 시 초기화되므로, '/etc/fail2ban/jail.d/\*.conf' 또는 '/etc/fail2ban/jail.local' 파일 생성 후 설정하는 것을 추천합니다.

- 설정 반영을 위해 아래 명령어로 fail2ban을 재시작합니다.

```console
$ sudo systemctl restart fail2ban.service
```

### 6. fail2ban 사용 방법
#### 6.1. 차단 목록 조회
```console
$ sudo fail2ban-client status
```

```console
Status
|- Number of jail:	1
`- Jail list:	sshd
```

```console
$ sudo fail2ban-client status sshd
```

```console
Status for the jail: sshd
|- Filter
|  |- Currently failed:	3
|  |- Total failed:	23
|  `- File list:	/var/log/auth.log
`- Actions
   |- Currently banned:	0
   |- Total banned:	0
   `- Banned IP list:
```

#### 6.2. 차단 설정
```console
$ sudo fail2ban-client set ${JAIL_NAME} banip ${차단 IP}
```

```console
$ sudo fail2ban-client set sshd banip 192.168.0.11
```

```console
192.168.0.11
```

#### 6.3. 차단 해제
```console
$ sudo fail2ban-client set ${JAIL_NAME} unbanip ${차단 해제 IP}
```

```console
$ sudo fail2ban-client set sshd unbanip 192.168.0.11
```

```console
192.168.0.11
```

#### 6.4. log 조회
```console
$ cat /var/log/fail2ban.log
```

```console
2020-10-29 05:15:09,537 fail2ban.server         [1588]: INFO    --------------------------------------------------
2020-10-29 05:15:09,537 fail2ban.server         [1588]: INFO    Starting Fail2ban v0.10.2
2020-10-29 05:15:09,548 fail2ban.database       [1588]: INFO    Connected to fail2ban persistent database '/var/lib/fail2ban/fail2ban.sqlite3'
2020-10-29 05:15:09,552 fail2ban.database       [1588]: WARNING New database created. Version '2'
2020-10-29 05:15:09,553 fail2ban.jail           [1588]: INFO    Creating new jail 'sshd'
2020-10-29 05:15:09,584 fail2ban.jail           [1588]: INFO    Jail 'sshd' uses pyinotify {}
2020-10-29 05:15:09,587 fail2ban.jail           [1588]: INFO    Initiated 'pyinotify' backend
2020-10-29 05:15:09,588 fail2ban.filter         [1588]: INFO      maxLines: 1
2020-10-29 05:15:09,614 fail2ban.server         [1588]: INFO    Jail sshd is not a JournalFilter instance
2020-10-29 05:15:09,615 fail2ban.filter         [1588]: INFO    Added logfile: '/var/log/auth.log' (pos = 0, hash = 25791bcc640f76a6b3ead17d55a5f0e66465461c)
2020-10-29 05:15:09,619 fail2ban.filter         [1588]: INFO      encoding: UTF-8
2020-10-29 05:15:09,619 fail2ban.filter         [1588]: INFO      maxRetry: 5
2020-10-29 05:15:09,619 fail2ban.filter         [1588]: INFO      findtime: 600
2020-10-29 05:15:09,619 fail2ban.actions        [1588]: INFO      banTime: 600
2020-10-29 05:15:09,621 fail2ban.jail           [1588]: INFO    Jail 'sshd' started
2020-10-29 05:15:27,281 fail2ban.filter         [1588]: INFO    [sshd] Found 193.112.23.105 - 2020-10-29 05:10:15
2020-10-29 05:15:27,281 fail2ban.filter         [1588]: INFO    [sshd] Found 159.89.1.242 - 2020-10-29 05:10:18
2020-10-29 05:15:27,281 fail2ban.filter         [1588]: INFO    [sshd] Found 51.15.94.14 - 2020-10-29 05:14:53
2020-10-29 05:15:27,282 fail2ban.filter         [1588]: INFO    [sshd] Found 122.51.238.27 - 2020-10-29 05:15:27
2020-10-29 05:26:15,805 fail2ban.filter         [1588]: INFO    [sshd] Found 141.98.10.211 - 2020-10-29 05:26:15
2020-10-29 05:26:19,199 fail2ban.filter         [1588]: INFO    [sshd] Found 141.98.10.212 - 2020-10-29 05:26:19
2020-10-29 05:26:26,824 fail2ban.filter         [1588]: INFO    [sshd] Found 141.98.10.214 - 2020-10-29 05:26:26
2020-10-29 05:26:31,431 fail2ban.filter         [1588]: INFO    [sshd] Found 141.98.10.209 - 2020-10-29 05:26:31
2020-10-29 05:26:35,751 fail2ban.filter         [1588]: INFO    [sshd] Found 141.98.10.210 - 2020-10-29 05:26:35
2020-10-29 05:26:39,393 fail2ban.filter         [1588]: INFO    [sshd] Found 141.98.10.211 - 2020-10-29 05:26:39
2020-10-29 05:26:49,698 fail2ban.filter         [1588]: INFO    [sshd] Found 141.98.10.213 - 2020-10-29 05:26:49
2020-10-29 05:26:55,779 fail2ban.filter         [1588]: INFO    [sshd] Found 141.98.10.214 - 2020-10-29 05:26:55
2020-10-29 05:26:59,919 fail2ban.filter         [1588]: INFO    [sshd] Found 141.98.10.209 - 2020-10-29 05:26:59
2020-10-29 05:27:29,170 fail2ban.filter         [1588]: INFO    [sshd] Found 200.73.128.64 - 2020-10-29 05:27:29
2020-10-29 05:27:49,164 fail2ban.filter         [1588]: INFO    [sshd] Found 137.74.5.81 - 2020-10-29 05:27:49
2020-10-29 05:35:28,107 fail2ban.filter         [1588]: INFO    [sshd] Found 91.74.129.82 - 2020-10-29 05:35:28
2020-10-29 05:36:05,060 fail2ban.filter         [1588]: INFO    [sshd] Found 179.184.0.112 - 2020-10-29 05:36:04
2020-10-29 05:36:49,781 fail2ban.filter         [1588]: INFO    [sshd] Found 161.35.200.233 - 2020-10-29 05:36:49
2020-10-29 05:41:38,714 fail2ban.filter         [1588]: INFO    [sshd] Found 60.248.199.194 - 2020-10-29 05:41:38
2020-10-29 05:41:45,894 fail2ban.filter         [1588]: INFO    [sshd] Found 151.80.176.191 - 2020-10-29 05:41:45
2020-10-29 05:46:13,683 fail2ban.filter         [1588]: INFO    [sshd] Found 122.51.130.21 - 2020-10-29 05:46:13
2020-10-29 05:52:53,319 fail2ban.filter         [1588]: INFO    [sshd] Found 51.83.41.120 - 2020-10-29 05:52:48
2020-10-29 05:55:49,062 fail2ban.filter         [1588]: INFO    [sshd] Found 161.35.162.11 - 2020-10-29 05:55:49
2020-10-29 06:14:12,583 fail2ban.actions        [1588]: NOTICE  [sshd] Ban 192.168.0.11
2020-10-29 06:15:31,272 fail2ban.actions        [1588]: NOTICE  [sshd] Unban 192.168.0.11
```


## 마무리(CONCLUSION)
ubuntu 환경에 package로 fail2ban 설치를 완료했습니다. <br />
fail2ban은 설치만으로 데몬(daemon)으로 실행되면서 기본 설정까지 완료됩니다. <br />
fail2ban은 IPv6 미지원과 분산 무차별 대입 공격에 취약한 단점이 있지만, 기본 설정으로 대부분의 brute force 공격을 막을 수 있습니다. <br />
기본 설정에는 Apache HTTP Server, Lighttpd, sshd, vsftpd, qmail, Postfix 그리고 Courier 메일 서버를 위한 필터가 제공됩니다. <br />
필터는 python 정규식으로 정의되며, 관리자는 필터를 커스터마이징 하여 더 강력한 보안 설정을 할 수 있습니다.


## 참고(REFERENCES)
- [https://ko.wikipedia.org/wiki/Fail2ban](https://ko.wikipedia.org/wiki/Fail2ban){: target="\_blank"}
- [https://www.fail2ban.org/](https://www.fail2ban.org/){: target="\_blank"}
- [http://www.fail2ban.org/wiki/index.php/Fail2Ban](http://www.fail2ban.org/wiki/index.php/Fail2Ban){: target="\_blank"}
