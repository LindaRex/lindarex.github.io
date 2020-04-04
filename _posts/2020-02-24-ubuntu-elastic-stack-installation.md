---
title: "우분투(Ubuntu) 환경에 패키지(Package)로 Elastic Stack 설치하기"
categories: 
  - elastic-stack
tags: 
  - "elastic stack"
  - "elk stack"
  - elasticsearch
  - logstash
  - kibana
  - filebeat
  - ubuntu
---


Elastic Stack을 소개하기에 앞서 ELK Stack을 먼저 소개하겠습니다. <br />
ELK Stack은 Elasticsearch, Logstash, Kibana의 연동으로 텍스트, 숫자, 위치 기반 정보, 정형 및 비정형 데이터(data) 등 모든 유형의 data를 수집 및 변환하고 분석하여 시각화하는 오픈소스(open source) 소프트웨어입니다. <br />
2015년에 ELK Stack에 경량의 단일 목적 data 수집기 제품군(Beats)을 도입하면서 Elastic Stack으로 명칭이 변경되었습니다. <br />
즉, ELK Stack에 Beats가 추가되어 Elastic Stack으로 통합되었습니다. <br />
이 포스트에서는 우분투(ubuntu) 환경에서 패키지(package)로 elastic stack를 설치하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- ubuntu 환경에 Java가 설치되어 있어야 합니다.
- 방화벽 설정이 필요합니다.
    + TCP 9200 포트, TCP 5601 포트가 개방되어 있어야 합니다.

