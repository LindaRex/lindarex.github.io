---
title: "AWS EC2 우분투(Ubuntu) 환경에 BOSH director 설치하기"
categories: 
  - bosh
tags: 
  - bosh
  - ubuntu
  - aws
  - ec2
---


Cloud Foundry BOSH는 소규모 및 대규모 클라우드 애플리케이션의 배포와 라이프 사이클 관리, release engineering을 통합하는 오픈소스 소프트웨어 프로젝트입니다. 
BOSH는 수백 개의 가상머신(Virtual Machine, 이하 VM)에 애플리케이션을 프로비저닝(Provisioning)하고 배포할 수 있으며, 모니터링과 오류 복구, 애플리케이션 업데이트를 수행합니다. <br />
BOSH는 Cloud Foundry PaaS(CFAR) 배포를 위해 개발되었지만, 거의 모든 소프트웨어를 배포하는 데에도 사용할 수 있습니다. 예를 들면, Hadoop 또는 Jenkins 등의 오픈소스 소프트웨어를 BOSH release로 작성하여 배포할 수 있으며 대규모 분산 시스템에 적합합니다. <br />
또한 BOSH는 Amazon Web Services EC2, Google Cloud Platform, Microsoft Azure, OpenStack, VMware vSphere 및 Alibaba Cloud와 같은 다양한 IaaS(Infrastructure as a Service) provider를 지원하며, Apache CloudStack, VirtualBox 등의 IaaS provider 지원을 위해 CPI(Cloud Provider Interface)를 제공합니다. <br />
이 포스트에서는 AWS EC2 우분투(이하 Ubuntu) 환경에서 BOSH 사용을 위한 BOSH director를 설치하는 방법을 소개합니다.

