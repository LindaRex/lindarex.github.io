---
title: "우분투(Ubuntu) 서버(Server) 18.04 설치하기"
categories: 
  - ubuntu
tags: 
  - ubuntu
  - "18.04"
  - "vmware workstation"
  - "open source software"
  - "open source"
  - oss
---


우분투(ubuntu) server 18.04는 ubuntu server 16.04와 비교해 새로운 기능과 개선 사항으로 많은 변화가 있으며, 릴리스의 코드 이름은 'Bionic Beaver'입니다.
<br /><br />
이 포스트에서는 VMware Workstation에 ubuntu server 18.04 LTS를 설치하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- VMware Workstation이 설치되어 있어야 합니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.1 build-15018445)
- Ubuntu 18.04.3 LTS (Bionic Beaver) Server (64-bit)

> ubuntu server 18.04 설치 시 필요한 시스템 요구사항은 [https://help.ubuntu.com/lts/serverguide/preparing-to-install.html](https://help.ubuntu.com/lts/serverguide/preparing-to-install.html){: target="\_blank"}를 확인해 주시기 바랍니다.


## 요약(SUMMARY)
1. ubuntu server 18.04.3 ISO 파일 내려받기
2. VMware workstation의 VM(virtual machine) 설정
3. ubuntu server 18.04.3 설치
4. 리눅스 명령어로 ubuntu VM 상태 확인


## 내용(CONTENTS)
### 1. ubuntu server 18.04.3 ISO 파일 내려받기
- 웹브라우저로 [ubuntu Releases 페이지](http://mirror.kakao.com/ubuntu-releases/){: target="\_blank"}를 엽니다.

![lindarex-ubuntu-1804-installation-001]

- 목록의 '18.04.3'을 클릭하여 ubuntu 18.04.3 LTS (Bionic Beaver) 페이지로 이동합니다.

![lindarex-ubuntu-1804-installation-002]

- Server install image 영역의 '64-bit PC (AMD64) server install image'를 클릭하여 ISO 파일을 내려받습니다.

> [http://mirror.kakao.com/ubuntu-releases/18.04.3/ubuntu-18.04.3-live-server-amd64.iso](http://mirror.kakao.com/ubuntu-releases/18.04.3/ubuntu-18.04.3-live-server-amd64.iso){: target="\_blank"}를 통해 바로 내려받을 수 있습니다. <br />
> ubuntu-18.04.3-live-server-amd64.iso 파일 사이즈는 약 868 MB입니다.

![lindarex-ubuntu-1804-installation-003]

### 2. VMware workstation의 VM(Virtual Machine) 설정
- VMware workstation을 실행합니다.
- 아래 메뉴를 통해 'New Virtual Machine Wizard' 팝업을 엽니다.

> 상단 메뉴 > File > 'New Virtual Machine...' 또는 Ctrl + N

![lindarex-ubuntu-1804-installation-004]

- 상세 설정을 위해 'Custom (advanced)'를 선택하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1804-installation-005]

- VMware Workstation 버전을 선택하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1804-installation-006]

- 상세 설정을 위해 'I will install the operating system later.'를 선택하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1804-installation-007]

- Guest operating system은 'Linux', Version은 'ubuntu 64-bit'를 선택하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1804-installation-008]

- Virtual machine name을 입력하고, Location에 vmdk 파일이 저장될 경로를 입력하거나 'Browse..'를 클릭해 위치를 선택하고 'Next'를 클릭합니다.

> VMDK란 Virtual Machine Disk의 약자이며, 자세한 정보는 [https://en.wikipedia.org/wiki/VMDK](https://en.wikipedia.org/wiki/VMDK){: target="\_blank"}를 참고하시기 바랍니다.

![lindarex-ubuntu-1804-installation-009]

- processors '1', cores per processor '1'을 입력하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1804-installation-010]

- 권장값인 '2048'MB를 입력하고 'Next'를 클릭합니다.

> 권장값은 자신의 PC 메모리(Host memory)에 따라 다를 수 있습니다.

![lindarex-ubuntu-1804-installation-011]

- 'Use network address translation (NAT)'를 선택하고 'Next'를 클릭합니다.

> Network type에 대한 자세한 정보는 [VMware Workstation의 가상 네트워크(Virtual Network) 알아보기](https://lindarex.github.io/vmware-workstation/vmware-workstation-virtual-network/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

![lindarex-ubuntu-1804-installation-012]

- 권장값인 'LSI Logic (Recommended)'를 선택하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1804-installation-013]

- 권장값인 'SCSI Logic (Recommended)'를 선택하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1804-installation-014]

- 'Create a new virtual disk'를 선택하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1804-installation-015]

- Maximum disk size를 '20.0'을 입력하고 'Split virtual disk into multiple files'를 선택하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1804-installation-016]

