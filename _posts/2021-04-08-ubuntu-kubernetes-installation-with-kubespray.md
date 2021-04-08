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
ubuntu@ip-10-0-0-12:~$ declare -a IPS=(10.0.0.12 10.0.0.16)
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
ubuntu@ip-10-0-0-12:~$ cat ${LINDAREX_WORKSPACE}/kubespray/inventory/lindarex-cluster/group_vars/all/all.yml
---
## Directory where etcd data stored
etcd_data_dir: /var/lib/etcd

## Experimental kubeadm etcd deployment mode. Available only for new deployment
etcd_kubeadm_enabled: false

## Directory where the binaries will be installed
bin_dir: /usr/local/bin

## The access_ip variable is used to define how other nodes should access
## the node.  This is used in flannel to allow other flannel nodes to see
## this node for example.  The access_ip is really useful AWS and Google
## environments where the nodes are accessed remotely by the "public" ip,
## but don't know about that address themselves.
# access_ip: 1.1.1.1


## External LB example config
## apiserver_loadbalancer_domain_name: "elb.some.domain"
# loadbalancer_apiserver:
#   address: 1.2.3.4
#   port: 1234

## Internal loadbalancers for apiservers
# loadbalancer_apiserver_localhost: true
# valid options are "nginx" or "haproxy"
# loadbalancer_apiserver_type: nginx  # valid values "nginx" or "haproxy"

## Local loadbalancer should use this port
## And must be set port 6443
loadbalancer_apiserver_port: 6443

## If loadbalancer_apiserver_healthcheck_port variable defined, enables proxy liveness check for nginx.
loadbalancer_apiserver_healthcheck_port: 8081

### OTHER OPTIONAL VARIABLES

## Upstream dns servers
# upstream_dns_servers:
#   - 8.8.8.8
#   - 8.8.4.4

## There are some changes specific to the cloud providers
## for instance we need to encapsulate packets with some network plugins
## If set the possible values are either 'gce', 'aws', 'azure', 'openstack', 'vsphere', 'oci', or 'external'
## When openstack is used make sure to source in the openstack credentials
## like you would do when using openstack-client before starting the playbook.
# cloud_provider:

## When cloud_provider is set to 'external', you can set the cloud controller to deploy
## Supported cloud controllers are: 'openstack' and 'vsphere'
## When openstack or vsphere are used make sure to source in the required fields
# external_cloud_provider:

## Set these proxy values in order to update package manager and docker daemon to use proxies
# http_proxy: ""
# https_proxy: ""

## Refer to roles/kubespray-defaults/defaults/main.yml before modifying no_proxy
# no_proxy: ""

## Some problems may occur when downloading files over https proxy due to ansible bug
## https://github.com/ansible/ansible/issues/32750. Set this variable to False to disable
## SSL validation of get_url module. Note that kubespray will still be performing checksum validation.
# download_validate_certs: False

## If you need exclude all cluster nodes from proxy and other resources, add other resources here.
# additional_no_proxy: ""

## If you need to disable proxying of os package repositories but are still behind an http_proxy set
## skip_http_proxy_on_os_packages to true
## This will cause kubespray not to set proxy environment in /etc/yum.conf for centos and in /etc/apt/apt.conf for debian/ubuntu
## Special information for debian/ubuntu - you have to set the no_proxy variable, then apt package will install from your source of wish
# skip_http_proxy_on_os_packages: false

## Since workers are included in the no_proxy variable by default, docker engine will be restarted on all nodes (all
## pods will restart) when adding or removing workers.  To override this behaviour by only including master nodes in the
## no_proxy variable, set below to true:
no_proxy_exclude_workers: false

## Certificate Management
## This setting determines whether certs are generated via scripts.
## Chose 'none' if you provide your own certificates.
## Option is  "script", "none"
## note: vault is removed
# cert_management: script

## Set to true to allow pre-checks to fail and continue deployment
# ignore_assert_errors: false

## The read-only port for the Kubelet to serve on with no authentication/authorization. Uncomment to enable.
# kube_read_only_port: 10255

## Set true to download and cache container
# download_container: true

## Deploy container engine
# Set false if you want to deploy container engine manually.
# deploy_container_engine: true

## Red Hat Enterprise Linux subscription registration
## Add either RHEL subscription Username/Password or Organization ID/Activation Key combination
## Update RHEL subscription purpose usage, role and SLA if necessary
# rh_subscription_username: ""
# rh_subscription_password: ""
# rh_subscription_org_id: ""
# rh_subscription_activation_key: ""
# rh_subscription_usage: "Development"
# rh_subscription_role: "Red Hat Enterprise Server"
# rh_subscription_sla: "Self-Support"

