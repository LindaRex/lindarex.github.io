---
title: "우분투(Ubuntu) 서버(Server) 초기 설정하기"
categories: 
  - ubuntu
tags: 
  - ubuntu
---


이 포스트에서는 우분투(ubuntu) 서버(server) 설치 후 초기 설정하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- ubuntu 환경이 필요합니다.

> Ubuntu 설치 방법은 [VMware workstation에 ubuntu server 16.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1604-installation/){: target="\_blank"} 또는 [VMware workstation에 ubuntu server 18.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1804-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.1 build-15018445)
- Ubuntu 18.04.4 LTS (Bionic Beaver) Server (64-bit)


## 요약(SUMMARY)
1. 리눅스 버전 조회
2. 관리자(root) 계정 활성화
3. 패키지 업데이트
4. 유용한 패키지 설치
5. 사용자 계정 생성
6. 시스템 언어 설정
7. 시스템 시간 설정


## 내용(CONTENTS)
- 아래 설정은 ubuntu server 16.04, ubuntu server 18.04에 모두 동일하게 적용됩니다.

### 1. 리눅스(linux) 버전 조회

- 아래 예제는 buntu 18.04의 정보입니다.

```console
$ grep . /etc/*-release
/etc/lsb-release:DISTRIB_ID=Ubuntu
/etc/lsb-release:DISTRIB_RELEASE=18.04
/etc/lsb-release:DISTRIB_CODENAME=bionic
/etc/lsb-release:DISTRIB_DESCRIPTION="Ubuntu 18.04.4 LTS"
/etc/os-release:NAME="Ubuntu"
/etc/os-release:VERSION="18.04.4 LTS (Bionic Beaver)"
/etc/os-release:ID=ubuntu
/etc/os-release:ID_LIKE=debian
/etc/os-release:PRETTY_NAME="Ubuntu 18.04.4 LTS"
/etc/os-release:VERSION_ID="18.04"
/etc/os-release:HOME_URL="https://www.ubuntu.com/"
/etc/os-release:SUPPORT_URL="https://help.ubuntu.com/"
/etc/os-release:BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
/etc/os-release:PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
/etc/os-release:VERSION_CODENAME=bionic
/etc/os-release:UBUNTU_CODENAME=bionic
```
​
- 아래 예제는 buntu 16.04의 정보입니다.

```console
$ grep . /etc/*-release
/etc/lsb-release:DISTRIB_ID=Ubuntu
/etc/lsb-release:DISTRIB_RELEASE=16.04
/etc/lsb-release:DISTRIB_CODENAME=xenial
/etc/lsb-release:DISTRIB_DESCRIPTION="Ubuntu 16.04.5 LTS"
/etc/os-release:NAME="Ubuntu"
/etc/os-release:VERSION="16.04.5 LTS (Xenial Xerus)"
/etc/os-release:ID=ubuntu
/etc/os-release:ID_LIKE=debian
/etc/os-release:PRETTY_NAME="Ubuntu 16.04.5 LTS"
/etc/os-release:VERSION_ID="16.04"
/etc/os-release:HOME_URL="http://www.ubuntu.com/"
/etc/os-release:SUPPORT_URL="http://help.ubuntu.com/"
/etc/os-release:BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
/etc/os-release:VERSION_CODENAME=xenial
/etc/os-release:UBUNTU_CODENAME=xenial
```

### 2. 관리자(root) 계정 활성화
- ubuntu를 비롯한 linux를 설치하면 기본적으로 관리자(root) 계정이 생성됩니다. root 계정은 비밀번호(password)를 설정하기 전까지 비활성화 상태이기 때문에, password를 설정하여 root 계정을 활성화합니다.

#### 2.1. root 계정 password 설정
- 'passwd' 명령어(command)로 root password를 설정합니다.
- password를 2번 동일하게 입력하면 아래와 같이 password가 갱신되었다는 메시지를 확인할 수 있습니다.

```console
$ sudo passwd root
[sudo] password for rex:
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
```

#### 2.2. root 계정 활성화 확인
- 'su -' command를 사용하여 root 계정으로 사용자 계정 전환에 성공하면, root 계정이 정상적으로 활성화된 것입니다.

