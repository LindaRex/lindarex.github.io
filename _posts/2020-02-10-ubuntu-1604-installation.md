---
title: "우분투(Ubuntu) 서버(Server) 16.04 설치하기"
categories: 
  - ubuntu
tags: 
  - ubuntu
  - "16.04"
  - "vmware workstation"
---


우분투(ubuntu)는 데비안(debian) 리눅스(linux) 기반으로, debian에 비해 사용자 편의성에 초점을 맞춘 linux 배포판이며 컴퓨터 운영체제(OS, operating system)입니다. <br />
새로운 버전은 6개월마다, 장기 지원 버전(LTS, long term support)은 2년에 한 번씩 출시되고, 다양한 언어를 지원하고 낮은 사양의 컴퓨터에서도 작동하도록 설계되어 있습니다. <br />
이 포스트에서는 VMware Workstation에 ubuntu server 16.04 LTS를 설치하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- VMware Workstation이 설치되어 있어야 합니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.1 build-15018445)
- Ubuntu 16.04.6 LTS (Xenial Xerus) Server (64-bit)

> ubuntu server 16.04 설치 시 필요한 시스템 요구사항은 [https://help.ubuntu.com/16.04/serverguide/preparing-to-install.html](https://help.ubuntu.com/16.04/serverguide/preparing-to-install.html){: target="\_blank"}를 확인해 주시기 바랍니다.


## 요약(SUMMARY)
1. ubuntu server 16.04.6 ISO 파일 내려받기
2. VMware workstation의 VM(virtual machine) 설정
3. ubuntu server 16.04.6 설치
4. linux 명령어로 ubuntu VM 상태 확인


## 내용(CONTENTS)
### 1. ubuntu server 16.04.6 ISO 파일 내려받기
- 웹브라우저로 [ubuntu Releases 페이지](http://mirror.kakao.com/ubuntu-releases/){: target="\_blank"}를 엽니다.

![lindarex-ubuntu-1604-installation-001]

- 목록의 '16.04.6'를 클릭하여 ubuntu 16.04.6 LTS (Xenial Xerus) 페이지로 이동합니다.

![lindarex-ubuntu-1604-installation-002]

- Server install image 영역의 '64-bit PC (AMD64) server install image'를 클릭하여 ISO 파일을 내려받습니다.

> [http://mirror.kakao.com/ubuntu-releases/16.04.6/ubuntu-16.04.6-server-amd64.iso](http://mirror.kakao.com/ubuntu-releases/16.04.6/ubuntu-16.04.6-server-amd64.iso){: target="\_blank"}를 통해 바로 내려받을 수 있습니다. <br />
> ubuntu-16.04.6-server-amd64.iso 파일 사이즈는 약 893 MB입니다.

![lindarex-ubuntu-1604-installation-003]

### 2. VMware workstation의 VM(Virtual Machine) 설정
- VMware workstation을 실행합니다.
- 아래 메뉴를 통해 'New Virtual Machine Wizard' 팝업을 엽니다.

> 상단 메뉴 > File > 'New Virtual Machine...' 또는 Ctrl + N

![lindarex-ubuntu-1604-installation-004]

- 상세 설정을 위해 'Custom (advanced)'를 선택하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1604-installation-005]

- VMware Workstation 버전을 선택하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1604-installation-006]

- 상세 설정을 위해 'I will install the operating system later.'를 선택하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1604-installation-007]

- Guest operating system은 'Linux', Version은 'ubuntu 64-bit'를 선택하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1604-installation-008]

- Virtual machine name을 입력하고, Location에 vmdk 파일이 저장될 경로를 입력하거나 'Browse..'를 클릭해 위치를 선택하고 'Next'를 클릭합니다.

> VMDK란 Virtual Machine Disk의 약자이며, 자세한 정보는 [https://en.wikipedia.org/wiki/VMDK](https://en.wikipedia.org/wiki/VMDK){: target="\_blank"}를 참고하시기 바랍니다.

![lindarex-ubuntu-1604-installation-009]

- processors '1', cores per processor '1'을 입력하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1604-installation-010]

- 권장값인 '2048'MB를 입력하고 'Next'를 클릭합니다.

> 권장값은 자신의 PC 메모리(Host memory)에 따라 다를 수 있습니다.

![lindarex-ubuntu-1604-installation-011]

- 'Use network address translation (NAT)'를 선택하고 'Next'를 클릭합니다.