- Disk file 명을 입력하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1804-installation-017]

- 'Customize Hardware..'를 클릭하여 Hardware 팝업을 엽니다.

![lindarex-ubuntu-1804-installation-018]

- 앞에서 설정한 내용을 확인합니다. (Memory, Processors)

![lindarex-ubuntu-1804-installation-019]

![lindarex-ubuntu-1804-installation-020]

- Use ISO image file 항목의 'Browse..'를 클릭해 내려받은 ubuntu 18.04.3 LTS (Bionic Beaver) ISO 파일을 선택합니다.

![lindarex-ubuntu-1804-installation-021]

- 앞에서 설정한 내용을 확인합니다. (Network Adapter, USB Controller, Sound Card, Printer, Display)

![lindarex-ubuntu-1804-installation-022]

![lindarex-ubuntu-1804-installation-023]

![lindarex-ubuntu-1804-installation-024]

![lindarex-ubuntu-1804-installation-025]

![lindarex-ubuntu-1804-installation-026]

- 모든 설정을 확인하고 'Finish'를 클릭합니다.

![lindarex-ubuntu-1804-installation-027]

### 3. ubuntu server 18.04.3 설치
- 아래 메뉴를 통해 ubuntu VM을 구동합니다.

> 상단 메뉴 > 녹색 플레이 버튼 또는 Ctrl + B

![lindarex-ubuntu-1804-installation-028]

- 설치가 진행됩니다.

![lindarex-ubuntu-1804-installation-029]

- 사용하고자 하는 언어를 선택합니다.

![lindarex-ubuntu-1804-installation-030]

- Keyboard layout을 설정합니다. 'English (US)'를 선택합니다.

![lindarex-ubuntu-1804-installation-031]

- Network를 설정합니다.