```console
rex@lindarex:~$ su - root
Password:
root@lindarex:~#
```

- 'exit' 또는 'logout' command로 root 계정에서 로그아웃(logout)을 할 수 있습니다.

```console
root@lindarex:~# exit
logout
rex@lindarex:~$
```

> 'sudo'(substitute user do)는 현재 계정에서 다른 계정, 즉 슈퍼 유저(superuser)로서 관리자(root) 권한을 가진 계정으로 프로그램(program)을 구동할 수 있도록 하는 command이며, 'sudo -i'는 root 계정으로 로그인(login)하며 '/root' 디렉터리(directory)로 이동하는 command이고, 'sudo -s'는 현재 directory를 유지하며 root 계정으로 login 하는 command입니다. 'sudo', 'sudo -i', 'sudo -s' command는 현재 계정의 password를 요구하고, 전환된 계정의 환경변수는 적용하지 않습니다.

> 'su'(substitute user, switch user)는 현재 계정에서 log out 하지 않고 다른 계정(기본값은 root 계정)으로 전환하며 현재 계정의 환경변수를 유지하는 command이고, 'su -'는 'su'와 동일하게 계정을 전환하며, 전환된 계정의 환경변수를 적용하는 command입니다. 'su', 'su -' command는 전환될 계정의 password를 요구합니다.

> 'sudo' command는 현재 계정에서 root 계정의 권한만 빌리기 때문에 작업 내역은 현재 계정으로 남고, 'su' command는 현재 계정에서 root 계정으로 전환하기 때문에 작업 내역은 root 계정으로 저장됩니다.