> Java 설치 방법은 [우분투(Ubuntu) 환경에 OpenJDK(Java) 설치하기](https://lindarex.github.io/ubuntu/ubuntu-openjdk-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

> 방화벽 설정 방법은 [우분투(Ubuntu) 환경에 방화벽(Firewalld) 설치 및 설정하기](https://lindarex.github.io/ubuntu/ubuntu-firewalld-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.1 build-15018445)
- Ubuntu 16.04.6 LTS (Xenial Xerus) Server (64-bit)
- Elasticsearch 7.2.0
- Logstash 7.2.0
- Kibana 7.2.0
- Filebeat 7.2.0
- OpenJDK 1.8.0_212


## 요약(SUMMARY)
1. Elasticsearch 설치
2. Logstash 설치
3. Kibana 설치
4. Filebeat 설치
5. 웹브라우저로 Kibana 접속


## 내용(CONTENTS)
### 1. Elasticsearch 설치
- Elasticsearch는 Apache Lucene으로 구축된 JSON 기반의 분산형 오픈소스 RESTful 검색 분석 엔진이며, Logstash를 통해 수신된 data를 저장소에 저장하는 역할을 담당합니다.

#### 1.1. Elasticsearch debian packages repository 추가
```console
$ echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```

#### 1.2. Elasticsearch debian packages repository key 추가
```console
$ wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

#### 1.3. apt install 명령어로 Elasticsearch 설치
```console
$ sudo apt update && sudo apt install elasticsearch -y
```

#### 1.4. systemctl 명령어로 Elasticsearch 서비스(service) 관리
##### 1.4.1. Elasticsearch service 설정 반영
```console
$ sudo systemctl daemon-reload
```

##### 1.4.2. Elasticsearch service 시작
```console
$ sudo systemctl start elasticsearch.service
```

##### 1.4.3. Elasticsearch service 중지
```console
$ sudo systemctl stop elasticsearch.service
```

##### 1.4.4. Elasticsearch service 재시작
```console
$ sudo systemctl restart elasticsearch.service
```

##### 1.4.5. Elasticsearch service 설정 재적용
```console
$ sudo systemctl reload elasticsearch.service
```

##### 1.4.6. Elasticsearch service 상태 조회
```console
$ sudo systemctl status elasticsearch.service
```

##### 1.4.7. Elasticsearch service 활성화(부팅 시 자동 시작)
```console
$ sudo systemctl enable elasticsearch.service
```

##### 1.4.8. Elasticsearch service 비활성화
```console
$ sudo systemctl disable elasticsearch.service
```

##### 1.4.9. Elasticsearch service 및 관련 프로세스(process) 모두 중지
```console
$ sudo systemctl kill elasticsearch.service
```

#### 1.5. curl 명령어로 Elasticsearch service 확인
```console
$ curl -X GET http://localhost:9200
{
  "name" : "lindarex-elk",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "RXW_zgPsS9mWiB5CDi2aCQ",
  "version" : {
    "number" : "7.2.0",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "508c38a",
    "build_date" : "2020-02-20T15:54:18.811730Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

### 2. Logstash 설치
- Logstash는 서버(server) 사이드 data 처리 파이프라인으로, 여러 다양한 소스에서 동시에 data를 수집 및 변환하여 Elasticsearch와 같은 stash 보관소로 전송합니다.

#### 2.1. (선택사항) SSL certificate 생성
##### 2.1.1. Hostname or FQDN 설정
```console
$ cd /etc/ssl/
$ sudo openssl req -x509 -nodes -newkey rsa:2048 -days 365 -keyout logstash-forwarder.key -out logstash-forwarder.crt -subj /CN=server.lindarex.local
```

##### 2.1.2. IP Address 설정
```console
$ sudo vi /etc/ssl/openssl.cnf
```

```shell
--------------------------------------------------------------------------------
...
[ v3_ca ]
subjectAltName = IP:192.168.10.20
...
--------------------------------------------------------------------------------
```

```console
$ cd /etc/ssl/
$ sudo openssl req -x509 -days 365 -batch -nodes -newkey rsa:2048 -keyout logstash-forwarder.key -out logstash-forwarder.crt
```

##### 2.1.3. SSL 변환
```console
$ cd /etc/ssl/
$ sudo openssl pkcs8 -in logstash-forwarder.key  -topk8 -nocrypt -out logstash-forwarder.key.pem
$ sudo chmod 644 /etc/ssl/logstash-forwarder.key.pem
```

#### 2.2. apt install 명령어로 Logstash 설치
```console
$ sudo apt install logstash -y
```

#### 2.3. Logstash 구성
```console
$ sudo vi /etc/logstash/conf.d/logstash.conf
```

```shell
--------------------------------------------------------------------------------
input {
 beats {
   port => 5044
   
   # Set to False if you do not SSL // (선택사항) SSL 미사용 시에 'false' 설정
   ssl => true
  
   # Delete below lines if no SSL is used  // (선택사항) SSL 미사용 시에 아래 설정 삭제
   ssl_certificate => "/etc/ssl/logstash-forwarder.crt"
   ssl_key => "/etc/ssl/logstash-forwarder.key.pem"
   }
}

filter {
if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGLINE}" }
    }

    date {
match => [ "timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
}
  }

}

output {
 elasticsearch {
  hosts => localhost
    index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
       }
stdout {
    codec => rubydebug
       }
}
--------------------------------------------------------------------------------
```

#### 2.4. systemctl 명령어로 Logstash service 관리
##### 2.4.1. Logstash service 설정 반영
```console
$ sudo systemctl daemon-reload
```

##### 2.4.2. Logstash service 시작
```console
$ sudo systemctl start logstash.service
```

##### 2.4.3. Logstash service 중지
```console
$ sudo systemctl stop logstash.service
```

##### 2.4.4. Logstash service 재시작
```console
$ sudo systemctl restart logstash.service
```

##### 2.4.5. Logstash service 설정 재적용
```console
$ sudo systemctl reload logstash.service
```

##### 2.4.6. Logstash service 상태 조회
```console
$ sudo systemctl status logstash.service
```

##### 2.4.7. Logstash service 활성화(부팅 시 자동 시작)
```console
$ sudo systemctl enable logstash.service
```

##### 2.4.8. Logstash service 비활성화
```console
$ sudo systemctl disable logstash.service
```

##### 2.4.9. Logstash service 및 관련 process 모두 중지
```console
$ sudo systemctl kill logstash.service
```

### 3. Kibana 설치
- Kibana는 프런트 엔드 애플리케이션으로, Elasticsearch에서 인덱스 된 data 검색 및 다양한 차트와 그래프를 제공하고, 실시간으로 data를 분석하여 시각화를 담당합니다.

#### 3.1. apt install 명령어로 Kibana 설치
```console
$ sudo apt install kibana -y
```

#### 3.2. Kibana 구성
```console
$ sudo vi /etc/kibana/kibana.yml
```

```shell
--------------------------------------------------------------------------------
...
#server.host: "localhost"
server.host: "192.168.10.20"
#elasticsearch.hosts: ["http://localhost:9200"]
elasticsearch.hosts: ["http://localhost:9200"]
...
--------------------------------------------------------------------------------
```

#### 3.3. systemctl 명령어로 Kibana service 관리
##### 3.3.1. Kibana service 설정 반영
```console
$ sudo systemctl daemon-reload
```

##### 3.3.2. Kibana service 시작
```console
$ sudo systemctl start kibana.service
```

##### 3.3.3. Kibana service 중지
```console
$ sudo systemctl stop kibana.service
```

##### 3.3.4. Kibana service 재시작
```console
$ sudo systemctl restart kibana.service
```

##### 3.3.5. Kibana service 설정 재적용
```console
$ sudo systemctl reload kibana.service
```

##### 3.3.6. Kibana service 상태 조회
```console
$ sudo systemctl status kibana.service
```

##### 3.3.7. Kibana service 활성화(부팅 시 자동 시작)
```console
$ sudo systemctl enable kibana.service
```

##### 3.3.8. Kibana service 비활성화
```console
$ sudo systemctl disable kibana.service
```

##### 3.3.9. Kibana service 및 관련 process 모두 중지
```console
$ sudo systemctl kill kibana.service
```

### 4. Filebeat 설치
- Filebeat는 경량 로그(log) 수집기로, SSH 터미널의 사용이 불가능한 상황(log를 생성하는 server나 가상 시스템, 컨테이너가 수백\~수천 개에 이르는 경우)에 log와 파일을 경량화된 방식으로 전달하고 중앙 집중화하여, 작업을 보다 간편하게 만들어 주는 역할을 합니다.

#### 4.1. apt install 명령어로 Filebeat 설치
```console
$ sudo apt install filebeat -y
```

#### 4.2. Filebeat 구성
```console
$ sudo vi /etc/filebeat/filebeat.yml
```

```shell
--------------------------------------------------------------------------------
...
- type: log

  # Change to true to enable this input configuration.
  #enabled: false
  enabled: true

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    #- /var/log/*.log
    - /var/log/syslog
...
#-------------------------- Elasticsearch output ------------------------------
#output.elasticsearch:
  # Array of hosts to connect to.
  #hosts: ["localhost:9200"]

  # Optional protocol and basic auth credentials.
  #protocol: "https"
  #username: "elastic"
  #password: "changeme"

#----------------------------- Logstash output --------------------------------
output.logstash:

  #hosts: ["localhost:5044"]
  hosts: ["192.168.10.20:5044"]
    
  # Comment out this line if you are not using SSL on Logstash server
  #ssl.certificate_authorities: ["/etc/pki/root/ca.pem"]
  ssl.certificate_authorities: ["/etc/ssl/logstash-forwarder.crt"]
...
--------------------------------------------------------------------------------
```

#### 4.3. systemctl 명령어로 Filebeat service 관리
##### 4.3.1. Filebeat service 설정 반영
```console
$ sudo systemctl daemon-reload
```

##### 4.3.2. Filebeat service 시작
```console
$ sudo systemctl start filebeat.service
```

##### 4.3.3. Filebeat service 중지
```console
$ sudo systemctl stop filebeat.service
```

##### 4.3.4. Filebeat service 재시작
```console
$ sudo systemctl restart filebeat.service
```

##### 4.3.5. Filebeat service 설정 재적용
```console
$ sudo systemctl reload filebeat.service
```

##### 4.3.6. Filebeat service 상태 조회
```console
$ sudo systemctl status filebeat.service
```

##### 4.3.7. Filebeat service 활성화(부팅 시 자동 시작)
```console
$ sudo systemctl enable filebeat.service
```

##### 4.3.8. Filebeat service 비활성화
```console
$ sudo systemctl disable filebeat.service
```

##### 4.3.9. Filebeat service 및 관련 process 모두 중지
```console
$ sudo systemctl kill filebeat.service
```

### 5. 웹브라우저로 Jenkins 접속
- http://[MY-IP]:5601


## 마무리(CONCLUSION)
ubuntu 환경에 package로 elastic stack 설치를 완료했습니다. <br />
다음 포스트에서는 elastic stack 사용 방법을 소개하겠습니다.


## 참고(REFERENCES)
- [https://www.elastic.co/kr/](https://www.elastic.co/kr/){: target="\_blank"}
- [https://ko.wikipedia.org/wiki/일래스틱서치](https://ko.wikipedia.org/wiki/%EC%9D%BC%EB%9E%98%EC%8A%A4%ED%8B%B1%EC%84%9C%EC%B9%98){: target="\_blank"}
