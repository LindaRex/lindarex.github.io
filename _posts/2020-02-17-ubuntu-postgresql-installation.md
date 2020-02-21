---
title: "우분투(Ubuntu) 환경에 패키지로 PostgreSQL 설치하기"
categories: 
  - postgresql
tags: 
  - postgresql
  - ubuntu
---


PostgreSQL은 BSD 또는 MIT 라이선스와 유사한 PostgreSQL 라이선스로 오픈소스 소프트웨어로 배포됩니다. <br />
PostgreSQL은 확장 가능성 및 표준 준수를 강조하는 객체-관계형 데이터베이스 관리 시스템(ORDBMS, object-relational database management system)으로, 트랜잭션과 ACID(Atomicity, Consistency, Isolation, Durability)를 지원합니다. <br />
macOS 서버의 경우 PostgreSQL이 기본 데이터베이스이며, MS Windows와 리눅스에서도 이용할 수 있습니다. <br />
이 포스트에서는 우분투(이하 Ubuntu) 환경에서 패키지로 PostgreSQL을 설치하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- Ubuntu 환경에 Java가 설치되어 있어야 합니다.

> Java 설치 방법은 [우분투(Ubuntu) 환경에 OpenJDK(Java) 설치하기](https://lindarex.github.io/ubuntu/ubuntu-openjdk-installation/){: target="_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.1 build-15018445)
- Ubuntu 18.04.3 LTS (Bionic Beaver) Server (64-bit)
- PostgreSQL 11.5 (Ubuntu 11.5-3.pgdg18.04+1)
- OpenJDK 1.8.0_222


## 요약(SUMMARY)
1. PostgreSQL Debian packages repository 설정
2. apt 명령어로 PostgreSQL 설치
3. PostgreSQL 설정
4. systemctl 명령어로 PostgreSQL 실행


## 내용(CONTENTS)
### 1. PostgreSQL Debian packages repository 추가
```shell
$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```

### 2. PostgreSQL Debian packages repository key 추가
```shell
$ sudo wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -
```

### 3. apt install 명령어로 PostgreSQL 설치
```shell
$ sudo apt update -y && sudo apt install postgresql postgresql-contrib -y
```

### 4. PostgreSQL 설정

> 일반 사용자 계정으로 진행합니다.

### 4.1. PGDATA 디렉터리 생성
```shell
$ export LINDAREX_WORKSPACE=${HOME}/workspace
$ mkdir -p ${LINDAREX_WORKSPACE}/pgsql/data
```

### 4.2. PGDATA 디렉터리 경로 추가
```shell
$ sudo vi /etc/profile
--------------------------------------------------------------------------------
export LINDAREX_WORKSPACE=/home/ubuntu/workspace
export PGDATA=${LINDAREX_WORKSPACE}/pgsql/data
--------------------------------------------------------------------------------
```

> 위 workspace 디렉터리 경로는 사용자 계정에 따라 다릅니다.

> 수정 내역 적용을 위해 아래 명령어를 입력합니다.
```shell
$ source /etc/profile
```

### 4.3. Database 초기화
- 아래 명령어로 Database 초기화를 실행합니다.

```shell
$ /usr/lib/postgresql/11/bin/initdb
```

### 4.4. PostgreSQL 설정 수정
- 아래 명령어는 외부 접속이 가능하게 합니다.

```shell
$ sudo vi /etc/postgresql/11/main/postgresql.conf
--------------------------------------------------------------------------------
listen_addresses = '*'
--------------------------------------------------------------------------------
```

- 아래 명령어는 모든 사용자(비밀번호 사용)가 접속이 가능하게 합니다.

```shell
$ sudo vi /etc/postgresql/11/main/pg_hba.conf
--------------------------------------------------------------------------------
## Add at the bottom
host    all             all             0.0.0.0/0               password
--------------------------------------------------------------------------------
```

### 5. systemctl 명령어로 PostgreSQL 서비스 관리
### 5.1. PostgreSQL 서비스 설정 반영
```shell
$ sudo systemctl daemon-reload
```

### 5.2. PostgreSQL 서비스 시작
```shell
$ sudo systemctl start postgresql.service
```

### 5.3. PostgreSQL 서비스 중지
```shell
$ sudo systemctl stop postgresql.service
```

### 5.4. PostgreSQL 서비스 재시작
```shell
$ sudo systemctl restart postgresql.service
```

### 5.5. PostgreSQL 서비스 설정 재적용
```shell
$ sudo systemctl reload postgresql.service
```

### 5.6. PostgreSQL 서비스 상태 조회
```shell
$ sudo systemctl status postgresql.service
```

### 5.7. PostgreSQL 서비스 활성화(부팅 시 자동 시작)
```shell
$ sudo systemctl enable postgresql.service
```

### 5.8. PostgreSQL 서비스 비활성화
```shell
$ sudo systemctl disable postgresql.service
```

### 5.9. PostgreSQL 서비스 및 관련 프로세스 모두 중지
```shell
$ sudo systemctl kill postgresql.service
```


## 마무리(CONCLUSION)
Ubuntu 환경에 패키지로 PostgreSQL 설치를 완료했습니다. <br />
다음 포스트에서는 PostgreSQL 사용 방법을 소개하겠습니다.


## 참고(REFERENCES)
- [https://www.postgresql.org/](https://www.postgresql.org/){: target="_blank"}
- [https://www.postgresql.org/download/linux/ubuntu/](https://www.postgresql.org/download/linux/ubuntu/){: target="_blank"}
- [https://ko.wikipedia.org/wiki/PostgreSQL](https://ko.wikipedia.org/wiki/PostgreSQL){: target="_blank"}
- [https://d2.naver.com/helloworld/227936](https://d2.naver.com/helloworld/227936){: target="_blank"}
