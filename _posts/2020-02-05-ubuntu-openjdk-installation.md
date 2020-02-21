---
title: "우분투(Ubuntu) 환경에 패키지로 OpenJDK(Java) 설치하기"
categories: 
  - ubuntu
tags: 
  - openjdk
  - java
  - ubuntu
---


OpenJDK는 Java 애플리케이션 구축을 위한 오픈 소스 기반의 JDK(Java Development Kit)입니다. <br />
JDK는 JVM(Java Virtual Machine), JRE(Java Runtime Environment)와 함께 Java 프로그래밍에 필요한 핵심 기술 패키지입니다. <br />
JDK는 2개로 나뉘는데, 하나는 BCL(Oracle Binary Code License) 라이선스의 Oracle JDK, 하나는 GNU GPL v2(GNU General Public License) 라이선스의 OpenJDK입니다. <br />
이 포스트에서는 우분투(이하 Ubuntu) 환경에서 패키지로 OpenJDK를 설치하는 방법을 소개합니다.


> Oracle JDK와 OpenJDK에 대한 자세한 정보는 [Oracle JDK와 OpenJDK의 차이점](https://lindarex.github.io/concepts/difference-between-oraclejdk-openjdk/){: target="_blank"} 포스트를 참고하시기 바랍니다.


## 선행조건(PREREQUISITE)
- Ubuntu 환경이 필요합니다.

> Ubuntu 설치 방법은 [VMware workstation에 Ubuntu 16.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1604-installation/){: target="_blank"} 또는 [VMware workstation에 Ubuntu 18.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1804-installation/){: target="_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.1 build-15018445)
- Ubuntu 16.04.4 LTS (Xenial Xerus)
- OpenJDK 1.8.0_222


## 요약(SUMMARY)
1. apt 명령어로 OpenJDK 설치
2. OpenJDK 설치 확인
3. Ubuntu 환경변수에 OpenJDK Java 경로 설정
4. (선택사항) apt 명령어로 OpenJDK 삭제


## 내용(CONTENTS)
### 1. apt 명령어로 OpenJDK 설치
```shell
$ sudo apt update -y && sudo apt install openjdk-8-jdk -y
```

### 2. OpenJDK 설치 확인
```shell
$ java -version
openjdk version "1.8.0_222"
OpenJDK Runtime Environment (build 1.8.0_222-8u222-b10-1ubuntu1~18.04.1-b10)
OpenJDK 64-Bit Server VM (build 25.222-b10, mixed mode)
```

### 3. Ubuntu 환경변수에 OpenJDK Java 경로 설정
#### 3.1. 설치된 Java 경로 확인
```shell
$ which java
/usr/bin/java
```

#### 3.2. OpenJDK Java 경로 추가
> 루트(root) 계정

```shell
$ sudo vi /etc/profile
or
# vi $HOME/.bashrc
```

> 일반 계정

```shell
$ vi $HOME/.profile
```

```shell
$ sudo vi /etc/profile
----------------------------------------------------------------------------------------------------
JAVA_HOME=$(dirname $(dirname $(update-alternatives --list javac)))
JAVA=${JAVA_HOME}/bin
CLASSPATH=.:${JAVA_HOME}/lib/tools.jar
PATH=${PATH}:${JAVA}

export JAVA_HOME JAVA CLASSPATH PATH
----------------------------------------------------------------------------------------------------
```

> 수정 내역 적용을 위해 아래 명령어를 입력합니다.
```shell
$ source /etc/profile
```

### 4. (선택사항) apt 명령어로 OpenJDK 삭제
- '--auto-remove' 옵션을 추가하면, 사용하지 않는 관련 패키지를 모두 삭제합니다.

#### 4.1. apt remove 명령어로 OpenJDK 삭제
- 설정 파일을 유지하며 OpenJDK를 삭제합니다.

```shell
$ sudo apt remove openjdk*
$ sudo apt remove --auto-remove openjdk*
```

#### 4.2. apt purge 명령어로 OpenJDK 삭제
- 설정 파일과 함께 OpenJDK를 삭제합니다. (단, 사용자 홈 디렉터리의 설정 파일은 유지됩니다.)

```shell
$ sudo apt purge openjdk*
$ sudo apt purge --auto-remove openjdk*
```

## 마무리(CONCLUSION)
Ubuntu 환경에 패키지로 OpenJDK 설치를 완료했습니다. <br />
JDK는 Java 프로그래밍에 필수 요소이며, 상당수의 오픈소스 소프트웨어와 국내 SI(System Integration) 프로젝트에서 JDK를 요구합니다. <br />
정부 발주 프로젝트에 다수를 차지하는 SI 회사가 Java + 스프링 프레임워크(Spring Framework) 또는 Java + 전자정부표준프레임워크(약칭 eGov) 구성을 사용하기 때문에 Java의 인기는 시들지 않고 있습니다. <br />
Java는 플랫폼에 독립적이고 수많은 개발자와 레퍼런스를 보유하고 있다는 장점과 속도 문제라는 단점을 가진 언어입니다. <br />
국내 현업에서는 Java를 비롯한 여러 가지 언어가 자주 사용되므로 개발 환경이나 작업 특성에 따라 적합한 언어를 선택할 수 있는 지식과 노하우가 필요합니다.


## 참고(REFERENCES)
- [https://openjdk.java.net/](https://openjdk.java.net/){: target="_blank"}
- [https://ko.wikipedia.org/wiki/OpenJDK](https://ko.wikipedia.org/wiki/OpenJDK){: target="_blank"}
- [https://namu.wiki/w/Java](https://namu.wiki/w/Java){: target="_blank"}
