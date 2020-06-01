---
title: "우분투(Ubuntu) 환경에 iptables 설정하기"
categories: 
  - ubuntu
tags: 
  - iptables
  - firewall
  - nat
  - "open source software"
  - "open source"
  - oss
  - ubuntu
---


iptables은 방화벽(firewall) 구성이나 NAT(network address translation)에 사용되는데, 리눅스(linux) 커널(kernel) firewall이 제공하는 테이블(table)과 체인(chain), 규칙(rule)을 시스템 관리자가 구성합니다. <br />
각각 다른 kernel 모듈(module)과 프로그램들은 다른 프로토콜(protocol)을 위해 사용되는데, iptables는 IPv4에, ip6tables는 IPv6에, arptables는 ARP에, ebtables는 이더넷(ethernet) 프레임에 적용됩니다. <br />
linux 시스템(system)에서 iptables는 /usr/sbin/iptables에 설치되며, iptables의 후속 버전은 nftables입니다. <br />
이 포스트에서는 ubuntu 환경에서 iptables를 설정하는 방법을 소개합니다.

> nftables이란 iptables에 비해 코드 중복이 적고 처리량이 더 많은 iptables의 후속 버전이며, linux kernel 3.13부터 사용 가능합니다. nftables에 대한 자세한 정보는 [https://en.wikipedia.org/wiki/Nftables](https://en.wikipedia.org/wiki/Nftables){: target="\_blank"}를 확인해 주시기 바랍니다.


## 선행조건(PREREQUISITE)
- ubuntu 환경이 필요합니다.

> Ubuntu 설치 방법은 [VMware workstation에 ubuntu server 16.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1604-installation/){: target="\_blank"} 또는 [VMware workstation에 ubuntu server 18.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1804-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.1 build-15018445)
- Ubuntu 18.04.4 LTS (Bionic Beaver) Server (64-bit)


## 요약(SUMMARY)
1. iptables 확인 및 초기화
2. iptables 조회
3. iptables 용어
4. iptables 사용 방법


## 내용(CONTENTS)
- iptables는 kernel 2.4 이전 버전에서 사용되던 ipchains를 대신하는 firewall 도구이며, NetFilter 프로젝트에서 개발했습니다.
- iptables는 kernel에서 netfilter 패킷(packet) 필터링(filtering) 기능을 사용자 공간에서 제어하는 수준으로 사용하고, protocol 상태 추적, packet 애플리케이션(application) 계층(layer) 검사, 속도 제한, filtering 정책(policy) 등의 기능을 제공합니다.
- ubuntu 18.04는 기본적으로 iptables와 함께 UFW(Uncomplicated Firewall, ufw)를 제공합니다. 
- iptables를 설정하기 전에 ufw를 비활성화하거나 제거하는 것을 추천합니다.
- iptables 설정에는 ROOT 권한을 요구하기 때문에, 이 포스트에서는 ROOT 사용자가 설정한다는 가정하에 설명합니다.

> ufw에 대한 자세한 설명은 [우분투(Ubuntu) 환경에 방화벽(UFW) 설정하기](https://lindarex.github.io/ubuntu/ubuntu-ufw-setting/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

### 1. iptables 확인 및 초기화
> ROOT 권한 필요

#### 1.1. iptables 확인
```console
# which iptables
/sbin/iptables
```

```console
# iptables -V
iptables v1.6.1
```

#### 1.2. iptables 초기화
- 모든 chain에 설정된 모든 rule을 제거합니다. 

```console
# iptables -F
```

- 기본 chain을 제외한 나머지 모든 chain에 설정된 모든 rule을 제거합니다. 

```console
# iptables -X
```

### 2. iptables 조회
> 아래 명령어의 결과는 iptables의 기본 설정 상태입니다.

- 기본 조회입니다.

```console
# iptables -L
chain INPUT (policy ACCEPT)
target     prot opt source               destination

chain FORWARD (policy ACCEPT)
target     prot opt source               destination

chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

- 아래는 조회에 사용되는 '-L' command의 옵션 목록입니다.
    + '-v' 또는 '--verbose' :: 각 chain의 packet과 byte counters 정보, 각 rule에 일치하는 packet과 byte counters 정보 및 특정 rule에 적용되는 인터페이스(interface) 등을 조회합니다.
    + '-n' 또는 '--numeric' :: 기본 호스트(host) 이름, 네트워크(network) 이름 또는 서비스(service) 형식이 아닌 IP 주소(address)와 포트(port) 번호(number)로 표시합니다.
    + '-x' 또는 '--exact' :: packet과 byte counters 정보를 정확한 값으로 확장하여 표시합니다.
    + '--line-numbers' :: chain의 각 rule의 시작 부분에 숫자를 추가합니다.

- 상세 조회입니다.

```console
# iptables -L -v
Chain INPUT (policy ACCEPT 10 packets, 676 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 5 packets, 812 bytes)
 pkts bytes target     prot opt in     out     source               destination
```

- IP address와 port number로 표시하여 조회합니다.

```console
# iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

- 정확한 값으로 확장하여 조회합니다.

```console
# iptables -L -x
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

- rule의 시작 부분에 숫자를 추가하여 조회합니다.

```console
# iptables -L --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination
```

- 전체 rule을 출력합니다.

```console
# iptables -S
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
```

- 전체 rule을 상세하게 출력합니다.

```console
# iptables -S -v
-P INPUT ACCEPT -c 117 8240
-P FORWARD ACCEPT -c 0 0
-P OUTPUT ACCEPT -c 103 13208
```

> '-L'과 '-S' 옵션의 차이점은 설명으로는 조회와 출력이 차이지만, 산출 형식(output format)이 다릅니다. '-L' 옵션은 reference, 즉 참조용이지만, '-S' 옵션은 reusable output, 즉 재사용을 위한 출력입니다. '-S' 옵션은 iptables-save 방식으로 생성되어 iptables-apply 및 iptables-restore에 다시 사용할 수 있습니다.

### 3. iptables 용어
#### 3.1. target
- target은 packet이 rule과 일치할 때 실행될 명령입니다.
- 각각의 target에 대한 설명은 아래와 같습니다.
    + ACCEPT :: packet을 허용합니다.
    + DROP :: packet을 차단하고, 사용자에게 오류 메시지를 보내지 않습니다.
    + REJECT :: packet을 차단하고, 사용자에게 오류 메시지를 보냅니다.
    + LOG :: packet을 syslog에 기록합니다.
    + RETURN :: chain 통과를 중지하고, 이전(호출) chain의 다음 rule에 따라 다시 시작합니다.
    + QUEUE :: application에 packet을 대기(queue)시키는 데 사용합니다.
    + SNAT :: 소스 네트워크 주소 변환(source network address translation)에 사용되며, 이는 packet의 IP 헤더에 source IP address를 다시 쓰는 데 사용됩니다.
    + DNAT :: 대상 네트워크 주소 변환(destination network address translation)에 사용되며, 이는 packet의 destination IP address를 다시 쓰는 데 사용됩니다.

#### 3.2. chain
- chain은 순차적으로 검사하는 rule 세트이며, packet이 rule 중 하나와 일치하면 관련 작업이 실행되고, chain의 나머지 rule은 확인하지 않습니다.
- filter table에는 미리 정의된 INPUT, FORWARD, OUTPUT 등 3개의 chain이 존재하며, 그 외에 PREROUTING, POSTROUTING 등 2개의 chain이 존재합니다.
- INPUT, FORWARD, OUTPUT chain은 기본 chain으로 삭제가 안 되고 영구적으로 사용되며, '-N' 옵션으로 사용자가 chain을 구성하여 사용할 수 있습니다.
- chain은 network 통신(IP packet)에 미리 설정한 rule을 적용하여 target을 결정합니다.
- 각각의 chain에 대한 설명은 아래와 같습니다.
    + INPUT :: system으로 들어오는 packet의 policy rules입니다.
    + OUTPUT :: system에서 나가는 packet의 policy rules입니다.
    + FORWARD :: system에서 다른 system으로 보내는 packet의 policy rules입니다.
    + PREROUTING :: packet이 INPUT chain에 도달하기 전에 packet을 변경합니다.
    + POSTROUTING :: packet이 OUTPUT chain을 종료한 후에 packet을 변경합니다.

#### 3.3. table
- iptables는 filter, nat, mangle, raw, security 등 5개의 table이 존재합니다.
- '-t' 또는 '--table' 옵션으로 packet matching table을 지정합니다.

- filter table
    + 기본 table이며, '-t' 옵션이 없을 때 적용합니다.
    + filter table은 built-in chain INPUT, FORWARD, OUTPUT으로 구성됩니다.

- nat table
    + nat table은 새로운 연결을 생성하는 packet이 발견되면 참조됩니다.
    + nat table은 built-in chain PREROUTING, INPUT, OUTPUT, POSTROUTING으로 구성됩니다.
    + IPv6 NAT는 kernel 3.7부터 지원합니다.

- mangle table
    + mangle table은 특수한 packet 변경에 사용됩니다.
    + mangle table은 kernel 2.4.17까지 built-in chain PREROUTING, OUTPUT으로 구성되었다가, kernel 2.4.18부터 built-in chain INPUT, FORWARD, POSTROUTING도 추가 구성되었습니다.

- raw table
    + raw table은 NOTRACK 대상과 함께 연결 추적의 제외 구성 시에 사용됩니다.
    + raw table은 built-in chain PREROUTING, OUTPUT으로 구성됩니다.

- security table
    + security table은 SELinux와 같은 linux 보안 module에 의해 구현되는 Mandatory Access Control(MAC) 네트워킹 rule에 사용됩니다.
    + security table은 filter table 다음에 호출되기 때문에, security table MAC rule이므로 filter table의 Discretionary Access Control(DAC)보다 나중에 적용됩니다.
    + security table은 built-in chain INPUT, FORWARD, OUTPUT으로 구성됩니다.

#### 3.4. parameter
- iptables의 rule 스펙(specification)을 구성하며, add, delete, insert, replace 및 append 명령어에 사용됩니다.
- 각각의 parameter에 대한 설명은 아래와 같습니다.
    + '-4' 또는 '--ipv4' :: iptables 및 iptables-restore에 영향이 없고, 단일 rule 파일(file)에서 iptables-restore 및 ip6tables-restore와 함께 사용하여 IPv4 및 IPv6 rule을 허용합니다.
    + '-6' 또는 '--ipv6' :: ip6tables 및 ip6tables-restore에 영향이 없고, 단일 rule 파일(file)에서 iptables-restore 및 ip6tables-restore와 함께 사용하여 IPv4 및 IPv6 rule을 허용합니다.
    + '-p' 또는 '--protocol' :: rule 또는 packet의 protocol을 확인합니다.
    + '-s' 또는 '--source' :: 출발지 network 이름, host 이름, network IP address(/mask) 또는 일반 IP address를 확인합니다.
    + '-d' 또는 '--destination' :: 목적지 network 이름, host 이름, network IP address(/mask) 또는 일반 IP address를 확인히며, 별칭(alias)은 '-dst'입니다.
    + '-m' 또는 '--match' :: 사용할 확장 패킷 모듈(extended packet modules)을 지정합니다.
    + '-j' 또는 '--jump' :: rule의 target을 지정합니다.
    + '-g' 또는 '--goto' :: 사용자 지정 chain으로 계속 처리되도록 명시합니다.
    + '-i' 또는 '--in-interface' :: packet을 수신할 interface를 명시합니다.
    + '-o' 또는 '--out-interface' :: packet이 전송될 interface를 명시합니다.
    + '-f' 또는 '--fragment' :: 조각난 packet의 두 번째와 그 이후 IPv4 조각만을 참조하는 rule을 명시합니다.
    + '-c' 또는 '--set-counters' :: INSERT, APPEND, REPLACE 조작 중에 rule의 packet 및 바이트 카운터(byte counters)를 초기화할 수 있습니다.

#### 3.5. command
- iptables이 수행할 작업을 지정합니다.
- 각각의 command에 대한 설명은 아래와 같습니다.
    + '-A' 또는 '--append' :: 선택한 chain 끝에 하나 이상의 rule을 추가합니다.
    + '-C' 또는 '--check' :: 선택한 chain의 rule과 일치하는 rule이 있는지 확인합니다.
    + '-D' 또는 '--delete' :: 선택한 chain에서 하나 이상의 rule을 삭제합니다.
    + '-I' 또는 '--insert' :: rule number로 선택한 chain에 하나 이상의 rule을 삽입합니다.
    + '-R' 또는 '--replace' :: rule number로 선택한 chain에서 rule을 교체합니다.
    + '-L' 또는 '--list' :: 선택한 chain의 모든 rule을 조회하며, chain을 선택하지 않으면 모든 chain의 rule을 조회합니다.
    + '-S' 또는 '--list-rules' :: 선택한 chain의 모든 rule을 출력하며, chain을 선택하지 않으면 모든 chain의 rule을 출력합니다.
    + '-F' 또는 '--flush' :: 선택한 chain의 rule을 모두 삭제합니다.
    + '-N' 또는 '--new' :: 사용자 정의 chain을 생성합니다.
    + '-E' 또는 '--rename-chain' :: 사용자 정의 chain의 이름을 변경합니다.
    + '-X' 또는 '--delete-chain' :: 사용자 정의 chain을 삭제합니다. 단, chain에 대한 참조(references)가 없고, rule이 없이 비어있어야 합니다.
    + '-P' 또는 '--policy' :: 사용자 정의 chain을 제외한 built-in chain의 policy를 지정한 target으로 설정합니다.
    + '-Z' 또는 '--zero' :: 모든 chain 또는 선택한 chain, chain의 지정된 rule의 packet과 byte counters를 0으로 설정합니다.

### 4. iptables 사용 방법
#### 4.1. rule 추가
- '-A' 또는 '--append' command를 사용합니다.

```console
# iptables [table] -A chain rule-specification
```

- 상세히 설명하면 아래와 같습니다.

```console
# iptables [-t TABLE-NAME] -A chain -m MATCH-NAME [match-options] -j TARGET-NAME [target-options]
```

- 예제 :: INPUT chain의 localhost 접속 허용 rule을 추가합니다.

```console
# iptables -A INPUT -i lo -j ACCEPT
```

#### 4.2. rule 확인
- '-C' 또는 '--check' command를 사용합니다.

```console
# iptables [table] -C chain rule-specification
```

- 상세히 설명하면 아래와 같습니다.

```console
# iptables [-t TABLE-NAME] -C chain -m MATCH-NAME [match-options] -j TARGET-NAME [target-options]
```

- 예제 :: INPUT chain의 tcp 8080 포트(port) 접속 허용 rule을 확인합니다.

```console
# iptables -C INPUT -p tcp --dport 8080 -j ACCEPT
```

#### 4.3. rule 삭제
- '-D' 또는 '--delete' command를 사용합니다.

```console
# iptables [table] -D chain rule-specification
```

- 상세히 설명하면 아래와 같습니다.

```console
# iptables [-t TABLE-NAME] -D chain -m MATCH-NAME [match-options] -j TARGET-NAME [target-options]
```

- 예제 :: INPUT chain의 tcp 22 port 접속 차단 rule을 삭제합니다.

```console
# iptables -D INPUT -p tcp -m tcp --dport 22 -j REJECT
```

#### 4.4. rule number로 rule 삭제
- '-D' 또는 '--delete' command를 사용합니다.

```console
# iptables [table] -D chain rule-number
```

- 상세히 설명하면 아래와 같습니다.

```console
# iptables [-t TABLE-NAME] -D chain rule-number
```

- 예제 :: FORWARD chain의 rule number 1번 rule을 삭제합니다.

```console
# iptables -D FORWARD 1
```

#### 4.5. rule 삽입
- '-I' 또는 '--insert' command를 사용합니다.

```console
# iptables [table] -I [rule-number] rule-specification
```

- 상세히 설명하면 아래와 같습니다.

```console
# iptables [-t TABLE-NAME] -I chain [rule-number] -m MATCH-NAME [match-options] -j TARGET-NAME [target-options]
```

- 예제 :: INPUT chain의 rule number 1번에 udp 53 port 접속 차단 rule을 삽입합니다.

```console
# iptables -I INPUT 1 -p udp --dport 53 -j DROP
```

#### 4.6. rule 교체
- '-R' 또는 '--replace' command를 사용합니다.

```console
# iptables [table] -R rule-number rule-specification
```

- 상세히 설명하면 아래와 같습니다.

```console
# iptables [-t TABLE-NAME] -R chain rule-number -m MATCH-NAME [match-options] -j TARGET-NAME [target-options]
```

- 예제 :: INPUT chain의 rule number 1번에 출발지 network '10.0.1.0/24' 대역의 tcp 8080 port 접속 허용 rule을 교체합니다.

```console
# iptables -R INPUT 1 -p tcp -s 10.0.1.0/24 --dport 8080 -j ACCEPT
```

#### 4.7. rule 조회
- '-L' 또는 '--list' command를 사용합니다.

```console
# iptables [table] -L [chain [rule-number]] [options]
```

- 상세히 설명하면 아래와 같습니다.

```console
# iptables [-t TABLE-NAME] -L [chain [rule-number]] [options]
```

- 예제 :: 모든 rule을 service 이름으로 조회합니다.

```console
# iptables -L
```

- 예제 :: 모든 rule을 port number로 조회합니다.

```console
# iptables -L -n
```

#### 4.8. rule 모두 삭제
- '-F' 또는 '--flush' command를 사용합니다.

```console
# iptables [table] -F [chain [rule-number]] [options]
```

- 상세히 설명하면 아래와 같습니다.

```console
# iptables [-t TABLE-NAME] -F [chain [rule-number]] [options]
```

- 예제 :: 모든 chain의 rule을 삭제합니다.

```console
# iptables -F
```

- 예제 :: FORWARD chain의 모든 rule을 삭제합니다.

```console
# iptables -F FORWARD
```

#### 4.9. rule 출력
- '-S' 또는 '--list-rules' command를 사용합니다.

```console
# iptables [table] -S [chain [rule-number]]
```

- 상세히 설명하면 아래와 같습니다.

```console
# iptables [-t TABLE-NAME] -S [chain [rule-number]]
```

- 예제 :: OUTPUT chain의 rule number 1번 rule을 출력합니다.

```console
# iptables -S OUTPUT 1
```

#### 4.10. 사용자 정의 chain 생성
- '-N' 또는 '--new' command를 사용합니다.

```console
# iptables [table] -N chain
```

- 상세히 설명하면 아래와 같습니다.

```console
# iptables [-t TABLE-NAME] -N chain
```

- 예제 :: 'lindarex-chain-income'이라는 이름의 사용자 정의 chain을 생성합니다.

```console
# iptables -N lindarex-chain-income
```

#### 4.11. 사용자 정의 chain 이름 변경
- '-E' 또는 '--rename-chain' command를 사용합니다.

```console
# iptables [table] -E old-chain-name new-chain-name
```

- 상세히 설명하면 아래와 같습니다.

```console
# iptables [-t TABLE-NAME] -E old-chain-name new-chain-name
```

- 예제 :: 생성한 'lindarex-chain-income'이라는 이름의 사용자 정의 chain의 이름을 'lindarex-chain-income-new'로 변경합니다.

```console
# iptables -E lindarex-chain-income lindarex-chain-income-new
```

#### 4.12. 사용자 정의 chain 삭제
- '-X' 또는 '--delete-chain' command를 사용합니다.

```console
# iptables [table] -X [chain]
```

- 상세히 설명하면 아래와 같습니다.

```console
# iptables [-t TABLE-NAME] -X [chain]
```

- 예제 :: 생성한 'lindarex-chain-income-new'라는 이름의 사용자 정의 chain을 삭제합니다.

```console
# iptables -X lindarex-chain-income-new
```

#### 4.13. built-in chain의 policy target 설정
- '-P' 또는 '--policy' command를 사용합니다.

```console
# iptables [table] -P chain target
```

- 상세히 설명하면 아래와 같습니다.

```console
# iptables [-t TABLE-NAME] -P chain -j TARGET-NAME [target-options]
```

- 예제 :: OUTPUT chain의 policy를 DROP으로 설정합니다.

```console
# iptables -P FORWARD DROP
```

#### 4.14. packet과 byte counters를 0으로 설정
- '-Z' 또는 '--zero' command를 사용합니다.

```console
# iptables [table] -Z [chain [rule-number]] [options]
```

- 상세히 설명하면 아래와 같습니다.

```console
# iptables [-t TABLE-NAME] -Z [chain [rule-number]] [options]
```

- 예제 :: FORWARD chain의 모든 rule의 counters를 0으로 설정합니다.

```console
# iptables -Z FORWARD
```


## 마무리(CONCLUSION)
ubuntu 환경에 iptables 설정을 완료했습니다. <br />
iptables 설정을 통해 서버(server) 보안을 많이 향상할 수 있지만, 잘못 설정할 경우에는 접속을 못 하는 경우가 발생할 수 있으니 신중하게 설정해야 합니다. <br />
다음 포스트에서는 [우분투(Ubuntu) 서버(Server) 초기 설정하기](https://lindarex.github.io/ubuntu/ubuntu-initial-setting/){: target="\_blank"}를 소개하겠습니다.


## 참고(REFERENCES)
- [https://ko.wikipedia.org/wiki/iptables](https://ko.wikipedia.org/wiki/iptables){: target="\_blank"}
- [https://www.linuxtopia.org/index.html](https://www.linuxtopia.org/index.html){: target="\_blank"}
- [https://fedoraproject.org/wiki/How_to_edit_iptables_rules](https://fedoraproject.org/wiki/How_to_edit_iptables_rules){: target="\_blank"}
- [https://linux.die.net/man/8/iptables](https://linux.die.net/man/8/iptables){: target="\_blank"}
