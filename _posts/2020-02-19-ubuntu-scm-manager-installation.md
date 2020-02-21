---
title: "우분투(Ubuntu) 환경에 SCM Manager 설치하기"
categories: 
  - scm-manager
tags: 
  - "scm manager"
  - ubuntu
---


SCM Manager는 사설 버전관리 레파지토리(Private source code and version control repository)로 BSD 라이선스가 적용된 오픈소스 소프트웨어입니다. <br />
SCM Manager는 Git과 Mercurial, Subversion을 지원하고, 다양한 플러그인과 RESTFul API를 제공합니다. <br />
SCM Manager는 크게 Standalone 방식(tar, RPM, DEB package)과 WebApp(war) 방식을 제공하는데, 이 포스트에서는 우분투(이하 Ubuntu) 환경에서 Standalone 방식으로 SCM Manager를 설치하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- Ubuntu 환경에 Java가 설치되어 있어야 합니다.

> Java 설치 방법은 [우분투(Ubuntu) 환경에 OpenJDK(Java) 설치하기](https://lindarex.github.io/ubuntu/ubuntu-openjdk-installation/){: target="_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.1 build-15018445)
- Ubuntu 18.04.3 LTS (Bionic Beaver) Server (64-bit)
- SCM Manager 1.6.0
- OpenJDK 1.8.0_222


## 요약(SUMMARY)
1. SCM Manager 파일 내려받기
2. SCM Manager 설정
3. 스크립트로 SCM Manager 살행
4. 웹브라우저로 SCM Manager 접속


## 내용(CONTENTS)

> 아래 명령어로 workspace 디렉터리를 생성합니다.

```shell
$ export LINDAREX_WORKSPACE=${HOME}/workspace
$ mkdir -p ${LINDAREX_WORKSPACE}
```

### 1. SCM Manager 파일 내려받기
```shell
$ wget -P ${LINDAREX_WORKSPACE} https://maven.scm-manager.org/nexus/content/repositories/releases/sonia/scm//scm-server/1.60/scm-server-1.60-app.tar.gz
```

### 2. 내려받은 파일 압축 해제하기
```shell
$ tar zxf ${LINDAREX_WORKSPACE}/scm-server-1.60-app.tar.gz -C ${LINDAREX_WORKSPACE}
```

### 3. SCM Manager 설정

- SCM Manager UI의 포트를 설정합니다.

```shell
$ vi ${LINDAREX_WORKSPACE}/scm-server/conf/server-config.xml
--------------------------------------------------------------------------------
<SystemProperty name="jetty.port" default="8080" />
--------------------------------------------------------------------------------
```

### 4. SCM Manager 실행
```shell
$ cd ${LINDAREX_WORKSPACE}/scm-server/bin/
$ nohup ./scm-server > /dev/null &
```

### 5. 웹브라우저로 SCM Manager 접속
- http://[MY-IP]:8080


## 마무리(CONCLUSION)
Ubuntu 환경에 Standalone 방식으로 SCM Manager 설치를 완료했습니다. <br />
다음 포스트에서는 SCM Manager 사용 방법을 소개하겠습니다.


## 참고(REFERENCES)
- [https://www.scm-manager.org/download/](https://www.scm-manager.org/download/){: target="_blank"}
