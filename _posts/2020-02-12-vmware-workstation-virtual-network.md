---
title: "VMware Workstation의 가상 네트워크(Virtual Network) 알아보기"
categories: 
  - vmware-workstation
tags: 
  - "vmware workstation"
  - "virtual network"
  - bridged
  - nat
  - "host-only"
---


VMware Workstation은 가상 머신(VM, virtual machine)을 생성하고 가상 네트워크(virtual network)를 구성할 수 있습니다.
<br /><br />
이 포스트에서는 VMware Workstation의 virtual network에 대해 소개합니다.


## 선행조건(PREREQUISITE)
- VMware Workstation이 설치되어 있어야 합니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.1 build-15018445)


## 요약(SUMMARY)
1. Bridged Networking
2. NAT Networking
3. Host-Only Networking

![lindarex_vmware-workstation-networking-configurations]


## 내용(CONTENTS)
### 1. Bridged Networking
- Bridged Networking은 호스트(host) 시스템(system)의 네트워크(network) 어댑터(adapter)를 사용하여 VM을 network에 연결합니다.
- Bridged Networking을 적용한 VM은 물리적 system으로 인식되고, 공유기는 개별적으로 IP를 할당합니다.
- 공유기를 통해 외부 통신을 합니다.
- 동일한 network에 있다는 전제하에, Bridged Networking을 적용한 VM은 host system을 비롯해 동일 network 내의 다른 system에도 엑세스가 가능합니다.
- VMware Workstation 설치 시 Bridged Network(VMnet0)가 설정됩니다.

![lindarex_vmware-workstation-networking-bridged]

> 출처 :: [https://docs.vmware.com/en/VMware-Workstation-Player-for-Windows/15.0/com.vmware.player.win.using.doc/GUID-BAFA66C3-81F0-4FCA-84C4-D9F7D258A60A.html](https://docs.vmware.com/en/VMware-Workstation-Player-for-Windows/15.0/com.vmware.player.win.using.doc/GUID-BAFA66C3-81F0-4FCA-84C4-D9F7D258A60A.html){: target="\_blank"}


### 2. NAT Networking
- NAT(Network Address Translation) Networking을 적용한 VM은 외부 network 통신을 위한 IP를 할당받지 않습니다.
- host system에 별도의 개인 network가 설정되고, NAT Networking을 적용한 VM은 가상 DHCP 서버(server)로부터 내부 network 대역을 할당받습니다.
- host system을 통해 외부 통신을 합니다.
- NAT Network는 하나만 존재합니다.
- VMware Workstation 설치 시 NAT Network(VMnet8)가 설정됩니다.

![lindarex_vmware-workstation-networking-nat]

> 출처 :: [https://docs.vmware.com/en/VMware-Workstation-Player-for-Windows/15.0/com.vmware.player.win.using.doc/GUID-89311E3D-CCA9-4ECC-AF5C-C52BE6A89A95.html](https://docs.vmware.com/en/VMware-Workstation-Player-for-Windows/15.0/com.vmware.player.win.using.doc/GUID-89311E3D-CCA9-4ECC-AF5C-C52BE6A89A95.html){: target="\_blank"}

### 3. Host-Only Networking
- Host-Only Networking을 적용한 VM은 기본적으로 외부 통신, host system과 통신이 불가능합니다.
- Host-Only Networking을 적용한 VM은 VMware Workstation에서 생성한 VM만 통신이 가능합니다.
- VMware Workstation 설치 시 Host-Only Network(VMnet1)가 설정됩니다.

![lindarex_vmware-workstation-networking-host-only]

> 출처 :: [https://docs.vmware.com/en/VMware-Workstation-Player-for-Windows/15.0/com.vmware.player.win.using.doc/GUID-93BDF7F1-D2E4-42CE-80EA-4E305337D2FC.html](https://docs.vmware.com/en/VMware-Workstation-Player-for-Windows/15.0/com.vmware.player.win.using.doc/GUID-93BDF7F1-D2E4-42CE-80EA-4E305337D2FC.html){: target="\_blank"}


## 마무리(CONCLUSION)
VMware Workstation의 virtual network에 대해 알아봤습니다.
<br /><br />
위에 기재한 networking 개념은 VMware Workstation뿐만 아니라 모든 운영체제(OS, operating system), 특히 IaaS network 설정에 필요합니다.
<br /><br />
대부분 로컬 테스트를 할 때, VMware Workstation의 virtual network 설정을 수정하는 경우는 드물지만, 클라우드(cloud) 기반 프로젝트에서는 기본적인 network 개념을 충분히 이해하고 테스트할 필요가 있습니다.
<br /><br />
더 자세한 내용은 아래 참고 페이지를 확인해 주시기 바랍니다.


## 참고(REFERENCES)
- [https://docs.vmware.com/en/VMware-Workstation-Player-for-Windows/15.0/com.vmware.player.win.using.doc/GUID-D9B0A52D-38A2-45D7-A9EB-987ACE77F93C.html](https://docs.vmware.com/en/VMware-Workstation-Player-for-Windows/15.0/com.vmware.player.win.using.doc/GUID-D9B0A52D-38A2-45D7-A9EB-987ACE77F93C.html){: target="\_blank"}


[lindarex_vmware-workstation-networking-bridged]:/assets/images/2020-02-12-vmware-workstation-virtual-network/lindarex_vmware-workstation-networking-bridged.png
[lindarex_vmware-workstation-networking-configurations]:/assets/images/2020-02-12-vmware-workstation-virtual-network/lindarex_vmware-workstation-networking-configurations.png
[lindarex_vmware-workstation-networking-host-only]:/assets/images/2020-02-12-vmware-workstation-virtual-network/lindarex_vmware-workstation-networking-host-only.png
[lindarex_vmware-workstation-networking-nat]:/assets/images/2020-02-12-vmware-workstation-virtual-network/lindarex_vmware-workstation-networking-nat.png