> Network type에 대한 자세한 정보는 [VMware Workstation의 가상 네트워크(Virtual Network) 알아보기](https://lindarex.github.io/vmware-workstation/vmware-workstation-virtual-network/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


![lindarex-ubuntu-1604-installation-012]

- 권장값인 'LSI Logic (Recommended)'를 선택하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1604-installation-013]

- 권장값인 'SCSI Logic (Recommended)'를 선택하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1604-installation-014]

- 'Create a new virtual disk'를 선택하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1604-installation-015]

- Maximum disk size를 '20.0'을 입력하고 'Split virtual disk into multiple files'를 선택하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1604-installation-016]

- Disk file 명을 입력하고 'Next'를 클릭합니다.

![lindarex-ubuntu-1604-installation-017]

- 'Customize Hardware..'를 클릭하여 Hardware 팝업을 엽니다.

![lindarex-ubuntu-1604-installation-018]

- 앞에서 설정한 내용을 확인합니다. (Memory, Processors)

![lindarex-ubuntu-1604-installation-019]

![lindarex-ubuntu-1604-installation-020]

- Use ISO image file 항목의 'Browse..'를 클릭해 내려받은 ubuntu 16.04.6 LTS (Xenial Xerus) ISO 파일을 선택합니다.

![lindarex-ubuntu-1604-installation-021]

- 앞에서 설정한 내용을 확인합니다. (Network Adapter, USB Controller, Sound Card, Printer, Display)

![lindarex-ubuntu-1604-installation-022]

![lindarex-ubuntu-1604-installation-023]

![lindarex-ubuntu-1604-installation-024]

![lindarex-ubuntu-1604-installation-025]

![lindarex-ubuntu-1604-installation-026]

- 모든 설정을 확인하고 'Finish'를 클릭합니다.

![lindarex-ubuntu-1604-installation-027]

### 3. ubuntu server 16.04.6 설치
- 아래 메뉴를 통해 ubuntu VM을 구동합니다.

> 상단 메뉴 > 녹색 플레이 버튼 또는 Ctrl + B

![lindarex-ubuntu-1604-installation-028]

- 설치할 때 사용하고자 하는 언어를 선택합니다.

![lindarex-ubuntu-1604-installation-029]

- 'Install ubuntu Server'를 선택하여 설치를 시작합니다.

![lindarex-ubuntu-1604-installation-030]

- ubuntu Server에 사용하고자 하는 언어를 설정합니다. 'English'를 선택합니다.

![lindarex-ubuntu-1604-installation-031]

- Location을 설정합니다. 'other'를 선택한 후, 'Asia'의 'Korea, Republic of'를 선택합니다.

![lindarex-ubuntu-1604-installation-032]

![lindarex-ubuntu-1604-installation-033]

![lindarex-ubuntu-1604-installation-034]

- Locale을 설정합니다. 'United States'를 선택합니다.

![lindarex-ubuntu-1604-installation-035]

- Keyboard layout을 설정합니다. 'No'를 선택한 후, 'English (US)'를 선택합니다.

![lindarex-ubuntu-1604-installation-036]

![lindarex-ubuntu-1604-installation-037]

![lindarex-ubuntu-1604-installation-038]

- 설치가 진행됩니다.

![lindarex-ubuntu-1604-installation-039]

- 호스트네임(Hostname)을 입력합니다.

![lindarex-ubuntu-1604-installation-040]

- 사용자 이름(User name)을 입력합니다.

![lindarex-ubuntu-1604-installation-041]

![lindarex-ubuntu-1604-installation-042]

- 비밀번호(Password)를 입력합니다.

![lindarex-ubuntu-1604-installation-043]

![lindarex-ubuntu-1604-installation-044]

- Home directory의 암호화를 설정합니다. 'No'를 선택합니다.

![lindarex-ubuntu-1604-installation-045]

- 설치가 진행됩니다.

![lindarex-ubuntu-1604-installation-046]

- 타임존(Time zone)을 설정합니다.

![lindarex-ubuntu-1604-installation-047]

- Disk partition을 설정합니다. 'Guided - use entire disk and set up LVM'을 선택합니다.

![lindarex-ubuntu-1604-installation-048]

![lindarex-ubuntu-1604-installation-049]

![lindarex-ubuntu-1604-installation-050]

![lindarex-ubuntu-1604-installation-051]

![lindarex-ubuntu-1604-installation-052]

- 설치가 진행됩니다.

