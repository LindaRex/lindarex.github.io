---
title: "우분투(Ubuntu) 환경에 Kubeadm으로 Kubernetes 클러스터(cluster) 설치하기"
categories: 
  - kubernetes
tags: 
  - kubernetes
  - cluster
  - "kubernetes cluster"
  - container
  - orchestration
  - "container orchestration"
  - docker
  - "open source software"
  - "open source"
  - kubeadm
  - oss
  - ubuntu
---


Kubernetes(kubernetes)는 컨테이너화된 애플리케이션(application)의 배포(deployment), 확장(scaling) 및 관리(management)를 자동화하는 Apache License 2.0이 적용되는 오픈소스(open source) 컨테이너(container) 오케스트레이션(orchestration) 플랫폼(platform)입니다. <br /><br />
kubernetes는 구글(Google)이 container management를 위해 개발한 Borg를 시작으로 container 운영 경험과 커뮤니티의 아이디어가 결합하여 개발되었습니다. <br /><br />
kubernetes는 리눅스(linux) container를 실행하는 호스트(host) 그룹을 함께 클러스터링(clustering) 할 수 있으며, 이러한 cluster를 효율적으로 관리할 수 있습니다. <br /><br />
이 포스트에서는 우분투(ubuntu) 환경에서 Kubeadm(kubeadm)을 사용하여 kubernetes cluster를 설치하는 방법을 소개합니다.

> kubernetes의 아키텍처(architectures) 및 개념(concepts), 리소스(resources) 등의 부가 설명은 이 포스트에서 다루지 않으며, 다음 포스트에서 kubernetes에 대한 자세한 설명을 소개하겠습니다.


## 선행조건(PREREQUISITE)
- 최소 2개(마스터, 워커 각각 1개)의 ubuntu 환경이 필요합니다.
- 방화벽 설정이 필요합니다.

> 마스터(master)는 master 노드(node) 또는 컨트롤 플레인(control plane) 또는 API 서버(server)로 불리며, 워커(worker)는 worker node 또는 node로 불립니다.

