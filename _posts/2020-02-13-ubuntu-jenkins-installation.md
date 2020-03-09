---
title: "우분투(Ubuntu) 환경에 패키지로 젠킨스(Jenkins) 설치하기"
categories: 
  - jenkins
tags: 
  - jenkins
  - ubuntu
---


젠킨스(이하 Jenkins)는 MIT license로 오픈소스 소프트웨어이며, 소프트웨어 개발 시 지속적 통합(CI, Continuous Integration) 서비스를 제공하는 자동화 서버입니다. <br />
Jenkins는 모든 프로젝트의 빌드와 배포, 자동화를 지원하며, 수백 개의 플러그인을 제공합니다. <br />
이 포스트에서는 우분투(이하 Ubuntu) 환경에서 패키지로 Jenkins를 설치하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- Ubuntu 환경에 Java가 설치되어 있어야 합니다.
    + Jenkins 버전에 따른 필요 Java 버전
        - 2.164 (2019-02) and newer: Java 8 or Java 11
        - 2.54 (2017-04) and newer: Java 8
        - 1.612 (2015-05) and newer: Java 7
- 방화벽 설정이 필요합니다.
    + TCP 8080 포트가 개방되어 있어야 합니다. 

> Java 설치 방법은 [우분투(Ubuntu) 환경에 OpenJDK(Java) 설치하기](https://lindarex.github.io/ubuntu/ubuntu-openjdk-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

> Java 버전에 대한 자세한 정보는 [https://jenkins.io/doc/administration/requirements/java/](https://jenkins.io/doc/administration/requirements/java/){: target="\_blank"}를 확인해 주시기 바랍니다.

> 방화벽 설정 방법은 [우분투(Ubuntu) 환경에 방화벽(Firewalld) 설치 및 설정하기](https://lindarex.github.io/ubuntu/ubuntu-firewalld-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.1 build-15018445)
- Ubuntu 16.04.4 LTS (Xenial Xerus) Server (64-bit)
- Jenkins 2.121.1
- OpenJDK 1.8.0_171


## 요약(SUMMARY)
1. Jenkins debian packages repository 설정
2. apt 명령어로 Jenkins 설치
3. (선택사항) Java 경로 설정 및 Jenkins의 기본 포트 변경
4. systemctl 명령어로 Jenkins 실행
5. 웹브라우저로 Jenkins 접속


## 내용(CONTENTS)
### 1. Jenkins debian packages repository key 추가
```console
$ sudo wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
```

### 2. Jenkins debian packages repository 추가
```console
$ sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'  
```

### 3. apt install 명령어로 Jenkins 설치
```console
$ sudo apt update -y && sudo apt install jenkins -y  
```

### 4. (선택사항) Java 경로 설정
#### 4.1. 설치된 Java 경로 확인
```console
$ sudo which java
/usr/bin/java
```

#### 4.2. Java 경로 추가
```console
$ sudo vi /etc/init.d/jenkins
```

```shell
--------------------------------------------------------------------------------
PATH="/bin:/usr/bin:/sbin:/usr/sbin:/usr/bin/java" 
--------------------------------------------------------------------------------
```

### 5. (선택사항) 기본 포트 변경
```console
$ sudo vi /etc/default/jenkins  
```

```shell
--------------------------------------------------------------------------------
JENKINS_PORT="8080"  
JENKINS_AJP_PORT="9091"  
JENKINS_USER="root"  
--------------------------------------------------------------------------------
```

### 6. systemctl 명령어로 Jenkins 서비스 관리
#### 6.1. Jenkins 서비스 설정 반영
```console
$ sudo systemctl daemon-reload
```

#### 6.2. Jenkins 서비스 시작
```console
$ sudo systemctl start jenkins.service
```

#### 6.3. Jenkins 서비스 중지
```console
$ sudo systemctl stop jenkins.service
```

#### 6.4. Jenkins 서비스 재시작
```console
$ sudo systemctl restart jenkins.service
```

#### 6.5. Jenkins 서비스 설정 재적용
```console
$ sudo systemctl reload jenkins.service
```

#### 6.6. Jenkins 서비스 상태 조회
```console
$ sudo systemctl status jenkins.service
```

#### 6.7. Jenkins 서비스 활성화(부팅 시 자동 시작)
```console
$ sudo systemctl enable jenkins.service
```

#### 6.8. Jenkins 서비스 비활성화
```console
$ sudo systemctl disable jenkins.service
```

#### 6.9. Jenkins 서비스 및 관련 프로세스 모두 중지
```console
$ sudo systemctl kill jenkins.service
```

### 7. 웹브라우저로 Jenkins 접속
- http://[MY-IP]:8080


## 마무리(CONCLUSION)
Ubuntu 환경에 패키지로 Jenkins 설치를 완료했습니다. <br />
Jenkins 초기 설정은 [Jenkins 초기 설정하기](https://lindarex.github.io/jenkins/jenkins-initial-setting/){: target="\_blank"} 포스트를 참고하시기 바랍니다. <br />
다음 포스트에서는 Ubuntu 환경에서 war 파일을 내려받아 java 명령어로 Jenkins를 실행하는 방법을 소개하겠습니다.


## 참고(REFERENCES)
- [https://jenkins.io/](https://jenkins.io/){: target="\_blank"}
- [http://pkg.jenkins-ci.org/debian-stable/](http://pkg.jenkins-ci.org/debian-stable/){: target="\_blank"}