![lindarex-ubuntu-1604-installation-053]

- HTTP proxy를 설정합니다.

![lindarex-ubuntu-1604-installation-054]

- 설치가 진행됩니다.

![lindarex-ubuntu-1604-installation-055]

- Upgrade 주기를 설정합니다.

![lindarex-ubuntu-1604-installation-056]

- 추가로 설치할 패키지를 선택합니다. 

> Xshell을 비롯해 SSH 클라이언트 사용을 위해 OpenSSH server 패키지를 추가합니다.

![lindarex-ubuntu-1604-installation-057]

- 설치가 진행됩니다.

![lindarex-ubuntu-1604-installation-058]

- GRUB boot loader를 설정합니다. 'Yes'를 선택합니다.

![lindarex-ubuntu-1604-installation-059]

- 설치가 진행됩니다.

![lindarex-ubuntu-1604-installation-060]

- 설치 과정의 마지막입니다. 'Continue'를 선택합니다.

![lindarex-ubuntu-1604-installation-061]

- 재부팅됩니다.

![lindarex-ubuntu-1604-installation-062]

- 정상 설치 후 로그인 프롬프트 상태입니다.

![lindarex-ubuntu-1604-installation-063]

### 4. linux 명령어(command)로 ubuntu VM 상태 확인

- 사용자 이름과 비밀번호를 입력하여 로그인합니다.

![lindarex-ubuntu-1604-installation-064]

- free command로 메모리 상태를 확인합니다.

![lindarex-ubuntu-1604-installation-065]

- df command로 디스크 사용량을 확인합니다.

![lindarex-ubuntu-1604-installation-066]

- ifconfig command로 네트워크 정보를 확인합니다.

![lindarex-ubuntu-1604-installation-067]

- ping command로 네트워크 상태를 확인합니다.

![lindarex-ubuntu-1604-installation-068]

- 'apt update' command로 패키지 인덱스를 업데이트합니다.

> 'apt update' command는 업그레이드 가능한 패키지 정보를 업데이트하는 것이며, 실제로 패키지를 업데이트하지 않습니다.

![lindarex-ubuntu-1604-installation-069]

- 'apt upgrade' command로 업그레이드 가능한 모든 패키지를 업데이트합니다.

![lindarex-ubuntu-1604-installation-070]

![lindarex-ubuntu-1604-installation-071]

![lindarex-ubuntu-1604-installation-072]


