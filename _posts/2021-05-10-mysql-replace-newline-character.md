---
title: "MySQL 개행 문자 처리(Replace newline character)"
categories: 
  - concepts
tags: 
  - mysql
  - "open source software"
  - "open source"
  - oss
---


개행 문자(newline)는 텍스트의 한 줄이 끝났음을 나타내는 문자 또는 문자열입니다.
<br /><br />
일반적으로 운영 체제(operating system, os)에 따라 개행 문자 코드가 다르기 때문에 다른 시스템으로 전송할 때 개행 문자의 치환 작업도 필요합니다.
<br /><br />
이 포스트에서는 MySQL 환경에서 개행 문자 처리 방법에 대해 소개합니다.


## 요약(SUMMARY)
1. 개행 문자 소개
2. MySQL 환경에서 개행 문자 처리


## 내용(CONTENTS)
### 1. 개행 문자 소개
- 새줄 문자, 줄바꿈 문자(line break), EOL(end-of-line), line ending과 같은 뜻입니다.
- 윈도우 계열의 개행 문자는 CRLF(\r\n), 16진수로 표현하면 0d0a입니다.
- 유닉스 계열(리눅스 또는 맥OS)의 개행 문자는 LF(\n), 16진수로 표현하면 0a입니다.


#### 1.1. 라인피드(line feed, lf) 소개
- 커서를 한 칸 아래로 이동합니다. 즉 새로운 행을 추가(new line feed)합니다.
- "\n"으로 표현하며, 아스키코드는 10번입니다.

#### 1.2. 캐리지 리턴(carrige return, cr) 소개
- 커서를 가장 왼쪽으로 이동합니다. 즉 시작 위치로 복귀(return)합니다.
- "\r"로 표현하며, 아스키코드는 13번입니다.

> 아스키코드



## 마무리(CONCLUSION)
Oracle은 2019년 1월부터 Java를 상용으로 사용하거나 지속적인 업데이트를 받으려는 기업 사용자에게 유료로 제공하고 있으며, 
현재 2020년 2월 기준으로 Oracle JDK 8은 개인 또는 개발 용도의 사용은 무료지만, 2021년부터는 유료화로 전환될 것으로 알려져 있습니다.
<br /><br />
JDK 유료화에 따라 개인 사용자를 비롯해 기업 사용자는 OpenJDK를 적용하기 위해 호환성, 안정성, 성능 등의 검증이 필요합니다. 또한, 배포 버전에 따른 생산성 확인과 보안 이슈 등을 충분히 인지하고 테스트할 필요가 있습니다.
<br /><br />
더 자세한 내용은 아래 참고 페이지를 확인해 주시기 바랍니다.


## 참고(REFERENCES)
- [https://ko.wikipedia.org/wiki/MySQL](https://ko.wikipedia.org/wiki/MySQL){: target="\_blank"}
- [https://ko.wikipedia.org/wiki/새줄_문자](https://ko.wikipedia.org/wiki/%EC%83%88%EC%A4%84_%EB%AC%B8%EC%9E%90){: target="\_blank"}