## Check if access_ip responds to ping. Set false if your firewall blocks ICMP.
# ping_access_ip: true
```

```console
$ cat {WORKSPACE}/kubespray/inventory/{CLUSTER_NAME}/group_vars/k8s-cluster/k8s-cluster.yml
```

```console
ubuntu@ip-10-0-0-12:~$ cat ${LINDAREX_WORKSPACE}/kubespray/inventory/lindarex-cluster/group_vars/k8s-cluster/k8s-cluster.yml
---
# Kubernetes configuration dirs and system namespace.
# Those are where all the additional config stuff goes
# the kubernetes normally puts in /srv/kubernetes.
# This puts them in a sane location and namespace.
# Editing those values will almost surely break something.
kube_config_dir: /etc/kubernetes
kube_script_dir: "{{ bin_dir }}/kubernetes-scripts"
kube_manifest_dir: "{{ kube_config_dir }}/manifests"

# This is where all the cert scripts and certs will be located
kube_cert_dir: "{{ kube_config_dir }}/ssl"

# This is where all of the bearer tokens will be stored
kube_token_dir: "{{ kube_config_dir }}/tokens"

kube_api_anonymous_auth: true

## Change this to use another Kubernetes version, e.g. a current beta release
kube_version: v1.19.7

# Where the binaries will be downloaded.
# Note: ensure that you've enough disk space (about 1G)
local_release_dir: "/tmp/releases"
# Random shifts for retrying failed ops like pushing/downloading
retry_stagger: 5

# This is the group that the cert creation scripts chgrp the
# cert files to. Not really changeable...
kube_cert_group: kube-cert

# Cluster Loglevel configuration
kube_log_level: 2

# Directory where credentials will be stored
credentials_dir: "{{ inventory_dir }}/credentials"

## It is possible to activate / deactivate selected authentication methods (oidc, static token auth)
# kube_oidc_auth: false
# kube_token_auth: false


## Variables for OpenID Connect Configuration https://kubernetes.io/docs/admin/authentication/
## To use OpenID you have to deploy additional an OpenID Provider (e.g Dex, Keycloak, ...)

# kube_oidc_url: https:// ...
# kube_oidc_client_id: kubernetes
## Optional settings for OIDC
# kube_oidc_ca_file: "{{ kube_cert_dir }}/ca.pem"
# kube_oidc_username_claim: sub
# kube_oidc_username_prefix: oidc:
# kube_oidc_groups_claim: groups
# kube_oidc_groups_prefix: oidc:

## Variables to control webhook authn/authz
# kube_webhook_token_auth: false
# kube_webhook_token_auth_url: https://...
# kube_webhook_token_auth_url_skip_tls_verify: false

## For webhook authorization, authorization_modes must include Webhook
# kube_webhook_authorization: false
# kube_webhook_authorization_url: https://...
# kube_webhook_authorization_url_skip_tls_verify: false

# Choose network plugin (cilium, calico, weave or flannel. Use cni for generic cni plugin)
# Can also be set to 'cloud', which lets the cloud provider setup appropriate routing
kube_network_plugin: calico

# Setting multi_networking to true will install Multus: https://github.com/intel/multus-cni
kube_network_plugin_multus: false

# Kubernetes internal network for services, unused block of space.
kube_service_addresses: 10.233.0.0/18

# internal network. When used, it will assign IP
# addresses from this range to individual pods.
# This network must be unused in your network infrastructure!
kube_pods_subnet: 10.233.64.0/18

# internal network node size allocation (optional). This is the size allocated
# to each node for pod IP address allocation. Note that the number of pods per node is
# also limited by the kubelet_max_pods variable which defaults to 110.
#
# Example:
# Up to 64 nodes and up to 254 or kubelet_max_pods (the lowest of the two) pods per node:
#  - kube_pods_subnet: 10.233.64.0/18
#  - kube_network_node_prefix: 24
#  - kubelet_max_pods: 110
#
# Example:
# Up to 128 nodes and up to 126 or kubelet_max_pods (the lowest of the two) pods per node:
#  - kube_pods_subnet: 10.233.64.0/18
#  - kube_network_node_prefix: 25
#  - kubelet_max_pods: 110
kube_network_node_prefix: 24

# The port the API Server will be listening on.
kube_apiserver_ip: "{{ kube_service_addresses|ipaddr('net')|ipaddr(1)|ipaddr('address') }}"
kube_apiserver_port: 6443  # (https)
# kube_apiserver_insecure_port: 8080  # (http)
# Set to 0 to disable insecure port - Requires RBAC in authorization_modes and kube_api_anonymous_auth: true
kube_apiserver_insecure_port: 0  # (disabled)

