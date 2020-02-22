---
title: "우분투(Ubuntu) 환경에 AWS CLI 설치하기"
categories: 
  - aws
tags: 
  - aws
  - cli
  - ubuntu
---


AWS CLI(명령줄 인터페이스, command line interface)는 AWS 서비스를 관리하는 통합 도구입니다. <br />
Python 2.6.5 이상이 필요하며, pip을 사용하여 AWS CLI를 설치합니다. <br />
이 포스트에서는 우분투(이하 Ubuntu) 환경에서 AWS를 사용하기 위한 AWS CLI를 설치하는 방법을 소개합니다.

> pip은 파이썬(Python)으로 작성된 패키지 소프트웨어를 설치 및 관리하는 패키지 관리 시스템입니다.


## 선행조건(PREREQUISITE)
- Ubuntu 환경이 필요합니다.

> Ubuntu 설치 방법은 [VMware workstation에 Ubuntu 16.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1604-installation/){: target="_blank"} 또는 [VMware workstation에 Ubuntu 18.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1804-installation/){: target="_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.1 build-15018445)
- Ubuntu 18.04.3 LTS (Bionic Beaver) Server (64-bit)
- aws-cli v1.16.261
- Python v2.7.17


## 요약(SUMMARY)
1. apt 명령어로 pip 설치
2. pip 명령어로 AWS CLI 설치


## 내용(CONTENTS)
### 1. apt 명령어로 pip 설치
```shell
$ sudo apt install python-setuptools python-pip -y
```

### 2. pip 명령어로 AWS CLI 설치
```shell
$ pip install awscli
```

> AWS CLI 설치 후 세션 재접속이 필요합니다.


## 마무리(CONCLUSION)
Ubuntu 환경에 AWS CLI 설치를 완료했습니다. <br />
다음 포스트에서는 AWS S3 사용 방법을 소개하겠습니다.


## 참고(REFERENCES)
- [https://aws.amazon.com/ko/cli/](https://aws.amazon.com/ko/cli/){: target="_blank"}