> ubuntu 설치 방법은 [우분투(Ubuntu) 서버(Server) 18.04 설치하기](https://lindarex.github.io/ubuntu/ubuntu-1804-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

> kubernetes 최소 설치 권장 사항은 [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin){: target="\_blank"} 페이지를 참고하시기 바랍니다.

> 방화벽 설정 방법은 [우분투(Ubuntu) 환경에 방화벽(Firewalld) 설치 및 설정하기](https://lindarex.github.io/ubuntu/ubuntu-firewalld-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

> kubernetes 실행에 필요한 필수 포트는 [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports){: target="\_blank"} 페이지를 참고하시기 바랍니다.

> kubernetes(정확히는 Pod) 실행을 위해서는 모든 node에 container 런타임(runtime)이 설치되어 있어야 하며, kubernetes에서 지원하는 container runtime은 containerd, CRI-O, 도커(docker) 등이 있습니다. 이 포스트에서는 docker를 기준으로 설명합니다. 

> container runtime에 대한 자세한 사항은 [https://kubernetes.io/docs/setup/production-environment/container-runtimes/](https://kubernetes.io/docs/setup/production-environment/container-runtimes/){: target="\_blank"} 페이지를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- AWS EC2 instance
    + Ubuntu Server 18.04 LTS (HVM), EBS General Purpose (SSD) Volume Type (64비트 Arm)
    + m5.large (ECU, 2 vCPUs, 3.1 GHz, 8 GiB 메모리, EBS 전용)
- Ubuntu 18.04.5 LTS (GNU/Linux 5.4.0-1038-aws x86_64)
- kubernetes 1.20.5
- docker 20.10.5


## 요약(SUMMARY)
1. docker 설치
2. 패키지(package)로 kubeadm, kubelet, kubectl 설치
3. master node에 kubeadm 설치
4. worker node를 master node에 연결
5. kubernetes cluster 설치 확인


## 내용(CONTENTS)
### 1. docker 설치

> master node와 worker node에서 실행합니다.

#### 1.1. package index 업데이트 
```console
# apt update
```

#### 1.2. package dependencies 설치
```console
# apt install -y apt-transport-https ca-certificates curl gnupg lsb-release
```

#### 1.3. doocker official GPG key 내려받기
```console
# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

#### 1.4. packages repository 설정 및 package index 업데이트
```console
# echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
# apt update
```

#### 1.5. 설치 가능한 docker 버전 조회

> 아래 목록은 2021년 3월 2일 기준입니다.

```console
# apt-cache madison docker-ce
 ---
 docker-ce | 5:20.10.5~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:20.10.4~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:20.10.3~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:20.10.2~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:20.10.1~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:20.10.0~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.15~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.14~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.13~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.12~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.11~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.10~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.9~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.8~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.7~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.6~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.5~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.4~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.3~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.2~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.1~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.0~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.9~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.8~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.7~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.6~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.5~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.4~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.3~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.2~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.1~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.0~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 18.06.3~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 18.06.2~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 18.06.1~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 18.06.0~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 18.03.1~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 ---
```

#### 1.6. docker 엔진(engine) 설치

> \# apt install -y docker-ce={VERSION_STRING} docker-ce-cli={VERSION_STRING} containerd.io

```console
# apt install -y docker-ce=5:20.10.5~3-0~ubuntu-bionic docker-ce-cli=5:20.10.5~3-0~ubuntu-bionic containerd.io
```

#### 1.7. (선택사항) docker 그룹에 사용자 계정 추가

> \# usermod -aG docker {USER}

```console
# usermod -aG docker rex
```

#### 1.8. docker 설치 확인
##### 1.8.1. docker 테스트 이미지(image) 실행
```console
# docker run hello-world
```

##### 1.8.2. docker container 목록 조회
```console
# docker ps -a
CONTAINER ID   IMAGE                    COMMAND                  CREATED             STATUS                      PORTS     NAMES
76735cb09a1e   hello-world              "/hello"                 41 seconds ago      Exited (0) 40 seconds ago             jovial_edison
```

##### 1.8.3. docker 버전 확인
```console
# docker -v
Docker version 20.10.5, build 55c4c88
```


### 2. package로 kubeadm, kubelet, kubectl 설치

> master node와 worker node에서 실행합니다.

#### (선택사항) 2.1. package index 업데이트 
```console
# apt update
```

#### (선택사항) 2.2. package dependencies 설치
```console
# apt install -y apt-transport-https ca-certificates curl
```

#### 2.3. Google Cloud public signing key 내려받기
```console
# curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

#### 2.4. packages repository 설정 및 package index 업데이트
```console
# echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list
# apt update
```

#### 2.5. kubelet, kubeadm, kubectl 설치
```console
# apt install -y kubelet kubeadm kubectl
```

#### 2.6. (선택사항) kubelet, kubeadm, kubectl 버전 유지 설정
```console
# apt-mark hold kubelet kubeadm kubectl
```

#### 2.7. kubelet, kubeadm, kubectl 설치 확인
##### 2.7.1. kubelet 버전 확인
```console
# kubelet --version
kubernetes v1.20.5
```

##### 2.7.2. kubeadm 버전 확인
```console
# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.5", GitCommit:"6b1d87acf3c8253c123756b9e61dac642678305f", GitTreeState:"clean", BuildDate:"2021-03-18T01:08:27Z", GoVersion:"go1.15.8", Compiler:"gc", Platform:"linux/amd64"}
```

##### 2.7.3. kubectl 버전 확인
```console
# kubectl version
Client Version: version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.5", GitCommit:"6b1d87acf3c8253c123756b9e61dac642678305f", GitTreeState:"clean", BuildDate:"2021-03-18T01:10:43Z", GoVersion:"go1.15.8", Compiler:"gc", Platform:"linux/amd64"}
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```


### 3. master node에 kubeadm 설치

> master에서 실행합니다.

#### 3.1. kubeadm init 명령어로 master node 초기화 (initializing)

```console
# kubeadm init --pod-network-cidr=10.244.0.0/16
```

> 위 '--pod-network-cidr=10.244.0.0/16' 옵션은 pod 통신을 위해 pod network add-on인 flannel CNI 설정입니다.

> flannel은 레이어(layer) 3 네트워크(network) 패브릭(fabric)을 구성하는 kubernetes를 위한 오버레이(overlay) network입니다. 자세한 설명은 [https://github.com/flannel-io/flannel](https://github.com/flannel-io/flannel){: target="\_blank"} 페이지를 참고하시기 바랍니다.

> CNI((Container Network Interface, cni)는 linux container에서 network interface를 구성하기 위해 필요한 플러그인(plugin)에 대한 사양(specification) 및 라이브러리(library)로 구성된 CNCF(Cloud Native Computing Foundation)에서 관리하는 프로젝트이며, GO 언어로 개발되었습니다. 자세한 설명은 [https://github.com/containernetworking/cni](https://github.com/containernetworking/cni){: target="\_blank"} 페이지를 참고하시기 바랍니다.

> 초기화 완료 후 아래와 같이 화면 하단에 출력되는 'kubeadm join' 명령어는 worker node 연결을 위해 필요하오니 별도로 저장해 두시기 바랍니다.

```console
kubeadm join 10.0.0.11:6443 --token qabzp1.86v3o2nfpf6qh4eu \
    --discovery-token-ca-cert-hash sha256:3f08ac5ef2d0592147e402be885cc7d39550af5b903a6d229e01358b45e17e0d
```

#### 3.2. kubectl 설정

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

#### 3.3. master node의 pod 목록 조회

> 아래 결과는 flannel 설치 전이기 때문에 coredns pod가 pending 상태입니다.

```console
# kubectl get pods -n kube-system
NAME                                    READY   STATUS     RESTARTS   AGE
coredns-74ff55c5b-jz6tz                 0/1     Pending    0          5m40s
coredns-74ff55c5b-ksnfl                 0/1     Pending    0          5m40s
etcd-ip-10-0-0-162                      1/1     Running    0          5m55s
kube-apiserver-ip-10-0-0-162            1/1     Running    0          5m55s
kube-controller-manager-ip-10-0-0-162   1/1     Running    0          5m55s
kube-proxy-rjqkc                        1/1     Running    0          5m40s
kube-scheduler-ip-10-0-0-162            1/1     Running    0          5m55s
```

#### 3.4. flannel 설치
```console
# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

#### 3.5. flannel 설치 확인
```console
# kubectl get pods -n kube-system
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-74ff55c5b-jz6tz                 1/1     Running   0          6m2s
coredns-74ff55c5b-ksnfl                 1/1     Running   0          6m2s
etcd-ip-10-0-0-162                      1/1     Running   0          6m17s
kube-apiserver-ip-10-0-0-162            1/1     Running   0          6m17s
kube-controller-manager-ip-10-0-0-162   1/1     Running   0          6m17s
kube-flannel-ds-dg45x                   1/1     Running   0          30s
kube-proxy-rjqkc                        1/1     Running   0          6m2s
kube-scheduler-ip-10-0-0-162            1/1     Running   0          6m17s
```

#### 3.6. kubernetes cluster 설치 확인

> 아래 결과는 worker node 연결 전이기 때문에 master node만 조회됩니다.

```console
# kubectl get nodes -o wide
NAME            STATUS     ROLES                  AGE    VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
ip-10-0-0-162   Ready      control-plane,master   8m2s   v1.20.5   10.0.0.11    <none>        Ubuntu 18.04.5 LTS   5.4.0-1041-aws   docker://20.10.5
```


### 4. worker node를 master node에 연결

> worker node에서 실행합니다.

#### 4.1. kubeadm join 명령어로 worker node를 master node에 연결
```console
# kubeadm join 10.0.0.11:6443 --token qabzp1.86v3o2nfpf6qh4eu \
    --discovery-token-ca-cert-hash sha256:3f08ac5ef2d0592147e402be885cc7d39550af5b903a6d229e01358b45e17e0d
[preflight] Running pre-flight checks
  [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
  [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.5. Latest validated version: 19.03
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```


### 5. kubernetes cluster 설치 확인

> master에서 실행합니다.

#### 5.1. 전체 nodes 조회
```console
# kubectl get nodes -o wide
NAME            STATUS   ROLES                  AGE    VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
ip-10-0-0-104   Ready    <none>                 36s    v1.20.5   10.0.0.12    <none>        Ubuntu 18.04.5 LTS   5.4.0-1041-aws   docker://20.10.5
ip-10-0-0-162   Ready    control-plane,master   8m8s   v1.20.5   10.0.0.11    <none>        Ubuntu 18.04.5 LTS   5.4.0-1041-aws   docker://20.10.5
```

#### 5.2. 전체 namespaces의 pod 목록 조회
```console
# kubectl get pods --all-namespaces
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
kube-system   coredns-74ff55c5b-jz6tz                 1/1     Running   0          106m
kube-system   coredns-74ff55c5b-ksnfl                 1/1     Running   0          106m
kube-system   etcd-ip-10-0-0-162                      1/1     Running   0          106m
kube-system   kube-apiserver-ip-10-0-0-162            1/1     Running   0          106m
kube-system   kube-controller-manager-ip-10-0-0-162   1/1     Running   0          106m
kube-system   kube-flannel-ds-2kd4z                   1/1     Running   0          99m
kube-system   kube-flannel-ds-dg45x                   1/1     Running   0          100m
kube-system   kube-proxy-rjqkc                        1/1     Running   0          106m
kube-system   kube-proxy-vzdqn                        1/1     Running   0          99m
kube-system   kube-scheduler-ip-10-0-0-162            1/1     Running   0          106m
```


## 마무리(CONCLUSION)
ubuntu 환경에 kubeadm으로 kubernetes cluster 설치를 완료했습니다. <br />
다음 포스트에서 kubernetes architectures 및 concepts, resources 등을 소개하겠습니다.


## 참고(REFERENCES)
- [https://kubernetes.io/ko/docs/setup/production-environment/tools/kubeadm/install-kubeadm/](https://kubernetes.io/ko/docs/setup/production-environment/tools/kubeadm/install-kubeadm/){: target="\_blank"}
- [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/){: target="\_blank"}