# Kube-proxy proxyMode configuration.
# Can be ipvs, iptables
kube_proxy_mode: ipvs

# configure arp_ignore and arp_announce to avoid answering ARP queries from kube-ipvs0 interface
# must be set to true for MetalLB to work
kube_proxy_strict_arp: false

# A string slice of values which specify the addresses to use for NodePorts.
# Values may be valid IP blocks (e.g. 1.2.3.0/24, 1.2.3.4/32).
# The default empty string slice ([]) means to use all local addresses.
# kube_proxy_nodeport_addresses_cidr is retained for legacy config
kube_proxy_nodeport_addresses: >-
  {%- if kube_proxy_nodeport_addresses_cidr is defined -%}
  [{{ kube_proxy_nodeport_addresses_cidr }}]
  {%- else -%}
  []
  {%- endif -%}

# If non-empty, will use this string as identification instead of the actual hostname
# kube_override_hostname: >-
#   {%- if cloud_provider is defined and cloud_provider in [ 'aws' ] -%}
#   {%- else -%}
#   {{ inventory_hostname }}
#   {%- endif -%}

## Encrypting Secret Data at Rest (experimental)
kube_encrypt_secret_data: false

# DNS configuration.
# Kubernetes cluster name, also will be used as DNS domain
cluster_name: cluster.local
# Subdomains of DNS domain to be resolved via /etc/resolv.conf for hostnet pods
ndots: 2
# Can be coredns, coredns_dual, manual or none
dns_mode: coredns
# Set manual server if using a custom cluster DNS server
# manual_dns_server: 10.x.x.x
# Enable nodelocal dns cache
enable_nodelocaldns: true
nodelocaldns_ip: 169.254.25.10
nodelocaldns_health_port: 9254
# nodelocaldns_external_zones:
# - zones:
#   - example.com
#   - example.io:1053
#   nameservers:
#   - 1.1.1.1
#   - 2.2.2.2
#   cache: 5
# - zones:
#   - https://mycompany.local:4453
#   nameservers:
#   - 192.168.0.53
#   cache: 0
# Enable k8s_external plugin for CoreDNS
enable_coredns_k8s_external: false
coredns_k8s_external_zone: k8s_external.local
# Enable endpoint_pod_names option for kubernetes plugin
enable_coredns_k8s_endpoint_pod_names: false

# Can be docker_dns, host_resolvconf or none
resolvconf_mode: docker_dns
# Deploy netchecker app to verify DNS resolve as an HTTP service
deploy_netchecker: false
# Ip address of the kubernetes skydns service
skydns_server: "{{ kube_service_addresses|ipaddr('net')|ipaddr(3)|ipaddr('address') }}"
skydns_server_secondary: "{{ kube_service_addresses|ipaddr('net')|ipaddr(4)|ipaddr('address') }}"
dns_domain: "{{ cluster_name }}"

## Container runtime
## docker for docker, crio for cri-o and containerd for containerd.
container_manager: docker

# Additional container runtimes
kata_containers_enabled: false

## Settings for containerd runtimes (only used when container_manager is set to containerd)
#
# Settings for default containerd runtime
# containerd_default_runtime:
#   type: io.containerd.runtime.v1.linux
#   engine: ''
#   root: ''
#
# Settings for additional runtimes for containerd configuration
# containerd_runtimes:
#   - name: ""
#     type: ""
#     engine: ""
#     root: ""
# Example for Kata Containers as additional runtime:
# containerd_runtimes:
#   - name: kata
#     type: io.containerd.kata.v2
#     engine: ""
#     root: ""
#
# Settings for untrusted containerd runtime
# containerd_untrusted_runtime_type: ''
# containerd_untrusted_runtime_engine: ''
# containerd_untrusted_runtime_root: ''

kubeadm_certificate_key: "{{ lookup('password', credentials_dir + '/kubeadm_certificate_key.creds length=64 chars=hexdigits') | lower }}"

# K8s image pull policy (imagePullPolicy)
k8s_image_pull_policy: IfNotPresent

# audit log for kubernetes
kubernetes_audit: false

# dynamic kubelet configuration
dynamic_kubelet_configuration: false

# define kubelet config dir for dynamic kubelet
# kubelet_config_dir:
default_kubelet_config_dir: "{{ kube_config_dir }}/dynamic_kubelet_dir"
dynamic_kubelet_configuration_dir: "{{ kubelet_config_dir | default(default_kubelet_config_dir) }}"

# pod security policy (RBAC must be enabled either by having 'RBAC' in authorization_modes or kubeadm enabled)
podsecuritypolicy_enabled: false