> VMware Workstation에 Ubuntu를 설치할 경우, 자동으로 DHCP(Dynamic Host Configuration Protocol)로 설정됩니다. <br />
> DHCP에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/동적_호스트_구성_프로토콜](https://ko.wikipedia.org/wiki/%EB%8F%99%EC%A0%81_%ED%98%B8%EC%8A%A4%ED%8A%B8_%EA%B5%AC%EC%84%B1_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C){: target="\_blank"}를 참고하시기 바랍니다.

![lindarex-ubuntu-1804-installation-032]

- Proxy를 설정합니다.

![lindarex-ubuntu-1804-installation-033]

- ubuntu archive mirror address를 설정합니다.

![lindarex-ubuntu-1804-installation-034]

- Filesystem을 설정합니다. 'Use An Entire Disk and Set Up LVM'을 선택합니다.

![lindarex-ubuntu-1804-installation-035]

![lindarex-ubuntu-1804-installation-036]

![lindarex-ubuntu-1804-installation-037]

![lindarex-ubuntu-1804-installation-038]

- 사용자 프로필을 설정합니다.

![lindarex-ubuntu-1804-installation-039]

- SSH 설치 여부를 선택합니다.

> Xshell을 비롯해 SSH 클라이언트 사용을 위해 OpenSSH server 패키지를 설치합니다.

![lindarex-ubuntu-1804-installation-040]

- 추가로 설치할 패키지를 선택합니다. 

![lindarex-ubuntu-1804-installation-041]

- 설치가 진행됩니다.

![lindarex-ubuntu-1804-installation-042]

![lindarex-ubuntu-1804-installation-043]

- 설치가 완료되었습니다. 'Reboot'를 선택합니다.

![lindarex-ubuntu-1804-installation-044]

- 재부팅됩니다.

![lindarex-ubuntu-1804-installation-045]

- 정상 설치 후 로그인 프롬프트 상태입니다.

![lindarex-ubuntu-1804-installation-046]

### 4. 리눅스 명령어(command)로 ubuntu VM 상태 확인

- 사용자 이름과 비밀번호를 입력하여 로그인합니다.

![lindarex-ubuntu-1804-installation-047]

- free command로 메모리 상태를 확인합니다.

![lindarex-ubuntu-1804-installation-048]

- df command로 디스크 사용량을 확인합니다.

![lindarex-ubuntu-1804-installation-049]

- ifconfig command로 네트워크 정보를 확인합니다.

![lindarex-ubuntu-1804-installation-050]

- ping command로 네트워크 상태를 확인합니다.

![lindarex-ubuntu-1804-installation-051]

- 'apt update' command로 패키지 인덱스를 업데이트합니다.

> 'apt update' command는 업그레이드 가능한 패키지 정보를 업데이트하는 것이며, 실제로 패키지를 업데이트하지 않습니다.

![lindarex-ubuntu-1804-installation-052]

- 'apt upgrade' command로 업그레이드 가능한 모든 패키지를 업데이트합니다.

![lindarex-ubuntu-1804-installation-053]

![lindarex-ubuntu-1804-installation-054]


## 마무리(CONCLUSION)
VMware Workstation에 Ubuntu 18.04.3 LTS (Bionic Beaver) Server 설치를 완료했습니다.
<br />
다음 포스트에서는 [우분투(Ubuntu) 서버(Server) 초기 설정하기](https://lindarex.github.io/ubuntu/ubuntu-initial-setting/){: target="\_blank"}를 소개하겠습니다.


## 참고(REFERENCES)
- [https://ubuntu.com/](https://ubuntu.com/){: target="\_blank"}
- [http://mirror.kakao.com/ubuntu-releases/](http://mirror.kakao.com/ubuntu-releases/){: target="\_blank"}
- [https://ubuntu.com/tutorials/tutorial-install-ubuntu-server#1-overview](https://ubuntu.com/tutorials/tutorial-install-ubuntu-server#1-overview){: target="\_blank"}
- [https://ko.wikipedia.org/wiki/우분투_(운영_체제)](https://ko.wikipedia.org/wiki/%EC%9A%B0%EB%B6%84%ED%88%AC_(%EC%9A%B4%EC%98%81_%EC%B2%B4%EC%A0%9C)){: target="\_blank"}



[lindarex-ubuntu-1804-installation-001]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-001.png
[lindarex-ubuntu-1804-installation-002]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-002.png
[lindarex-ubuntu-1804-installation-003]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-003.png
[lindarex-ubuntu-1804-installation-004]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-004.png
[lindarex-ubuntu-1804-installation-005]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-005.png
[lindarex-ubuntu-1804-installation-006]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-006.png
[lindarex-ubuntu-1804-installation-007]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-007.png
[lindarex-ubuntu-1804-installation-008]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-008.png
[lindarex-ubuntu-1804-installation-009]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-009.png
[lindarex-ubuntu-1804-installation-010]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-010.png
[lindarex-ubuntu-1804-installation-011]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-011.png
[lindarex-ubuntu-1804-installation-012]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-012.png
[lindarex-ubuntu-1804-installation-013]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-013.png
[lindarex-ubuntu-1804-installation-014]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-014.png
[lindarex-ubuntu-1804-installation-015]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-015.png
[lindarex-ubuntu-1804-installation-016]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-016.png
[lindarex-ubuntu-1804-installation-017]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-017.png
[lindarex-ubuntu-1804-installation-018]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-018.png
[lindarex-ubuntu-1804-installation-019]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-019.png
[lindarex-ubuntu-1804-installation-020]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-020.png
[lindarex-ubuntu-1804-installation-021]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-021.png
[lindarex-ubuntu-1804-installation-022]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-022.png
[lindarex-ubuntu-1804-installation-023]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-023.png
[lindarex-ubuntu-1804-installation-024]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-024.png
[lindarex-ubuntu-1804-installation-025]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-025.png
[lindarex-ubuntu-1804-installation-026]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-026.png
[lindarex-ubuntu-1804-installation-027]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-027.png
[lindarex-ubuntu-1804-installation-028]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-028.png
[lindarex-ubuntu-1804-installation-029]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-029.png
[lindarex-ubuntu-1804-installation-030]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-030.png
[lindarex-ubuntu-1804-installation-031]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-031.png
[lindarex-ubuntu-1804-installation-032]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-032.png
[lindarex-ubuntu-1804-installation-033]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-033.png
[lindarex-ubuntu-1804-installation-034]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-034.png
[lindarex-ubuntu-1804-installation-035]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-035.png
[lindarex-ubuntu-1804-installation-036]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-036.png
[lindarex-ubuntu-1804-installation-037]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-037.png
[lindarex-ubuntu-1804-installation-038]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-038.png
[lindarex-ubuntu-1804-installation-039]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-039.png
[lindarex-ubuntu-1804-installation-040]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-040.png
[lindarex-ubuntu-1804-installation-041]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-041.png
[lindarex-ubuntu-1804-installation-042]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-042.png
[lindarex-ubuntu-1804-installation-043]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-043.png
[lindarex-ubuntu-1804-installation-044]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-044.png
[lindarex-ubuntu-1804-installation-045]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-045.png
[lindarex-ubuntu-1804-installation-046]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-046.png
[lindarex-ubuntu-1804-installation-047]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-047.png
[lindarex-ubuntu-1804-installation-048]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-048.png
[lindarex-ubuntu-1804-installation-049]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-049.png
[lindarex-ubuntu-1804-installation-050]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-050.png
[lindarex-ubuntu-1804-installation-051]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-051.png
[lindarex-ubuntu-1804-installation-052]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-052.png
[lindarex-ubuntu-1804-installation-053]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-053.png
[lindarex-ubuntu-1804-installation-054]:/assets/images/2020-02-11-ubuntu-1804-installation/lindarex-ubuntu-1804-installation-054.png