> CFAR에 대해서는 본문의 [CF CLI 설치](#2-%EC%84%A0%ED%83%9D%EC%82%AC%ED%95%AD-cf-cli-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0)에서 설명합니다.


## 선행조건(PREREQUISITE)
- AWS 환경이 필요합니다.
    + VPC 설정이 필요합니다.
    + Subnet 설정이 필요합니다.
    + Internet gateway 설정이 필요합니다.
    + NAT Gateway 설정이 필요합니다.
    + Routing table 설정이 필요합니다.
    + security group 설정이 필요합니다.
    + Quota 설정이 필요합니다.
- AWS EC2 Inception VM이 필요합니다.

> BOSH director 설치를 위한 AWS 환경 설정 및 AWS EC2 Inception VM 생성 방법은 다음 포스트에서 소개하겠습니다.

> AWS, GCP, MS Azure, OpenStack, VMware vSphere 등 IaaS 환경에 BOSH director를 설치하여 BOSH를 사용하거나, 로컬(이하 local) 환경에 BOSH-LITE를 설치하여 BOSH를 사용할 수 있습니다.

> Provisioning이란 사용자의 요구에 맞게 시스템 자원을 할당, 배치, 배포해 두었다가 필요 시 시스템을 즉시 사용할 수 있는 상태로 미리 준비해 두는 것을 의미합니다.

> Provisioning에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/프로비저닝](https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9C%EB%B9%84%EC%A0%80%EB%8B%9D){: target="\_blank"}, [https://en.wikipedia.org/wiki/Provisioning_(telecommunications)](https://en.wikipedia.org/wiki/Provisioning_(telecommunications)){: target="\_blank"}를 확인해 주시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- AWS EC2
- Ubuntu 18.04.3 LTS (Bionic Beaver) Server (64-bit)


## 요약(SUMMARY)
1. BOSH CLI 설치
2. (선택사항) CF CLI 설치
3. CF UAA CLI 설치
4. Credhub CLI 설치
5. BOSH director 설치
6. BOSH UAA 통합 인증
7. BOSH director 설정
8. BOSH jumpbox 설정
9. Credhub 설정


## 내용(CONTENTS)

> 아래 명령어로 환경변수를 설정하고, workspace 디렉터리를 생성합니다.

```console
$ export LINDAREX_INCEPTION_USER_NAME='ubuntu'
$ export LINDAREX_BOSH_WORKSPACE=/home/${LINDAREX_INCEPTION_USER_NAME}/workspace
$ export LINDAREX_BOSH_DIRECTOR='micro-bosh'
$ export LINDAREX_BOSH_IAAS='aws'
$ mkdir -p ${LINDAREX_BOSH_WORKSPACE}
```

### 1. BOSH CLI 설치하기
- BOSH CLI는 v1 버전과 v2 버전이 존재하며, 이 포스트에서는 v2 버전을 기준으로 설명합니다.

#### 1.1. 종속 패키지 설치
- 자신이 사용하는 Ubuntu 버전에 따라 아래 명령어로 종속 패키지(Package dependencies)를 설치합니다.

> Ubuntu 18.04 (Bionic)

```console
$ sudo apt update -y
$ sudo apt install -y build-essential zlibc zlib1g-dev ruby ruby-dev openssl libxslt1-dev libxml2-dev libssl-dev libreadline7 libreadline-dev libyaml-dev libsqlite3-dev sqlite3
```

> Ubuntu 16.04 (Xenial) 또는 Ubuntu Trusty (14.04)

```console
$ sudo apt update -y
$ sudo apt install -y libcurl4-openssl-dev gcc g++ build-essential zlibc zlib1g-dev ruby ruby-dev openssl libxslt-dev libxml2-dev libssl-dev libreadline6 libreadline6-dev libyaml-dev libsqlite3-dev sqlite3
```

#### 1.2. BOSH CLI 파일 내려받기
```console
$ curl -Lo ${LINDAREX_BOSH_WORKSPACE}/bosh https://github.com/cloudfoundry/bosh-cli/releases/download/v6.2.1/bosh-cli-6.2.1-linux-amd64
```

#### 1.3. chmod(change mode) 명령어로 파일 권한 변경(실행 권한 부여)
```console
$ chmod +x ${LINDAREX_BOSH_WORKSPACE}/bosh
```

#### 1.4. 실행 파일을 사용자 프로그램 경로로 옮기기
```console
$ sudo mv ${LINDAREX_BOSH_WORKSPACE}/bosh /usr/local/bin/bosh
```

#### 1.5. BOSH CLI 설치 확인
```console
$ bosh -v
version 6.2.1-a28042ac-2020-02-10T18:40:57Z

Succeeded
```

### 2. (선택사항) CF CLI 설치하기
- CF CLI에서 CF는 정확히 표현하면 CFAR(cloud foundry application runtime)입니다.
- CF CLI는 CFAR 명령어를 사용하기 위한 설치입니다.
- CFAR은 BOSH로 배포하며, 다음 포스트에서 소개하겠습니다.

> CFAR은 CFCR(cloud foundry container runtime)이 생기기 전까지 CF(cloud foundry)로 불렸었고, CFCR도 Cloud Foundry Foundation incubating project 초반에는 kubo로 불렸습니다.

> 최근 컨테이너 플랫폼(Container platform)의 성장과 함께 Cloud Foundry도 kubernetes를 이용한 container runtime을 제공하면서, application runtime인 CF를 CFAR로, container runtime인 kubo를 CFCR로 명칭을 변경했습니다.


#### 2.1. CF CLI Debian packages repository key 추가
```console
$ wget -q -O - https://packages.cloudfoundry.org/debian/cli.cloudfoundry.org.key | sudo apt-key add -
```

#### 2.2. CF CLI Debian packages repository 추가
```console
$ echo "deb https://packages.cloudfoundry.org/debian stable main" | sudo tee /etc/apt/sources.list.d/cloudfoundry-cli.list
```

#### 2.3. apt install 명령어로 CF CLI 설치
```console
$ sudo apt update -y && sudo apt install cf-cli -y
```

#### 2.4. CF CLI 설치 확인
```console
$ cf --version
cf version 6.49.0+d0dfa93bb.2020-01-07
```

### 3. CF UAA CLI 설치하기
- UAA는 사용자 계정 및 인증 서버(User Account and Authentication server)입니다.
- UAA는 CFAR과 BOSH에 각각 존재하며 CF UAA CLI를 통해 모두 사용할 수 있습니다.

#### 3.1. Rubygems으로 CF UAA CLI 설치
```console
$ sudo gem install cf-uaac
```

#### 3.2. CF UAA CLI 설치 확인
```console
$ uaac -v
UAA client 4.2.0
```

### 4. Credhub CLI 설치
- CredHub는 비밀번호, 인증서(certificates), 인증 기관(certificate authorities), ssh 키, rsa 키와 같은 credentials 정보를 관리합니다. 

#### 4.1. Credhub CLI 파일 내려받기
```console
$ wget -P ${LINDAREX_BOSH_WORKSPACE} https://github.com/cloudfoundry-incubator/credhub-cli/releases/download/2.6.2/credhub-linux-2.6.2.tgz
```

#### 4.2. 내려받은 파일 압축 해제
```console
$ tar zxf ${LINDAREX_BOSH_WORKSPACE}/credhub-linux-2.6.2.tgz -C ${LINDAREX_BOSH_WORKSPACE}
```

#### 4.3. chmod 명령어로 파일 권한 변경(실행 권한 부여)
```console
$ chmod +x ${LINDAREX_BOSH_WORKSPACE}/credhub
```

#### 4.4. 실행 파일을 사용자 프로그램 경로로 옮기기
```console
$ sudo mv ${LINDAREX_BOSH_WORKSPACE}/credhub /usr/local/bin/credhub
```

#### 4.5. Credhub CLI 설치 확인
```console
$ credhub --version
CLI Version: 2.6.2
Server Version: Not Found. Have you targeted and authenticated against a CredHub server?
```

### 5. BOSH director 설치
#### 5.1. BOSH deployment repository clone 받기
```console
$ git clone https://github.com/cloudfoundry/bosh-deployment.git ${LINDAREX_BOSH_WORKSPACE}/bosh-deployment
```

> Cloud Foundry github repository에서 제공하는 BOSH deployment는 branch나 tag가 존재하지 않습니다. Clone 받는 당시의 버전으로 설치를 해야 하며, 간혹 오류를 포함한 버전도 배포되곤 합니다.

#### 5.2. 배포 스크립트 작성
```console
$ vi ${LINDAREX_BOSH_WORKSPACE}/bosh-deployment/deploy-${LINDAREX_BOSH_IAAS}.sh
```

```shell
--------------------------------------------------------------------------------
#!/bin/bash

bosh create-env bosh.yml \
	--state=aws/state.json \
	--vars-store=aws/creds.yml \
	-o aws/cpi.yml \
	-o uaa.yml \
	-o credhub.yml \
	-o jumpbox-user.yml \
	-v internal_cidr='10.0.1.0/24' \
	-v internal_gw='10.0.1.1' \
	-v internal_ip='10.0.1.6' \
	-v director_name='micro-bosh' \
	-v access_key_id='AKIBI7UEWB42Q4THBBFQ' \
	-v secret_access_key='br3K3YQM7SwvyP/0G4AJozpSLUFwaBxy+KRFy+3q' \
	-v region='ap-northeast-2' \
	-v az='ap-northeast-2c' \
	-v default_key_name='lindarex-inception' \
	-v default_security_groups=[lindarex-security] \
	-v subnet_id='subnet-087f7203d0bcd1396' \
	-v private_key='~/.ssh/lindarex-inception.pem'
--------------------------------------------------------------------------------
```

- '-o' 옵션은 YAML로 작성된 옵션 파일의 경로를 지정하여 배포 시에 적용합니다. 
- '-v' 옵션은 변수를 설정하여 배포 시에 적용합니다.

- 배포 스크립트를 상세하게 설명하겠습니다.
    + 'create-env' :: BOSH director를 배포하는 명령어입니다.
    + 'bosh.yml' :: 설정한 YAML 파일을 기반으로 Single VM을 생성합니다.
    + '--state' :: 배포 상태 파일의 경로입니다.
    + '--vars-store' :: Credentials 정보를 저장하는 YAML 파일의 경로입니다.
    + '-o aws/cpi.yml' :: AWS CPI를 적용합니다. IaaS 마다 CPI가 다릅니다.
    + '-o uaa.yml' :: UAA를 적용합니다.
    + '-o credhub.yml' :: Credhub을 적용합니다.
    + '-o jumpbox-user.yml' :: Jumpbox user를 적용합니다. BOSH director VM에 SSH 접속을 할 수 있습니다.
    + '-v internal_cidr' :: BOSH director VM에 적용할 네트워크의 cidr입니다.
    + '-v internal_gw' :: BOSH director VM에 적용할 네트워크의 gateway ip입니다.
    + '-v internal_ip' :: BOSH director VM에 적용할 네트워크의 private ip입니다.
    + '-v director_name' :: BOSH director 명입니다.
    + '-v access_key_id' :: IaaS가 AWS인 경우 필요한 변수이며, AWS access key id입니다.
    + '-v secret_access_key' :: IaaS가 AWS인 경우 필요한 변수이며, AWS secret access key입니다.
    + '-v region' :: IaaS가 AWS인 경우 필요한 변수이며, AWS region입니다.
    + '-v az' :: IaaS가 AWS인 경우 필요한 변수이며, AWS az(availability zone, 가용 영역)입니다.
    + '-v default_key_name' :: IaaS가 AWS인 경우 필요한 변수이며, AWS key name입니다.
    + '-v default_security_groups' :: IaaS가 AWS인 경우 필요한 변수이며, AWS security_group입니다.
    + '-v subnet_id' :: IaaS가 AWS인 경우 필요한 변수이며, BOSH director VM에 적용할 네트워크의 subnet id입니다.
    + '-v private_key' :: IaaS가 AWS인 경우 필요한 변수이며, AWS private key 파일의 경로입니다.

> 위 배포 스크립트는 AWS 서울 region에 배포되는 설정이지만, region과 az를 제외한 AWS 설정값은 잘못된 값이므로 참고만 하시기를 바랍니다.

#### 5.3. chmod 명령어로 파일 권한 변경(실행 권한 부여)
```console
$ chmod +x ${LINDAREX_BOSH_WORKSPACE}/bosh-deployment/deploy-${LINDAREX_BOSH_IAAS}.sh
```

#### 5.4. (선택사항) 배포 상태 파일 삭제
```console
$ rm -rf ${LINDAREX_BOSH_WORKSPACE}/bosh-deployment/${LINDAREX_BOSH_IAAS}/state.json
$ rm -rf ${LINDAREX_BOSH_WORKSPACE}/bosh-deployment/${LINDAREX_BOSH_IAAS}/creds.yml
```

> state.json 파일과 creds.yml 파일은 배포 성공 여부와 관계없이 'bosh create-env' 명령어를 한 번이라도 실행하면 생성되는 파일입니다.

#### 5.5. 배포 스크립트 실행
```console
$ cd ${LINDAREX_BOSH_WORKSPACE}/bosh-deployment
$ ./deploy-${LINDAREX_BOSH_IAAS}.sh
```

### 6. BOSH UAA 통합 인증
#### 6.1. BOSH UAA target 설정
```console
$ uaac target https://10.0.1.6:8443 --skip-ssl-validation
Unknown key: Max-Age = 86400

Target: https://10.0.1.6:8443

```

#### 6.2. Client credentials grant를 통해 UAA admin token 얻기
```console
$ uaac token client get uaa_admin -s `bosh int ${LINDAREX_BOSH_WORKSPACE}/bosh-deployment/${LINDAREX_BOSH_IAAS}/creds.yml --path /uaa_admin_client_secret`
Unknown key: Max-Age = 86400

Successfully fetched token via client credentials grant.
Target: https://10.0.1.6:8443
Context: uaa_admin, from client uaa_admin

```

#### 6.3. UAA admin token 파싱 및 clients 목록 조회
```console
$ uaac token decode && uaac clients

Note: no key given to validate token signature

  jti: 1b82b1e6cfbb4606a182d5ab88c28b8d
  sub: uaa_admin
  authorities: clients.read password.write clients.secret clients.write uaa.admin scim.write scim.read
  scope: clients.read password.write clients.secret clients.write uaa.admin scim.write scim.read
  client_id: uaa_admin
  cid: uaa_admin
  azp: uaa_admin
  revocable: true
  grant_type: client_credentials
  rev_sig: 34bac1ad
  iat: 1573525863
  exp: 1573569063
  iss: https://10.0.1.6:8443/oauth/token
  zid: uaa
  aud: scim uaa_admin password clients uaa
  admin
    scope: uaa.none
    resource_ids: none
    authorized_grant_types: client_credentials
    autoapprove:
    authorities: bosh.admin
    lastmodified: 1573525068032
  bosh_cli
    scope: openid bosh.admin bosh.read bosh.*.admin bosh.*.read bosh.teams.*.admin bosh.teams.*.read
    resource_ids: none
    authorized_grant_types: password refresh_token
    autoapprove:
    access_token_validity: 120
    refresh_token_validity: 86400
    authorities: uaa.none
    lastmodified: 1573525068153
  credhub-admin
    scope: uaa.none
    resource_ids: none
    authorized_grant_types: client_credentials
    autoapprove:
    access_token_validity: 3600
    authorities: credhub.write credhub.read
    lastmodified: 1573525068271
  credhub_cli
    scope: credhub.read credhub.write
    resource_ids: none
    authorized_grant_types: password refresh_token
    autoapprove:
    access_token_validity: 60
    refresh_token_validity: 1800
    authorities: uaa.none
    lastmodified: 1573525068386
  director_to_credhub
    scope: uaa.none
    resource_ids: none
    authorized_grant_types: client_credentials
    autoapprove:
    access_token_validity: 3600
    authorities: credhub.write credhub.read
    lastmodified: 1573525068492
  hm
    scope: uaa.none
    resource_ids: none
    authorized_grant_types: client_credentials
    autoapprove:
    authorities: bosh.admin
    lastmodified: 1573525068590
  uaa_admin
    scope: uaa.none
    resource_ids: none
    authorized_grant_types: client_credentials
    autoapprove:
    authorities: clients.read password.write clients.secret clients.write uaa.admin scim.write scim.read
    lastmodified: 1573525068689
```

### 7. BOSH director 설정
#### 7.1. Local alias 설정
```console
$ export BOSH_CLIENT='admin'
$ export BOSH_CLIENT_SECRET=`bosh int ${LINDAREX_BOSH_WORKSPACE}/bosh-deployment/${LINDAREX_BOSH_IAAS}/creds.yml --path /admin_password`
$ bosh alias-env ${LINDAREX_BOSH_DIRECTOR} -e 10.0.1.6 --ca-cert <(bosh int ${LINDAREX_BOSH_WORKSPACE}/bosh-deployment/${LINDAREX_BOSH_IAAS}/creds.yml --path /director_ssl/ca)
```

#### 7.2. BOSH 로그인
```console
$ bosh -e ${LINDAREX_BOSH_DIRECTOR} l
Successfully authenticated with UAA

Succeeded
```

#### 7.3. BOSH director 정보 조회
```console
$ bosh -e ${LINDAREX_BOSH_DIRECTOR} env
Using environment '10.0.1.6' as client 'admin'

Name               micro-bosh
UUID               b002986f-3a05-441a-be08-3a46dbe29a8a
Version            270.5.0 (00000000)
Director Stemcell  ubuntu-xenial/456.27
CPI                aws_cpi
Features           compiled_package_cache: disabled
                   config_server: enabled
                   local_dns: enabled
                   power_dns: disabled
                   snapshots: disabled
User               admin

Succeeded
```

#### 7.4. (선택사항) 사용자 프로필에 BOSH director 정보 추가
```console
$ vi $HOME/.profile
```

```shell
--------------------------------------------------------------------------------
export LINDAREX_INCEPTION_USER_NAME='ubuntu'
export LINDAREX_BOSH_WORKSPACE=/home/${LINDAREX_INCEPTION_USER_NAME}/workspace
export LINDAREX_BOSH_DIRECTOR='micro-bosh'
export LINDAREX_BOSH_IAAS='aws'
export BOSH_ENVIRONMENT='10.0.1.6'
export BOSH_CLIENT='admin'
export BOSH_CLIENT_SECRET=`bosh -e ${LINDAREX_BOSH_DIRECTOR} int ${LINDAREX_BOSH_WORKSPACE}/bosh-deployment/${LINDAREX_BOSH_IAAS}/creds.yml --path /admin_password`
--------------------------------------------------------------------------------
```

> 수정 내역 적용을 위해 아래 명령어를 입력합니다.
```console
$ source $HOME/.profile
```

### 8. BOSH jumpbox 설정
#### 8.1. Jumpbox key 생성
```console
$ cd ${LINDAREX_BOSH_WORKSPACE}/bosh-deployment
$ bosh int ${LINDAREX_BOSH_IAAS}/creds.yml --path /jumpbox_ssh/private_key > jumpbox.key
```

#### 8.2. chmod 명령어로 파일 권한 변경(소유자만 읽기 및 쓰기 권한 부여)
```console
$ chmod 600 jumpbox.key
```

#### 8.3. BOSH director VM에 SSH 접속
```console
$ ssh jumpbox@${BOSH_ENVIRONMENT} -i ${LINDAREX_BOSH_WORKSPACE}/bosh-deployment/jumpbox.key
The authenticity of host '10.0.1.6 (10.0.1.6)' can't be established.
ECDSA key fingerprint is SHA256:Lc0OsEocqPAEgAk0c1X7Y7y+iNWqeFMGkfFFLRlA8ww.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.0.1.6' (ECDSA) to the list of known hosts.
Unauthorized use is strictly prohibited. All access and activity
is subject to logging and monitoring.
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-54-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Tue Feb 18 07:23:17 2020 from 10.0.201.149
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

bosh/0:~$ 
```

#### 8.4. BOSH director VM의 외부 통신 상태 확인
```console
bosh/0:~$ sudo ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=53 time=30.8 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=53 time=29.1 ms
^C
--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 29.135/30.009/30.884/0.891 ms
```

```console
bosh/0:~$ wget https://wordpress.org/latest.zip
```

> 아래 명령어로 BOSH director VM의 SSH 접속을 종료할 수 있습니다.
```console
bosh/0:~$ exit
logout
Connection to 10.0.1.6 closed.
```

### 9. Credhub 설정
#### 9.1. Credhub api server 설정
```console
$ credhub api \
--ca-cert=<(bosh int ${LINDAREX_BOSH_WORKSPACE}/bosh-deployment/${LINDAREX_BOSH_IAAS}/creds.yml --path /credhub_ca/ca) \
--ca-cert=<(bosh int ${LINDAREX_BOSH_WORKSPACE}/bosh-deployment/${LINDAREX_BOSH_IAAS}/creds.yml --path /uaa_ssl/ca) \
--server=10.0.1.6:8844
Setting the target url: https://10.0.1.6:8844
```

#### 9.2. Credhub 로그인
```console
$ credhub login --client-name=credhub-admin --client-secret=`bosh int ${LINDAREX_BOSH_WORKSPACE}/bosh-deployment/${LINDAREX_BOSH_IAAS}/creds.yml --path /credhub_admin_client_secret`
Login Successful
```

#### 9.3. credentials 전체 조회
```console
$ credhub find
credentials: []
```


## 마무리(CONCLUSION)
AWS EC2 Ubuntu 환경에 BOSH director 설치를 완료했습니다. <br />
BOSH director 설치는 CFAR 및 CFCR 등 BOSH release 배포를 위한 기반 작업이며, 설치 진행을 위해서는 적지 않은 IaaS 설정이 필요합니다. <br />
다음 포스트에서는 BOSH director 설치를 위한 AWS 환경 설정과 AWS EC2 Inception VM 생성 방법, BOSH components에 대한 설명과 BOSH release 작성 방법을 소개하겠습니다.


## 참고(REFERENCES)
- [https://bosh.io/docs/cli-v2-install/](https://bosh.io/docs/cli-v2-install/){: target="\_blank"}
- [https://github.com/cloudfoundry/bosh-cli/releases](https://github.com/cloudfoundry/bosh-cli/releases){: target="\_blank"}
- [https://docs.cloudfoundry.org/cf-cli/install-go-cli.html](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html){: target="\_blank"}
- [https://github.com/cloudfoundry/cf-uaac](https://github.com/cloudfoundry/cf-uaac){: target="\_blank"}
- [https://github.com/cloudfoundry-incubator/credhub-cli](https://github.com/cloudfoundry-incubator/credhub-cli){: target="\_blank"}

