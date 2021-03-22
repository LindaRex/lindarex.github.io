---
title: "우분투(Ubuntu) 환경에 SCM Manager v2 설치하기"
categories: 
  - scm-manager
tags: 
  - "scm manager"
  - "v2"
  - "version control"
  - "source code management"
  - "open source software"
  - "open source"
  - oss
  - ubuntu
---


SCM Manager v2(이하 scm manager)는 개별적으로 확장 가능한 가벼운 소스 코드 관리 도구로, MIT 라이선스(license)가 적용된 오픈소스(open source) 소프트웨어입니다.
<br /><br />
scm manager는 다양한 플러그인과 Level 3 RESTful WebService를 제공하고, Git과 Mercurial, Subversion 저장소(repository)를 지원합니다.
<br /><br />
scm manager는 웹 서버(web server), 데이터베이스(database) 등의 종속성이 없고, Ubuntu/CentOS/Fedora/Windows/MacOS X/Docker/Kubernetes 환경에 편리한 설치 가이드를 제공합니다.
<br /><br />
이 포스트에서는 우분투(ubuntu) 환경에서 패키지(package)로 scm manager를 설치하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- 방화벽 설정이 필요합니다.
    + TCP 8080 포트가 개방되어 있어야 합니다.

> 방화벽 설정 방법은 [우분투(Ubuntu) 환경에 방화벽(Firewalld) 설치 및 설정하기](https://lindarex.github.io/ubuntu/ubuntu-firewalld-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- VULTR High Frequency Cloud Compute (256 GB NVMe, 3 CPU, 8192MB Memory, 4000GB Bandwidth)
- Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-136-generic x86_64)
- SCM Manager 2.14.1


## 요약(SUMMARY)
1. scm manager debian packages repository 설정
2. apt install 명령어로 scm manager 설치
3. (선택사항) scm manager 설정
4. systemctl 명령어로 scm manager 관리
5. 웹브라우저로 scm manager 접속


## 내용(CONTENTS)
### 1. scm manager debian packages repository key 추가
```console
$ sudo apt-key adv --recv-keys --keyserver hkps://keys.openpgp.org 0x975922F193B07D6E
Executing: /tmp/apt-key-gpghome.mDvF5MxIa1/gpg.1.sh --recv-keys --keyserver hkps://keys.openpgp.org 0x975922F193B07D6E
gpg: key 975922F193B07D6E: "SCM Packages (signing key for packages.scm-manager.org) <scm-team@cloudogu.com>" not changed
gpg: Total number processed: 1
gpg:              unchanged: 1
```

### 2. scm manager debian packages repository 추가
```console
$ echo 'deb [arch=all] https://packages.scm-manager.org/repository/apt-v2-releases/ stable main' | sudo tee /etc/apt/sources.list.d/scm-manager.list
```

### 3. apt install 명령어로 scm manager 설치
#### 3.1. package 업데이트 
```console
$ sudo apt-get update
```

#### 3.2. scm manager 설치
```console
$ sudo apt-get install scm-server -y
```

### 4. (선택사항) scm manager 설정

- SCM Manager UI의 포트를 설정합니다.

```console
$ sudo vi /etc/default/scm-server
```

```shell
----------------------------------------------------------------------------------------------------
PORT=8080
----------------------------------------------------------------------------------------------------
```

### 5. systemctl 명령어로 scm manager 관리
#### 5.1. scm manager 설정 반영
```console
$ sudo systemctl daemon-reload
```

#### 5.2. scm manager 시작
```console
$ sudo systemctl start scm-server
```

#### 5.3. scm manager 중지
```console
$ sudo systemctl stop scm-server
```

#### 5.4. scm manager 재시작
```console
$ sudo systemctl restart scm-server
```

#### 5.5. scm manager 설정 재적용
```console
$ sudo systemctl reload scm-server
```

#### 5.6. scm manager 상태 조회
```console
$ sudo systemctl status scm-server
```

#### 5.7. scm manager 활성화(부팅 시 자동 시작)
```console
$ sudo systemctl enable scm-server
```

#### 5.8. scm manager 비활성화
```console
$ sudo systemctl disable scm-server
```

#### 5.9. scm manager 및 관련 프로세스 모두 중지
```console
$ sudo systemctl kill scm-server
```

### 6. 웹브라우저로 scm manager 접속
- http://[MY-IP]:8080/scm

> SCM Manager의 기본 계정은 'scmadmin', 비밀번호도 'scmadmin' 입니다.


## 마무리(CONCLUSION)
ubuntu 환경에 package로 scm manager 설치를 완료했습니다.
<br /><br />
SCM Manager의 자세한 내용은 아래 페이지를 확인해 주시기 바랍니다.


## 참고(REFERENCES)
- [https://www.scm-manager.org/](https://www.scm-manager.org/){: target="\_blank"}
