---
title: "우분투(Ubuntu) 환경에 SCM Manager 설치하기"
categories: 
  - scm-manager
tags: 
  - "scm manager"
  - "version control"
  - "open source software"
  - "open source"
  - oss
  - ubuntu
---


SCM Manager는 사설 버전 관리 레파지토리(private source code and version control repository)로 BSD(berkeley software distribution) 라이선스(license)가 적용된 오픈소스(open source) 소프트웨어입니다.
<br /><br />
SCM Manager는 Git과 Mercurial, Subversion을 지원하고, 다양한 플러그인과 RESTFul API를 제공하며, 설치 방법으로는 Standalone 방식(tar, RPM, DEB package)과 WebApp(war) 방식을 제공합니다.
<br /><br />
이 포스트에서는 우분투(ubuntu) 환경에서 standalone 방식으로 SCM Manager를 설치하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- ubuntu 환경에 Java가 설치되어 있어야 합니다.
- 방화벽 설정이 필요합니다.
    + TCP 8080 포트가 개방되어 있어야 합니다.

> Java 설치 방법은 [우분투(Ubuntu) 환경에 OpenJDK(Java) 설치하기](https://lindarex.github.io/ubuntu/ubuntu-openjdk-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

> 방화벽 설정 방법은 [우분투(Ubuntu) 환경에 방화벽(Firewalld) 설치 및 설정하기](https://lindarex.github.io/ubuntu/ubuntu-firewalld-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


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

```console
$ export LINDAREX_WORKSPACE=${HOME}/workspace
$ mkdir -p ${LINDAREX_WORKSPACE}
```

### 1. SCM Manager 파일 내려받기
```console
$ wget -P ${LINDAREX_WORKSPACE} https://maven.scm-manager.org/nexus/content/repositories/releases/sonia/scm//scm-server/1.60/scm-server-1.60-app.tar.gz
```

### 2. 내려받은 파일 압축 해제
```console
$ tar zxf ${LINDAREX_WORKSPACE}/scm-server-1.60-app.tar.gz -C ${LINDAREX_WORKSPACE}
```

### 3. SCM Manager 설정

- SCM Manager UI의 포트를 설정합니다.

```console
$ vi ${LINDAREX_WORKSPACE}/scm-server/conf/server-config.xml
```

```shell
--------------------------------------------------------------------------------
<SystemProperty name="jetty.port" default="8080" />
--------------------------------------------------------------------------------
```

### 4. SCM Manager 실행
```console
$ cd ${LINDAREX_WORKSPACE}/scm-server/bin/
$ nohup ./scm-server > /dev/null &
```

### 5. 웹브라우저로 SCM Manager 접속
- http://[MY-IP]:8080

> SCM Manager의 기본 계정은 'scmadmin', 비밀번호도 'scmadmin' 입니다.


## 마무리(CONCLUSION)
ubuntu 환경에 standalone 방식으로 SCM Manager 설치를 완료했습니다.
<br /><br />
SCM Manager 외에도 GitHub, GitLab, Bitbucket, GitStack 등 다양한 private git 호스팅 서비스(service)가 존재합니다.
<br />
타 service와 비교 시, SCM Manager의 장점은 간단히 설치하여 사용 가능하고, 설정하기 쉽다는 것입니다.
<br />
또한, git 외에도 svn과 mercurial 서버(server)까지 동시에 사용할 수 있으며, 기존에 사용 중이던 svn server를 그대로 이용할 수 있습니다.
<br /><br />
더 자세한 내용은 아래 참고 페이지를 확인해 주시기 바랍니다.


## 참고(REFERENCES)
- [https://www.scm-manager.org/](https://www.scm-manager.org/){: target="\_blank"}
- [https://www.scm-manager.org/download/](https://www.scm-manager.org/download/){: target="\_blank"}
