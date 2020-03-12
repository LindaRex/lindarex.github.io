---
title: "우분투(Ubuntu) 환경에 패키지(Package)로 Azure CLI 설치하기"
categories: 
  - azure
tags: 
  - azure
  - cli
  - ubuntu
---


Azure CLI(command line interface, 명령줄 인터페이스)는 Microsoft Azure에 리소스(resource)를 만들고 관리할 수 도구입니다. <br />
모든 azure 서비스(service)에서 사용할 수 있으며, azure를 빠르게 사용할 수 있도록 자동화에 초점을 두고 있습니다. <br />
이 포스트에서는 우분투(ubuntu) 환경에서 패키지(package)로 azure cli를 설치하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- ubuntu 환경이 필요합니다.

> Ubuntu 설치 방법은 [VMware workstation에 ubuntu server 16.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1604-installation/){: target="\_blank"} 또는 [VMware workstation에 ubuntu server 18.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1804-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.1 build-15018445)
- Ubuntu 18.04.3 LTS (Bionic Beaver) Server (64-bit)


## 요약(SUMMARY)
1. azure cli debian packages repository 설정
2. apt 명령어로 azure cli 설치
3. azure cli 로그인
4. (선택사항) apt 명령어로 azure cli 삭제


## 내용(CONTENTS)
### 1. 종속 package 설치

> Ubuntu 18.04 (Bionic)

```console
$ sudo apt update
$ sudo apt install -y ca-certificates curl apt-transport-https lsb-release gnupg
```

### 2. Microsoft 서명 키 설치
```console
$ curl -sL https://packages.microsoft.com/keys/microsoft.asc | \
    gpg --dearmor | \
    sudo tee /etc/apt/trusted.gpg.d/microsoft.asc.gpg > /dev/null
```

### 3. azure cli debian packages repository 추가
```console
$ AZ_REPO=$(lsb_release -cs)
$ echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | \
    sudo tee /etc/apt/sources.list.d/azure-cli.list
```

### 4. azure cli 설치
```console
$ sudo apt update
$ sudo apt install azure-cli -y
```

### 5. azure 로그인
```console
$ az login
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code HGCW1YCF3 to authenticate.


[
  {
    "cloudName": "AzureCloud",
    "id": "cd85e66d-8955-67b9-cdb5-cb6b706eb6b9",
    "isDefault": true,
    "name": "Microsoft Azure-LindaRex",
    "state": "Enabled",
    "tenantId": "2a755963-e627-665a-b039-ba6576750ae1",
    "user": {
      "name": "lindarex@lindarex.github.io",
      "type": "user"
    }
  }
]
```

> 위 로그인 결과는 잘못된 값이므로 참고만 하시기를 바랍니다.

### 6. (선택사항) azure cli 삭제
```console
$ sudo apt remove azure-cli -y
$ sudo apt autoremove -y
$ sudo rm /etc/apt/sources.list.d/azure-cli.list
$ sudo rm /etc/apt/trusted.gpg.d/microsoft.asc.gpg
```


## 마무리(CONCLUSION)
ubuntu 환경에 package로 azure cli 설치를 완료했습니다. <br />
다음 포스트에서는 azure 사용을 위한 service principal, resource group, network security group, virtual network, subnet, storage account 등의 설정과 Inception VM 생성 방법을 소개하겠습니다.


## 참고(REFERENCES)
- [https://docs.microsoft.com/ko-kr/cli/azure/?view=azure-cli-latest](https://docs.microsoft.com/ko-kr/cli/azure/?view=azure-cli-latest){: target="\_blank"}
