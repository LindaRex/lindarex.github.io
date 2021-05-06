---
title: "MySQL 개행 문자(newline character) 치환(replace)하기"
categories: 
  - concepts
tags: 
  - mysql
  - "newline"
  - "newline character"
  - "line break"
  - "eol"
  - "end-of-line"
  - "line ending"
  - "line feed"
  - "carrige return"
  - "open source software"
  - "open source"
  - oss
---


개행 문자(newline character)는 텍스트의 한 줄이 끝났음을 나타내는 문자 또는 문자열입니다.
<br /><br />
일반적으로 운영 체제(operating system, os)에 따라 개행 문자 코드가 다르기 때문에 다른 시스템으로 전송할 때 개행 문자의 치환 작업도 필요합니다.
<br /><br />
이 포스트에서는 MySQL(mysql) 환경에서 개행 문자를 치환하는 방법에 대해 소개합니다.


## 요약(SUMMARY)
1. 개행 문자 소개
2. mysql 환경에서 개행 문자 치환


## 내용(CONTENTS)
### 1. 개행 문자 소개
- 개행 문자는 새줄 문자, 줄바꿈 문자(line break), EOL(end-of-line), line ending과 같은 뜻입니다.
- 윈도우 계열의 개행 문자는 CRLF(\r\n)이고, 16진수로 표현하면 "0d0a"입니다.
- 유닉스 계열(리눅스 또는 맥OS)의 개행 문자는 LF(\n)이고, 16진수로 표현하면 "0a"입니다.

#### 1.1. 라인피드(line feed, lf)
- line feed는 새로운 행을 추가(new line feed)하는, 커서를 한 칸 아래로 이동하는 것을 의미합니다.
- "\n"으로 표현하며, 아스키코드(ASCII)는 10번입니다.

#### 1.2. 캐리지 리턴(carrige return, cr)
- 캐리지 리턴은 시작 위치로 복귀(return)하는, 커서를 가장 왼쪽으로 이동하는 것을 의미합니다.
- "\r"로 표현하며, ASCII는 13번입니다.

> ASCII(American Standard Code for Information Interchange)에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/ASCII](https://ko.wikipedia.org/wiki/ASCII){: target="\_blank"} 페이지를 참고하시기 바랍니다.

### 2. mysql 환경에서 개행 문자 치환
#### 2.1. line feed(lf) 처리
```
SELECT REPLACE(lindarex_column, CHR(10),'') FROM lindarex_table;
SELECT REPLACE(lindarex_column, '\n', '') FROM lindarex_table;
```

#### 2.2. carrige return(cr) 처리
```
SELECT REPLACE(lindarex_column, CHR(13),'') FROM lindarex_table;
SELECT REPLACE(lindarex_column, '\r', '') FROM lindarex_table;
```

#### 2.3. crlf 처리
```
SELECT REPLACE(REPLACE(lindarex_column, CHR(13),''), CHR(10),'') FROM lindarex_table;
SELECT REPLACE(REPLACE(lindarex_column, '\n', ''), '\r', '') FROM lindarex_table;
```


## 마무리(CONCLUSION)
mysql 환경에 개행 문자를 치환하는 방법에 대해 간단히 소개했습니다.
<br />
개행 문자에 대한 더 자세한 내용은 아래 참고 페이지를 확인해 주시기 바랍니다.


## 참고(REFERENCES)
- [https://ko.wikipedia.org/wiki/새줄_문자](https://ko.wikipedia.org/wiki/%EC%83%88%EC%A4%84_%EB%AC%B8%EC%9E%90){: target="\_blank"}
- [https://ko.wikipedia.org/wiki/MySQL](https://ko.wikipedia.org/wiki/MySQL){: target="\_blank"}
