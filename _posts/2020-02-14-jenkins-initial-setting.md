---
title: "젠킨스(Jenkins) 초기 설정하기"
categories: 
  - jenkins
tags: 
  - jenkins
---


이 포스트에서는 우분투(ubuntu) 환경에 젠킨스(Jenkins)를 설치한 후 초기 설정하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- ubuntu 환경에 Jenkins가 설치되어 있어야 합니다.

> Jenkins 설치 방법은 [우분투(Ubuntu) 환경에 패키지로 젠킨스(Jenkins) 설치하기](https://lindarex.github.io/jenkins/jenkins-initial-setting/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- Ubuntu 16.04.6 LTS (Xenial Xerus) Server (64-bit)
- OpenJDK 1.8.0_242
- Jenkins 2.204.2


## 요약(SUMMARY)
1. Jenkins 접속 및 활성화
2. Jenkins 추천 플러인 설치
3. 관리자(Admin) 계정 등록
4. Jenkins 접속 URL 확인
5. OpenJDK Java 설정


## 내용(CONTENTS)
### 1. Jenkins 접속

![lindarex-jenkins-initial-setting-001]

### 2. Jenkins 활성화
- 아래 명령어로 Jenkins 활성화에 필요한 암호(Password)를 조회합니다.

```console
$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
91623591371f4f57bf6814a674bfeda9
```

![lindarex-jenkins-initial-setting-002]

### 3. Jenkins 플러인(plugin) 설치
- 'Install suggested plugins'를 선택합니다.

![lindarex-jenkins-initial-setting-003]

- plugin 설치를 진행합니다.

![lindarex-jenkins-initial-setting-004]

### 4. 관리자(Admin) 계정 등록

![lindarex-jenkins-initial-setting-005]

### 5. Jenkins 접속 URL 확인
- 포트(port) 번호를 포함한 전체 URL을 기재합니다.

![lindarex-jenkins-initial-setting-006]

- Jenkins 사용을 위한 준비를 완료했습니다.

![lindarex-jenkins-initial-setting-007]

- 초기 설정 완료 후 Jenkins 첫 페이지로 이동합니다.

![lindarex-jenkins-initial-setting-008]

### 6. OpenJDK Java 설정
- 아래 메뉴를 통해 'Global Tool Configuration' 페이지로 이동합니다.

> Jenkins > 왼쪽 상단 메뉴 > Jenkins 관리 > Global Tool Configuration

![lindarex-jenkins-initial-setting-009]

![lindarex-jenkins-initial-setting-010]

- JDK 설정

    + 'JDK Installations' 버튼을 클릭한 후, 'Add JDK' 버튼을 클릭합니다.
    + 'Install automatically'을 체크해제하여, JDK name과 JAVA_HOME을 기재합니다.

   > JAVA_HOME은 아래 명령어로 조회할 수 있습니다.
   ```console
   $ echo $JAVA_HOME
   /usr/lib/jvm/java-8-openjdk-amd64
   ```

![lindarex-jenkins-initial-setting-011]


## 마무리(CONCLUSION)
Jenkins 설치 후 초기 설정을 완료했습니다. <br />
다음 포스트에서는 [우분투(Ubuntu) 환경에 WAR 파일로 젠킨스(Jenkins) 설치하기](https://lindarex.github.io/jenkins/ubuntu-jenkins-war-installation/){: target="\_blank"}를 소개하겠습니다.


## 참고(REFERENCES)
- [https://jenkins.io/](https://jenkins.io/){: target="\_blank"}


[lindarex-jenkins-initial-setting-001]:/assets/images/2020-02-14-jenkins-initial-setting/lindarex-jenkins-initial-setting-001.png
[lindarex-jenkins-initial-setting-002]:/assets/images/2020-02-14-jenkins-initial-setting/lindarex-jenkins-initial-setting-002.png
[lindarex-jenkins-initial-setting-003]:/assets/images/2020-02-14-jenkins-initial-setting/lindarex-jenkins-initial-setting-003.png
[lindarex-jenkins-initial-setting-004]:/assets/images/2020-02-14-jenkins-initial-setting/lindarex-jenkins-initial-setting-004.png
[lindarex-jenkins-initial-setting-005]:/assets/images/2020-02-14-jenkins-initial-setting/lindarex-jenkins-initial-setting-005.png
[lindarex-jenkins-initial-setting-006]:/assets/images/2020-02-14-jenkins-initial-setting/lindarex-jenkins-initial-setting-006.png
[lindarex-jenkins-initial-setting-007]:/assets/images/2020-02-14-jenkins-initial-setting/lindarex-jenkins-initial-setting-007.png
[lindarex-jenkins-initial-setting-008]:/assets/images/2020-02-14-jenkins-initial-setting/lindarex-jenkins-initial-setting-008.png
[lindarex-jenkins-initial-setting-009]:/assets/images/2020-02-14-jenkins-initial-setting/lindarex-jenkins-initial-setting-009.png
[lindarex-jenkins-initial-setting-010]:/assets/images/2020-02-14-jenkins-initial-setting/lindarex-jenkins-initial-setting-010.png
[lindarex-jenkins-initial-setting-011]:/assets/images/2020-02-14-jenkins-initial-setting/lindarex-jenkins-initial-setting-011.png