> 'sudo' command에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/Sudo](https://ko.wikipedia.org/wiki/Sudo){: target="\_blank"}와 [https://zetawiki.com/wiki/리눅스_sudo\,_su_차이점](https://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_sudo,_su_%EC%B0%A8%EC%9D%B4%EC%A0%90){: target="\_blank"}을 확인해 주시기 바랍니다.

### 3. 패키지(package) 업데이트(update)
- 'apt update' command로 package 인덱스를 update합니다.

```console
$ sudo apt update
[sudo] password for rex:
```

- 'apt upgrade' command로 업그레이드 가능한 모든 package를 update합니다.

```console
$ sudo apt upgrade -y
[sudo] password for rex:
```

- ubuntu를 재시작합니다.

```console
$ sudo reboot
[sudo] password for rex:
```

### 4. 유용한 package 설치

- 'apt install' command로 arp, ifconfig, netstat,  rarp 등 네트워크 제어 command를 포함한 net-tools package를 설치합니다.

```console
$ sudo apt install net-tools -y
[sudo] password for rex:
```

- 'apt install' command로 압축 program인 unzip package를 설치합니다.

```console
$ sudo apt install unzip -y
[sudo] password for rex:
```

- 'apt install' command로 tree 구조로 디렉터리를 조회할 수 있는 tree package를 설치합니다.

```console
$ sudo apt install tree -y
[sudo] password for rex:
```

### 5. 사용자(user) 계정 생성
- 'adduser' command로 user 계정을 추가합니다.

```console
$ sudo adduser rex2
[sudo] password for rex:
```

- root 계정의 password 설정 시와 동일하게, 'passwd' command로 user 계정의 password를 설정합니다.

```console
$ sudo passwd rex2
[sudo] password for rex:
```

​- 'visudo' command로 생성한 user 계정에 root(sudo) 권한을 추가합니다.

> '/etc/sudoers'를 vi로 수정할 수도 있지만, 설정 유효성과 문법 체크를 위해 'visudo' command로 수정하시기 바랍니다.

```console
$ sudo visudo
[sudo] password for rex:
```

```shell
----------------------------------------------------------------------------------------------------
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root    ALL=(ALL:ALL) ALL
rex     ALL=(ALL:ALL) ALL
rex2    ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d
----------------------------------------------------------------------------------------------------
```

​- 'visudo' command로 생성한 user 계정에 root(sudo) 권한 추가와 함께 password 입력 없이 사용할 때는 아래와 같이 설정합니다.

> sudo password 입력 없이 사용하면, 보안상 안전하지 않을 수 있으니 주의하시기 바랍니다.

```console
$ sudo visudo
[sudo] password for rex:
```

```shell
----------------------------------------------------------------------------------------------------
~
# User privilege specification
root    ALL=(ALL:ALL) ALL
rex     ALL=NOPASSWD:ALL
rex2    ALL=NOPASSWD:ALL
~
----------------------------------------------------------------------------------------------------
```

### 6. 시스템(system) 언어(locale) 설정

- 'locale' command로 현재 locale 설정을 조회합니다.

```console
$ locale
LANG=en_US.UTF-8
LANGUAGE=
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
```

- 'locale -a' command로 설치되어 있는 locale을 조회합니다.

```console
$ locale -a
C
C.UTF-8
en_US.utf8
POSIX
```

- 'apt install' command로 한국어(Korean) package를 설치합니다.

```console
$ sudo apt install language-pack-ko -y
[sudo] password for rex:
```

- Korean 설정을 합니다.

```console
$ sudo vi /etc/default/locale
[sudo] password for rex:
```

```shell
----------------------------------------------------------------------------------------------------
LANG="ko_KR.UTF-8"
LANGUAGE="ko_KR:ko:en_US:en"
----------------------------------------------------------------------------------------------------
```

- 설정 후 재접속하여 한글 출력을 확인합니다.

```console
$ sudo ls
[sudo] rex의 암호:
죄송합니다만, 다시 시도하십시오.
[sudo] rex의 암호:
```
​
- 'locale -a' command로 설치된 Korean locale을 조회합니다.

```console
$ locale -a
C
C.UTF-8
en_US.utf8
ko_KR.utf8
POSIX
```

### 7. 시스템(system) 시간(timezone) 설정

- 'date' command로 현재 시각을 조회합니다.

```console
$ date
Mon Mar 14 07:29:26 UTC 2020
```

- 'timedatectl' command로 자세한 timezone 설정을 조회합니다.

```console
$ timedatectl
                      Local time: Mon 2020-03-14 07:26:52 UTC
                  Universal time: Mon 2020-03-14 07:26:52 UTC
                        RTC time: Mon 2020-03-14 07:26:53
                       Time zone: Etc/UTC (UTC, +0000)
       System clock synchronized: yes
systemd-timesyncd.service active: yes
                 RTC in local TZ: no
```

- 'timedatectl list-timezones' command로 사용 가능한 timezone 목록을 조회합니다.

```console
$ timedatectl list-timezones
Africa/Abidjan
Africa/Accra
Africa/Addis_Ababa
~
Pacific/Tongatapu
Pacific/Wake
Pacific/Wallis
UTC
```

- 'timedatectl set-timezone' command로 'Asia/Seoul' timezone으로 설정합니다.

```console
$ sudo timedatectl set-timezone Asia/Seoul
```

- 'date' command로 설정한 timezone이 적용된 현재 시각을 조회합니다.

```console
$ date
Mon Mar 14 16:35:27 KST 2020
```

- 'timedatectl' command로 'Asia/Seoul'으로 설정한 timezone 설정을 조회합니다.

```console
$ timedatectl
                      Local time: Mon 2020-03-14 16:35:29 KST
                  Universal time: Mon 2020-03-14 07:35:29 UTC
                        RTC time: Mon 2020-03-14 07:35:30
                       Time zone: Asia/Seoul (KST, +0900)
       System clock synchronized: yes
systemd-timesyncd.service active: yes
                 RTC in local TZ: no
```


## 마무리(CONCLUSION)
ubuntu server 설치 후 초기 설정을 완료했습니다. <br />
다음 포스트에서는 ubuntu 보안 강화를 위한 오픈소스(open source) 소프트웨어인 fail2ban을 소개하겠습니다.


## 참고(REFERENCES)
- [https://ubuntu.com/](https://ubuntu.com/){: target="\_blank"}
- [https://ubuntu.com/tutorials/tutorial-install-ubuntu-server#1-overview](https://ubuntu.com/tutorials/tutorial-install-ubuntu-server#1-overview){: target="\_blank"}