# Custom PodSecurityPolicySpec for restricted policy
# podsecuritypolicy_restricted_spec: {}

# Custom PodSecurityPolicySpec for privileged policy
# podsecuritypolicy_privileged_spec: {}

# Make a copy of kubeconfig on the host that runs Ansible in {{ inventory_dir }}/artifacts
# kubeconfig_localhost: false
# Download kubectl onto the host that runs Ansible in {{ bin_dir }}
# kubectl_localhost: false

# A comma separated list of levels of node allocatable enforcement to be enforced by kubelet.
# Acceptable options are 'pods', 'system-reserved', 'kube-reserved' and ''. Default is "".
# kubelet_enforce_node_allocatable: pods

## Optionally reserve resources for OS system daemons.
# system_reserved: true
## Uncomment to override default values
# system_memory_reserved: 512Mi
# system_cpu_reserved: 500m
## Reservation for master hosts
# system_master_memory_reserved: 256Mi
# system_master_cpu_reserved: 250m

# An alternative flexvolume plugin directory
# kubelet_flexvolumes_plugins_dir: /usr/libexec/kubernetes/kubelet-plugins/volume/exec

## Supplementary addresses that can be added in kubernetes ssl keys.
## That can be useful for example to setup a keepalived virtual IP
# supplementary_addresses_in_ssl_keys: [10.0.0.1, 10.0.0.2, 10.0.0.3]

## Running on top of openstack vms with cinder enabled may lead to unschedulable pods due to NoVolumeZoneConflict restriction in kube-scheduler.
## See https://github.com/kubernetes-sigs/kubespray/issues/2141
## Set this variable to true to get rid of this issue
volume_cross_zone_attachment: false
## Add Persistent Volumes Storage Class for corresponding cloud provider (supported: in-tree OpenStack, Cinder CSI,
## AWS EBS CSI, Azure Disk CSI, GCP Persistent Disk CSI)
persistent_volumes_enabled: false

## Container Engine Acceleration
## Enable container acceleration feature, for example use gpu acceleration in containers
# nvidia_accelerator_enabled: true
## Nvidia GPU driver install. Install will by done by a (init) pod running as a daemonset.
## Important: if you use Ubuntu then you should set in all.yml 'docker_storage_options: -s overlay2'
## Array with nvida_gpu_nodes, leave empty or comment if you don't want to install drivers.
## Labels and taints won't be set to nodes if they are not in the array.
# nvidia_gpu_nodes:
#   - kube-gpu-001
# nvidia_driver_version: "384.111"
## flavor can be tesla or gtx
# nvidia_gpu_flavor: gtx
## NVIDIA driver installer images. Change them if you have trouble accessing gcr.io.
# nvidia_driver_install_centos_container: atzedevries/nvidia-centos-driver-installer:2
# nvidia_driver_install_ubuntu_container: gcr.io/google-containers/ubuntu-nvidia-driver-installer@sha256:7df76a0f0a17294e86f691c81de6bbb7c04a1b4b3d4ea4e7e2cccdc42e1f6d63
## NVIDIA GPU device plugin image.
# nvidia_gpu_device_plugin_container: "k8s.gcr.io/nvidia-gpu-device-plugin@sha256:0842734032018be107fa2490c98156992911e3e1f2a21e059ff0105b07dd8e9e"

## Support tls min version, Possible values: VersionTLS10, VersionTLS11, VersionTLS12, VersionTLS13.
# tls_min_version: ""

## Support tls cipher suites.
# tls_cipher_suites: {}
#   - TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
#   - TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
#   - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
#   - TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
#   - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
#   - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
#   - TLS_ECDHE_ECDSA_WITH_RC4_128_SHA
#   - TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA
#   - TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
#   - TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
#   - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
#   - TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
#   - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
#   - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
#   - TLS_ECDHE_RSA_WITH_RC4_128_SHA
#   - TLS_RSA_WITH_3DES_EDE_CBC_SHA
#   - TLS_RSA_WITH_AES_128_CBC_SHA
#   - TLS_RSA_WITH_AES_128_CBC_SHA256
#   - TLS_RSA_WITH_AES_128_GCM_SHA256
#   - TLS_RSA_WITH_AES_256_CBC_SHA
#   - TLS_RSA_WITH_AES_256_GCM_SHA384
#   - TLS_RSA_WITH_RC4_128_SHA

## Amount of time to retain events. (default 1h0m0s)
event_ttl_duration: "1h0m0s"
##  Force regeneration of kubernetes control plane certificates without the need of bumping the cluster version
force_certificate_regeneration: false
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
$ kubectl get pods -n kube-system
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