## 마무리(CONCLUSION)
VMware Workstation에 Ubuntu 16.04.6 LTS (Xenial Xerus) Server 설치를 완료했습니다. <br />
2020년 2월 12일 [W3Techs.com 통계](https://w3techs.com/technologies/details/os-linux){: target="\_blank"}에 따르면, ubuntu 점유율은 linux 배포판을 사용하는 웹사이트 중 1위(38.9%)로, 2위인 debian(18.4%)의 2배 이상 차이가 납니다. 또한 대부분의 오픈소스 소프트웨어는 Ubuntu를 지원하고, 국내외 사용자와 레퍼런스가 많습니다. <br />
linux 입문자 또는 오픈소스 소프트웨어에 관심이 있다면, Ubuntu를 사용하며 linux command를 숙지하고 다양한 커뮤니티에서 활동하는 것을 권합니다.


## 참고(REFERENCES)
- [https://ubuntu.com/](https://ubuntu.com/){: target="\_blank"}
- [http://mirror.kakao.com/ubuntu-releases/](http://mirror.kakao.com/ubuntu-releases/){: target="\_blank"}
- [https://ubuntu.com/tutorials/tutorial-install-ubuntu-server#1-overview](https://ubuntu.com/tutorials/tutorial-install-ubuntu-server#1-overview){: target="\_blank"}
- [https://ko.wikipedia.org/wiki/우분투_(운영_체제)](https://ko.wikipedia.org/wiki/%EC%9A%B0%EB%B6%84%ED%88%AC_(%EC%9A%B4%EC%98%81_%EC%B2%B4%EC%A0%9C)){: target="\_blank"}


[lindarex-ubuntu-1604-installation-001]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-001.png
[lindarex-ubuntu-1604-installation-002]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-002.png
[lindarex-ubuntu-1604-installation-003]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-003.png
[lindarex-ubuntu-1604-installation-004]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-004.png
[lindarex-ubuntu-1604-installation-005]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-005.png
[lindarex-ubuntu-1604-installation-006]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-006.png
[lindarex-ubuntu-1604-installation-007]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-007.png
[lindarex-ubuntu-1604-installation-008]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-008.png
[lindarex-ubuntu-1604-installation-009]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-009.png
[lindarex-ubuntu-1604-installation-010]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-010.png
[lindarex-ubuntu-1604-installation-011]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-011.png
[lindarex-ubuntu-1604-installation-012]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-012.png
[lindarex-ubuntu-1604-installation-013]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-013.png
[lindarex-ubuntu-1604-installation-014]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-014.png
[lindarex-ubuntu-1604-installation-015]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-015.png
[lindarex-ubuntu-1604-installation-016]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-016.png
[lindarex-ubuntu-1604-installation-017]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-017.png
[lindarex-ubuntu-1604-installation-018]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-018.png
[lindarex-ubuntu-1604-installation-019]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-019.png
[lindarex-ubuntu-1604-installation-020]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-020.png
[lindarex-ubuntu-1604-installation-021]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-021.png
[lindarex-ubuntu-1604-installation-022]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-022.png
[lindarex-ubuntu-1604-installation-023]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-023.png
[lindarex-ubuntu-1604-installation-024]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-024.png
[lindarex-ubuntu-1604-installation-025]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-025.png
[lindarex-ubuntu-1604-installation-026]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-026.png
[lindarex-ubuntu-1604-installation-027]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-027.png
[lindarex-ubuntu-1604-installation-028]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-028.png
[lindarex-ubuntu-1604-installation-029]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-029.png
[lindarex-ubuntu-1604-installation-030]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-030.png
[lindarex-ubuntu-1604-installation-031]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-031.png
[lindarex-ubuntu-1604-installation-032]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-032.png
[lindarex-ubuntu-1604-installation-033]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-033.png
[lindarex-ubuntu-1604-installation-034]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-034.png
[lindarex-ubuntu-1604-installation-035]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-035.png
[lindarex-ubuntu-1604-installation-036]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-036.png
[lindarex-ubuntu-1604-installation-037]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-037.png
[lindarex-ubuntu-1604-installation-038]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-038.png
[lindarex-ubuntu-1604-installation-039]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-039.png
[lindarex-ubuntu-1604-installation-040]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-040.png
[lindarex-ubuntu-1604-installation-041]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-041.png
[lindarex-ubuntu-1604-installation-042]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-042.png
[lindarex-ubuntu-1604-installation-043]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-043.png
[lindarex-ubuntu-1604-installation-044]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-044.png
[lindarex-ubuntu-1604-installation-045]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-045.png
[lindarex-ubuntu-1604-installation-046]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-046.png
[lindarex-ubuntu-1604-installation-047]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-047.png
[lindarex-ubuntu-1604-installation-048]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-048.png
[lindarex-ubuntu-1604-installation-049]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-049.png
[lindarex-ubuntu-1604-installation-050]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-050.png
[lindarex-ubuntu-1604-installation-051]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-051.png
[lindarex-ubuntu-1604-installation-052]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-052.png
[lindarex-ubuntu-1604-installation-053]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-053.png
[lindarex-ubuntu-1604-installation-054]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-054.png
[lindarex-ubuntu-1604-installation-055]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-055.png
[lindarex-ubuntu-1604-installation-056]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-056.png
[lindarex-ubuntu-1604-installation-057]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-057.png
[lindarex-ubuntu-1604-installation-058]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-058.png
[lindarex-ubuntu-1604-installation-059]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-059.png
[lindarex-ubuntu-1604-installation-060]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-060.png
[lindarex-ubuntu-1604-installation-061]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-061.png
[lindarex-ubuntu-1604-installation-062]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-062.png
[lindarex-ubuntu-1604-installation-063]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-063.png
[lindarex-ubuntu-1604-installation-064]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-064.png
[lindarex-ubuntu-1604-installation-065]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-065.png
[lindarex-ubuntu-1604-installation-066]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-066.png
[lindarex-ubuntu-1604-installation-067]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-067.png
[lindarex-ubuntu-1604-installation-068]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-068.png
[lindarex-ubuntu-1604-installation-069]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-069.png
[lindarex-ubuntu-1604-installation-070]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-070.png
[lindarex-ubuntu-1604-installation-071]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-071.png
[lindarex-ubuntu-1604-installation-072]:/assets/images/2020-02-10-ubuntu-1604-installation/lindarex-ubuntu-1604-installation-072.png
