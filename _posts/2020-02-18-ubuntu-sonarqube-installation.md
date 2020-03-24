---
title: "우분투(Ubuntu) 환경에 SonarQube 설치하기"
categories: 
  - sonarqube
tags: 
  - sonarqube
  - ubuntu
---


SonarQube는 정적 코드 분석기로 20개 이상의 프로그래밍 언어의 버그와 Code smells, 보안 취약점을 발견하고 자동 리뷰를 수행하여 지속적인 코드 품질 검사를 위한 플랫폼입니다. <br />
SonarQube는 LGPL(lesser gnu general public license)로 오픈소스(open source) 소프트웨어이며, 코드 커버리지, 유닛 테스트, 코딩 표준, 중복 코드, 코드 복잡도, 주석, 버그 및 보안 취약점의 보고서를 제공합니다. <br />
또한 Maven, Ant, Gradle, MSBuild 및 CI(Continuous Integration) 도구인 Atlassian Bamboo, Jenkins, Hudson 등과의 연동을 제공합니다. <br />
이 포스트에서는 우분투(ubuntu) 환경에서 SonarQube를 설치하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- ubuntu 환경에 PostgreSQL이 설치되어 있어야 합니다.
- 방화벽 설정이 필요합니다.
    + TCP 9000 포트가 개방되어 있어야 합니다.

> PostgreSQL 설치 방법은 [우분투(Ubuntu) 환경에 PostgreSQL 설치하기](https://lindarex.github.io/postgresql/ubuntu-postgresql-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

> 방화벽 설정 방법은 [우분투(Ubuntu) 환경에 방화벽(Firewalld) 설치 및 설정하기](https://lindarex.github.io/ubuntu/ubuntu-firewalld-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.1 build-15018445)
- Ubuntu 18.04.3 LTS (Bionic Beaver) Server (64-bit)
- SonarQube 7.9.1
- PostgreSQL 11.5 (Ubuntu 11.5-3.pgdg18.04+1)
- OpenJDK 1.8.0_222


## 요약(SUMMARY)
1. SonarQube 파일 내려받기
2. PostgreSQL 설정
3. SonarQube 설정
4. 스크립트로 SonarQube 서비스 관리
5. 웹브라우저로 SonarQube 접속


## 내용(CONTENTS)

> 아래 명령어로 workspace 디렉터리를 생성합니다.

```console
$ export LINDAREX_WORKSPACE=${HOME}/workspace
$ mkdir -p ${LINDAREX_WORKSPACE}
```

### 1. SonarQube 파일 내려받기
```console
$ wget -P ${LINDAREX_WORKSPACE} https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.9.1.zip
```

### 2. 내려받은 파일 압축 해제
```console
$ unzip -q ${LINDAREX_WORKSPACE}/sonarqube-7.9.1.zip -d ${LINDAREX_WORKSPACE}
```

### 3. Symbolic link 설정
```console
$ ln -s ${LINDAREX_WORKSPACE}/sonarqube-7.9.1 ${LINDAREX_WORKSPACE}/sonarqube
```

### 4. PostgreSQL 설정
- SonarQube와 연동 될 사용자 계정과 데이터베이스(Database)를 생성합니다.

#### 4.1. postgres 계정 로그인
```console
$ sudo su - postgres
```

#### 4.2. psql utility 실행
```console
$ psql postgres
```

#### 4.3. 사용자 생성
```console
postgres=# create user sonar;
```

#### 4.4. 사용자 Role 설정
```console
postgres=# alter role sonar with createdb;
```

#### 4.5. 사용자 비밀번호 설정
```console
postgres=# alter user sonar with encrypted password 'sonar-password';
postgres=# alter user postgres password 'postgres-password';
```

> 생성한 사용자 정보를 조회합니다.
```console
postgres=# \du
```

#### 4.6. Database 생성
```console
postgres=# create database sonar owner sonar;
```

#### 4.7. Privileges 설정
```console
postgres=# grant all privileges on database sonar to sonar;
```

> 생성한 Database를 조회합니다.
```console
postgres=# \l
```

#### 4.8. psql utility 종료
```console
postgres=# \q
```

#### 4.9. postgres 계정 로그아웃
```console
$ exit
```

### 5. SonarQube 설정
- 위에서 생성한 PostgreSQL 사용자 정보와 Database 정보, SonarQube UI의 포트를 설정합니다.

```console
$ vi ${LINDAREX_WORKSPACE}/sonarqube/conf/sonar.properties
```

```shell
--------------------------------------------------------------------------------
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar-password
sonar.jdbc.url=jdbc:postgresql://{MY-IP}/sonar
sonar.web.port=9000
--------------------------------------------------------------------------------
```

- OpenJDK(Java) 경로를 설정합니다.

```console
$ vi ${LINDAREX_WORKSPACE}/sonarqube/conf/wrapper.conf
```

```shell
--------------------------------------------------------------------------------
wrapper.java.command=/home/rex/workspace/tool/java11/bin/java
--------------------------------------------------------------------------------
```

### 6. Max map count 설정
```console
$ sudo vi /etc/profile
```

```shell
--------------------------------------------------------------------------------
sudo sysctl -w vm.max_map_count=262144
--------------------------------------------------------------------------------
```

> SonarQube를 Linux에 설치 시, 아래 사항이 요구됩니다.
- vm.max_map_count :: 262144 이상
- fs.file-max :: 65536 이상
- file descriptors :: 65536 이상
- threads :: 4096 이상

> SonarQube 설치 시 필요 요구사항에 대한 자세한 정보는 [https://docs.sonarqube.org/latest/requirements/requirements/](https://docs.sonarqube.org/latest/requirements/requirements/){: target="\_blank"}를 확인해 주시기 바랍니다.

> 수정 내역 적용을 위해 아래 명령어를 입력합니다.
```console
$ sudo source /etc/profile
```

### 7. 스크립트로 SonarQube 서비스(service) 관리
#### 7.1. SonarQube service 시작
```console
$ ${LINDAREX_WORKSPACE}/sonarqube/bin/linux-x86-64/sonar.sh start
```

#### 7.2. SonarQube service 중지
```console
$ ${LINDAREX_WORKSPACE}/sonarqube/bin/linux-x86-64/sonar.sh stop
```

#### 7.3. SonarQube service 재시작
```console
$ ${LINDAREX_WORKSPACE}/sonarqube/bin/linux-x86-64/sonar.sh restart
```

#### 7.3. SonarQube service 상태 조회
```console
$ ${LINDAREX_WORKSPACE}/sonarqube/bin/linux-x86-64/sonar.sh status
```

### 8. 웹브라우저로 SonarQube 접속
- http://[MY-IP]:9000


## 마무리(CONCLUSION)
ubuntu 환경에 SonarQube 설치를 완료했습니다. <br />
다음 포스트에서는 SonarQube 사용 방법을 소개하겠습니다.


## 참고(REFERENCES)
- [https://www.sonarqube.org/](https://www.sonarqube.org/){: target="\_blank"}
- [https://docs.sonarqube.org/latest/](https://docs.sonarqube.org/latest/){: target="\_blank"}
- [http://www.sonarqube.org/downloads/](http://www.sonarqube.org/downloads/){: target="\_blank"}
