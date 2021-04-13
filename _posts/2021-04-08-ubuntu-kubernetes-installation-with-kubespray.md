---
title: "우분투(Ubuntu) 환경에 Kubespray로 Kubernetes 클러스터(cluster) 설치하기"
categories: 
  - kubernetes
tags: 
  - kubernetes
  - cluster
  - "kubernetes cluster"
  - container
  - orchestration
  - "container orchestration"
  - kubespray
  - docker
  - "open source software"
  - "open source"
  - oss
  - ubuntu
---

kubernetes cluster를 설치하는 방법은 다양합니다.
<br /><br />
이 포스트에서는 우분투(ubuntu) 환경에서 kubespray를 이용하여 kubernetes cluster를 설치하는 방법을 소개합니다.

> Kubeadm으로 kubernetes cluster를 설치하는 방법은 [우분투(Ubuntu) 환경에 Kubeadm으로 kubernetes 클러스터(cluster) 설치하기](https://lindarex.github.io/kubernetes/ubuntu-kubernetes-installation-with-kubeadm/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

> kubernetes의 아키텍처(architectures) 및 개념(concepts), 리소스(resources) 등의 부가 설명은 이 포스트에서 다루지 않으며, 다음 포스트에서 kubernetes에 대한 자세한 설명을 소개하겠습니다.


## 선행조건(PREREQUISITE)
- 최소 2개(마스터, 워커 각각 1개)의 ubuntu 환경이 필요합니다.
- 방화벽 설정이 필요합니다.

> 마스터(master)는 master 노드(node) 또는 컨트롤 플레인(control plane) 또는 API 서버(server)로 불리며, 워커(worker)는 worker node 또는 node로 불립니다.

> ubuntu 설치 방법은 [우분투(Ubuntu) 서버(Server) 18.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1804-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

> kubernetes 최소 설치 권장 사항은 [https://kubernetes.io/docs/setup/production-environment/tools/kubespray/install-kubespray/#before-you-begin](https://kubernetes.io/docs/setup/production-environment/tools/kubespray/install-kubespray/#before-you-begin){: target="\_blank"} 페이지를 참고하시기 바랍니다.

> 방화벽 설정 방법은 [우분투(Ubuntu) 환경에 방화벽(Firewalld) 설치 및 설정하기](https://lindarex.github.io/ubuntu/ubuntu-firewalld-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

> kubernetes 실행에 필요한 필수 포트는 [https://kubernetes.io/docs/setup/production-environment/tools/kubespray/install-kubespray/#check-required-ports](https://kubernetes.io/docs/setup/production-environment/tools/kubespray/install-kubespray/#check-required-ports){: target="\_blank"} 페이지를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- AWS EC2 instance
    + Ubuntu Server 18.04 LTS (HVM), EBS General Purpose (SSD) Volume Type (64비트 Arm)
    + m5.large (ECU, 2 vCPUs, 3.1 GHz, 8 GiB 메모리, EBS 전용)
- Ubuntu 18.04.5 LTS (GNU/Linux 5.4.0-1038-aws x86_64)
- kubespray 2.15.0
- kubernetes 1.19.7
- docker 19.03.14


## 요약(SUMMARY)
1. SSH Key 생성 및 배포
2. kubespray 내려받기
3. apt 명령어로 python 설치
4. kubespray python package 설치
5. kubespray 설정 구성
6. kubespray 설치
7. kubernetes cluster 설치 확인
8. (선택사항) kubespray 삭제


## 내용(CONTENTS)
### 1. SSH Key 생성 및 배포

- SSH key 기반의 인증을 사용할 수 있도록 master node에서 RSA 공개키를 생성한 후, master node를 포함한 모든 node에 복사합니다.
- kubespray는 Ansible(ansible)를 이용하여 kubernetes cluster를 설치합니다. ansible은 일반적으로 SSH 프로토콜을 통해 원격 시스템과 통신하기 때문에 모든 node에 SSH 구성이 필요합니다.

#### 1.1. SSH Key 생성

> master node에서 실행합니다.

```console
$ ssh-keygen -t rsa
```

```console
ubuntu@ip-10-0-0-12:~$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/id_rsa.
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:F15JTayumjfr3urkY0CB74Ap04TDQYgGby6hJRk3VAY ubuntu@ip-10-0-0-12
The key's randomart image is:
+---[RSA 2048]----+
|==E=o  .    .+.  |
|o*+o. . .  . .o  |
|= ++ o . .. o.   |
|.*o + . o. o.    |
|o .o   +S o.     |
| .      o.  .    |
|         ...     |
|         +*.     |
|        oBB=.    |
+----[SHA256]-----+
```

#### 1.2. SSH Key를 모든 node에 복사

> 아래 명령어를 master node에서 실행하여 RSA 공개키를 복사합니다.

```console
$ cat ~/.ssh/id_rsa.pub
```

```console
ubuntu@ip-10-0-0-12:~$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDfCi8yYpfM203r9ZmkftJfaNX+sbF+EmQ4QW8ozOS2GCQhIg/mmQjSRhk5Pi+Ey2/ptzVi82XhKUpUkN334NVf8kdQhmQ4po6Yi4e9Zm/HDIiHmjA0PR5cE3o7ab9QhjsgV1RUGJbrvZdZPCxjSUzlK3suk/49/3PKlRrdNuCogznjPpeIF16oqlK8W1gxYi/lcdhXJ09Fa3cucUU5GnkAXAHsisdC8VyE3f78TlYA6rjb4+eY+mryK0WA+UH1pDYLyVCB2NH/WNiawln//pRSWIYgzSZDvhlR/eJKDXzaz6xGopdzK0qd1toJ3tJ5uDlO1kXVfTTNkgQER/IPNzxX ubuntu@ip-10-0-0-12
```

> 아래 명령어를 master node와 worker node에서 실행합니다.

```console
$ vi ~/.ssh/authorized_keys
```

```console
ubuntu@ip-10-0-0-12:~$ vi ~/.ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCIU1EZY5t3yMfd5JwQJVPEYOctTT34TEGT8yp0gmaCE/zDXPpZOGUkDlIH3F3HQVSaE7gr/qdMuWf2xdtRzfhQTCFtEK1CjjjMsZriJ2nhDALp5JbujkSoWdX+CkD//egssZa98XPEDDP8fijOmmDjqh36axfiJP54kQ3cSmSFKPONnzsqiqvEBTbilEVwxAWc1tqL0NrZB72MDJ+WyHfzwifVkC8/h9B2h6WtE440CxpGFVfXMJQxxGYrCaJaSjruO7KzQBzJ54NynNH5XzMqCDA+UhztWK+7NlBNSpNWvMYQ5dw7kQ8L6kr4aqpKKyQUH5JoMa0J/+N6qM8ihn5f aws-key
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC2PS9vPKdB9TMlGHaV53bTi60+pyqyguYjPz32xBMtGXmXmbOfkplgxo2xJvsOIUhiw5Jq1VKJP6kBIMv3PxtCZ8LX1eDLgun+3E6vxu8BpIOo9JEW3Rc6tpezBebqfVzqO/PgYY2dxH1MD6tUqWRiJz544wnPoAc5LUjcvkUmmFndUlTMjxBFIFifA52e0tFwH3UxFGQAVocDDggRw23uMv9NHiyAxwpFP88FGksizYq9RDyc7rhjeMh7ruuGu5nOII02XGZhjdzLDUfcra911w1+47kAOpcsrrXnlHyWLUTSXkaGrkfESWbXnLh15wUSym2T5H1Jv9SKf2blenPZ ubuntu@ip-10-0-0-12
```

- 위 과정(1.2.)은 ssh-copy-id 명령어로 대체 가능합니다.

> ssh-copy-id 명령어에 대한 자세한 사항은 [http://manpages.ubuntu.com/manpages/bionic/en/man1/ssh-copy-id.1.html](http://manpages.ubuntu.com/manpages/bionic/en/man1/ssh-copy-id.1.html){: target="\_blank"} 페이지를 참고하시기 바랍니다.

```console
$ ssh-copy-id {USER}@{HOST}
```

```console
ubuntu@ip-10-0-0-12:~$ ssh-copy-id ubuntu@10.0.0.16
```

#### 1.3. ssh-keyscan 명령어로 모든 node의 호스트(host) 등록

> master node에서 실행합니다.

- master node를 포함한 모든 node의 host를 등록합니다.
- 아래 과정을 진행하지 않을 경우, kubespray 설치 과정에서 worker node의 인증 권한 설정에 따라 RSA key fingerprint 확인이 발생할 수 있습니다. 

> ssh-keyscan 명령어에 대한 자세한 사항은 [http://manpages.ubuntu.com/manpages/bionic/man1/ssh-keyscan.1.html](http://manpages.ubuntu.com/manpages/bionic/man1/ssh-keyscan.1.html){: target="\_blank"} 페이지를 참고하시기 바랍니다.


```console
$ ssh-keyscan -t rsa {HOST} >> ~/.ssh/known_hosts
```

```console
ubuntu@ip-10-0-0-12:~$ ssh-keyscan -t rsa 10.0.0.12 >> ~/.ssh/known_hosts
# 10.0.0.12:22 SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.3
```

```console
ubuntu@ip-10-0-0-12:~$ ssh-keyscan -t rsa 10.0.0.16 >> ~/.ssh/known_hosts
# 10.0.0.16:22 SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.3
```

- 등록할 host가 많을 경우, host 목록 파일을 생성하여 아래 명령어로 한 번에 등록할 수 있습니다.

```console
$ vi {HOST-LIST-FILE}
---
{HOST-1}, {HOST-2}
```

```console
ubuntu@ip-10-0-0-12:~$ vi lindarex-host-file
---
10.0.0.12, 10.0.0.16
```

```console
$ ssh-keyscan -t rsa -f {HOST-LIST-FILE} >> ~/.ssh/known_hosts
```

```console
ubuntu@ip-10-0-0-12:~$ ssh-keyscan -t rsa -f lindarex-host-file >> ~/.ssh/known_hosts
```

#### 1.4. ssh 명령어로 worker node 접속 확인

> master node에서 실행합니다.

```console
$ ssh {USER}@{HOST}
```

```console
ubuntu@ip-10-0-0-12:~$ ssh ubuntu@10.0.0.16
```


### 2. kubespray 내려받기

> master node에서 실행합니다.

#### 2.1. (선택사항) workspace 디렉터리 생성
```console
$ export LINDAREX_WORKSPACE=${HOME}/workspace
$ mkdir -p ${LINDAREX_WORKSPACE}
```

#### 2.2. git clone 명령어로 kubespray 내려받기
```console
$ git clone https://github.com/kubernetes-sigs/kubespray.git -b v2.15.0 ${LINDAREX_WORKSPACE}/kubespray
```


### 3. apt 명령어로 python 설치

> master node에서 실행합니다.

#### 3.1. package index 업데이트 
```console
$ sudo apt update
```

#### 3.2. python 설치
```console
$ sudo apt install -y python3-pip
```


### 4. kubespray python package 설치

> master node에서 실행합니다.

```console
$ sudo pip3 install -r ${LINDAREX_WORKSPACE}/kubespray/requirements.txt
```


### 5. kubespray 설정 구성
#### 5.1. inventory 구성

> master node에서 실행합니다.

##### 5.1.1. inventory 파일 생성

- sample 디렉터리(directory)를 복사하여 inventory directory를 생성합니다.

```console
$ cp -r ${LINDAREX_WORKSPACE}/kubespray/inventory/sample ${LINDAREX_WORKSPACE}/kubespray/inventory/lindarex-cluster
```

##### 5.1.2. inventory 파일 수정
```console
$ vi {WORKSPACE}/kubespray/inventory/{CLUSTER_NAME}/inventory.ini
```

```console
ubuntu@ip-10-0-0-12:~$ vi ${LINDAREX_WORKSPACE}/kubespray/inventory/lindarex-cluster/inventory.ini
---
# ## Configure 'ip' variable to bind kubernetes services on a
# ## different ip than the default iface
# ## We should set etcd_member_name for etcd cluster. The node that is not a etcd member do not need to set the value, or can set the empty string value.
[all]
ip-10-0-0-12 ansible_host=10.0.0.12 ip=10.0.0.12 etcd_member_name=etcd1
ip-10-0-0-16 ansible_host=10.0.0.16 ip=10.0.0.16
# node3 ansible_host=95.54.0.14  # ip=10.3.0.3 etcd_member_name=etcd3
# node4 ansible_host=95.54.0.15  # ip=10.3.0.4 etcd_member_name=etcd4
# node5 ansible_host=95.54.0.16  # ip=10.3.0.5 etcd_member_name=etcd5
# node6 ansible_host=95.54.0.17  # ip=10.3.0.6 etcd_member_name=etcd6

# ## configure a bastion host if your nodes are not directly reachable
# bastion ansible_host=x.x.x.x ansible_user=some_user

[kube-master]
ip-10-0-0-12
# node2
# node3

[etcd]
ip-10-0-0-12
# node2
# node3

[kube-node]
ip-10-0-0-16
# node3
# node4
# node5
# node6

[calico-rr]

[k8s-cluster:children]
kube-master
kube-node
calico-rr
```

#### 5.2. (선택사항) 호스트명(hostname) 변경(override) 설정

- kubespray의 기본 설정은 모든 node의 hostname을 node1, node2... 형식으로 변경합니다.
- 이 포스트에서는 hostname을 변경하지 않게 설정을 수정하여 설치합니다.

```console
$ vi {WORKSPACE}/kubespray/roles/bootstrap-os/defaults/main.yml
---
override_system_hostname: false
```

```console
ubuntu@ip-10-0-0-12:~$ cat ${LINDAREX_WORKSPACE}/kubespray/roles/bootstrap-os/defaults/main.yml
---
## CentOS/RHEL specific variables
# Use the fastestmirror yum plugin
centos_fastestmirror_enabled: false

## Flatcar Container Linux specific variables
# Disable locksmithd or leave it in its current state
coreos_locksmithd_disable: false

## Oracle Linux specific variables
# Install public repo on Oracle Linux
use_oracle_public_repo: true

fedora_coreos_packages:
  - python
  - python3-libselinux
  - ethtool                 # required in kubeadm preflight phase for verifying the environment
  - ipset                   # required in kubeadm preflight phase for verifying the environment
  - conntrack-tools         # required by kube-proxy

## General
# Set the hostname to inventory_hostname
override_system_hostname: false

is_fedora_coreos: false

skip_http_proxy_on_os_packages: false
```


### 6. kubespray 설치

> master node에서 실행합니다.

#### 6.1. inventory builder로 inventory 생성
```console
$ declare -a IPS=({MASTER_NODE_IP} {WORKER_NODE_IP1} {WORKER_NODE_IP2} {WORKER_NODE_IP3}...)
$ CONFIG_FILE={WORKSPACE}/kubespray/inventory/{CLUSTER_NAME}/hosts.yaml python3 {WORKSPACE}/kubespray/contrib/inventory_builder/inventory.py ${IPS[@]}
```

```console
ubuntu@ip-10-0-0-12:~$ CONFIG_FILE=${LINDAREX_WORKSPACE}/kubespray/inventory/lindarex-cluster/hosts.yaml python3 ${LINDAREX_WORKSPACE}/kubespray/contrib/inventory_builder/inventory.py ${IPS[@]}
DEBUG: Adding group all
DEBUG: Adding group kube-master
DEBUG: Adding group kube-node
DEBUG: Adding group etcd
DEBUG: Adding group k8s-cluster
DEBUG: Adding group calico-rr
DEBUG: adding host node1 to group all
DEBUG: adding host node2 to group all
DEBUG: adding host node1 to group etcd
DEBUG: adding host node1 to group kube-master
DEBUG: adding host node2 to group kube-master
DEBUG: adding host node1 to group kube-node
DEBUG: adding host node2 to group kube-node
```

#### 6.2. (선택사항) master node 개수 설정

- kubespray의 기본 설정은 master node 개수가 2개입니다.
- inventory.ini 설정과 상관없이 worker node에도 master role을 부여합니다.
- 아래 hosts.yaml을 수정하여 master node 개수를 1개로 수정할 수 있습니다.
- 이 포스트에서는 master node 개수를 1개 설정하여 설치합니다.

```console
$ vi {WORKSPACE}/kubespray/inventory/{CLUSTER_NAME}/hosts.yaml
---
~
  children:
    kube-master:
      hosts:
        node1:
        node2: # 삭제
    kube-node:
      hosts:
        node1:
        node2:
~
```

```console
ubuntu@ip-10-0-0-12:~$ vi ${LINDAREX_WORKSPACE}/kubespray/inventory/lindarex-cluster/hosts.yaml
---
all:
  hosts:
    node1:
      ansible_host: 10.0.0.12
      ip: 10.0.0.12
      access_ip: 10.0.0.12
    node2:
      ansible_host: 10.0.0.16
      ip: 10.0.0.16
      access_ip: 10.0.0.16
  children:
    kube-master:
      hosts:
        node1:
    kube-node:
      hosts:
        node1:
        node2:
    etcd:
      hosts:
        node1:
    k8s-cluster:
      children:
        kube-master:
        kube-node:
    calico-rr:
      hosts: {}
```

#### 6.3. (선택사항) inventory 설정 확인
```console
$ cat {WORKSPACE}/kubespray/inventory/{CLUSTER_NAME}/group_vars/all/all.yml
```

```console
$ cat {WORKSPACE}/kubespray/inventory/{CLUSTER_NAME}/group_vars/k8s-cluster/k8s-cluster.yml
```

#### 6.4. ansible-playbook 명령어로 kubernetes cluster 설치

- ansible-playbook 명령어를 root 사용자로 실행하기 위해 '--become' 옵션과 '--become-user=root' 옵션은 필수입니다.

> ansible-playbook 명령어 사용 방법은 [https://docs.ansible.com/ansible/latest/user_guide/playbooks.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html){: target="\_blank"} 페이지를 참고하시기 바랍니다.

> ansible playbook에 대한 자세한 설명은 [https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html](https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html){: target="\_blank"} 페이지를 참고하시기 바랍니다.

```console
$ ansible-playbook -i {WORKSPACE}/kubespray/inventory/{CLUSTER_NAME}/hosts.yaml --become --become-user=root {WORKSPACE}/kubespray/cluster.yml
```

```console
ubuntu@ip-10-0-0-12:~$ ansible-playbook -i ${LINDAREX_WORKSPACE}/kubespray/inventory/lindarex-cluster/hosts.yaml --become --become-user=root ${LINDAREX_WORKSPACE}/kubespray/cluster.yml
```


### 7. kubernetes cluster 설치 확인

> master node에서 실행합니다.

#### 7.1. kubectl 설정

> root 사용자

```console
# mkdir -p $HOME/.kube
# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
# chown $(id -u):$(id -g) $HOME/.kube/config
```

> 일반 사용자

```console
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 7.2. node 목록 조회
```console
ubuntu@ip-10-0-0-12:~$ kubectl get nodes -o wide
NAME    STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
node1   Ready    master   7m45s   v1.19.7   10.0.0.12    <none>        Ubuntu 18.04.5 LTS   5.4.0-1041-aws   docker://19.3.14
node2   Ready    <none>   7m22s   v1.19.7   10.0.0.16    <none>        Ubuntu 18.04.5 LTS   5.4.0-1041-aws   docker://19.3.14
```

#### 7.3. pod 목록 조회
```console
ubuntu@ip-10-0-0-12:~$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-8b5ff5d58-k8j72   1/1     Running   0          8m40s
kube-system   calico-node-2n868                         1/1     Running   0          8m42s
kube-system   calico-node-wlqsq                         1/1     Running   0          8m42s
kube-system   coredns-85967d65-bwv44                    1/1     Running   0          8m20s
kube-system   coredns-85967d65-lfn9v                    1/1     Running   0          8m24s
kube-system   dns-autoscaler-5b7b5c9b6f-c7jrh           1/1     Running   0          8m21s
kube-system   kube-apiserver-node1                      1/1     Running   0          9m54s
kube-system   kube-apiserver-node2                      1/1     Running   0          9m35s
kube-system   kube-controller-manager-node1             1/1     Running   0          9m54s
kube-system   kube-controller-manager-node2             1/1     Running   0          9m34s
kube-system   kube-proxy-m5jqx                          1/1     Running   0          8m42s
kube-system   kube-proxy-pdcbl                          1/1     Running   0          8m42s
kube-system   kube-scheduler-node1                      1/1     Running   0          9m54s
kube-system   kube-scheduler-node2                      1/1     Running   1          9m34s
kube-system   nodelocaldns-9m4m9                        1/1     Running   0          8m20s
kube-system   nodelocaldns-dvj84                        1/1     Running   0          8m20s
```


### 8. (선택사항) kubespray 삭제

> master node에서 실행합니다.

```console
$ ansible-playbook -i {WORKSPACE}/kubespray/inventory/{CLUSTER_NAME}/hosts.yaml --become --become-user=root {WORKSPACE}/kubespray/reset.yml
```

```console
ubuntu@ip-10-0-0-12:~$ ansible-playbook -i ${LINDAREX_WORKSPACE}/kubespray/inventory/lindarex-cluster/hosts.yaml --become --become-user=root ${LINDAREX_WORKSPACE}/kubespray/reset.yml
```


## 마무리(CONCLUSION)
ubuntu 환경에 kubespray로 kubernetes cluster 설치를 완료했습니다. <br />
다음 포스트에서 kubernetes architectures 및 concepts, resources 등을 소개하겠습니다.


## 참고(REFERENCES)
- [https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray){: target="\_blank"}
- [https://kubernetes.io/ko/docs/setup/production-environment/tools/kubespray/](https://kubernetes.io/ko/docs/setup/production-environment/tools/kubespray/){: target="\_blank"}
