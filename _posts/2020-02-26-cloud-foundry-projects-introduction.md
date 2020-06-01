---
title: "클라우드 파운드리(Cloud Foundry) 프로젝트(projects) 소개"
categories: 
  - cfar
tags: 
  - "cloud foundry"
  - cfar
  - cfcr
  - bosh
  - quarks
  - eirini
  - kubernetes
  - paas
  - "open source software"
  - "open source"
  - oss
---


이 포스트에서는 클라우드 파운드리(Cloud Foundry) 프로젝트(projects)를 간단히 소개합니다.


## 내용(CONTENTS)
### 1. BOSH
- BOSH는 복잡한 분산 시스템의 release engineering, 배포(deploy) 및 애플리케이션(application) 수명 주기(lifecycle) 관리를 위한 클라우드(cloud) 오픈소스(open source) 소프트웨어입니다.
- BOSH는 가상 머신(VM, virtual machine)이나 컨테이너(container), 베어 메탈(bare metal) 등 어디에 deploy 하든 상관없이 동일한 접근 방식을 취하고, 대부분의 application을 deploy 할 때 BOSH가 사용됩니다.
- BOSH는 백그라운드에서 작동하며, 구성요소(component)가 변경될 때, application을 최신 상태로 유지하고, 환경이 올바르게 구성되었는지 확인하여 환경이 정의되었을 때 실행 방식을 조정하도록 재구성합니다.
- BOSH는 스템셀(stemcell), 릴리스(release) 및 배포 매니페스트(deployment manifest)를 사용하여 cloud 환경의 무결성(integrity)을 유지합니다.
    + stemcell
        - VM을 생성하는 데 사용되는 골든 운영체제 이미지(golden operating system image)와 유사합니다.
        - 배포에 포함된 기본 운영체제(OS, operating system)를 다른 application 패키지(package)와 분리합니다.
    + release
        - stemcell 위에 놓인 레이어(layer)로 어떤 application을 deploy하고 어떻게 구성해야 하는지 BOSH에 application을 패키지화하여 전달합니다.
        - release는 구성 속성(configuration properties) 및 템플릿(templates), 시작(startup) scripts, 소스 코드(source code), 바이너리(binary) 아티팩트(artifacts)를 포함하며, 재현 가능한 방식(reproducible)으로 application을 build하고 deploy 하는 데 필요한 모든 것이 포함됩니다.
    + deployment manifest
        - 어떤 cloud 또는 인프라(infra)에 어떤 BOSH release를 deploy하고, 어떻게 구성해야 하는지 설명하는 yaml file입니다.
        - BOSH는 manifest file을 사용하여 대상 infra에 deploy하고, VM 또는 container의 상태를 모니터링하고, 필요시에 복구합니다.
- BOSH는 CPI(cloud provider interface) 모델(model)을 포함하며, 이는 멀티(multi) cloud 기능(capability)의 핵심입니다.
- BOSH는 CPI를 사용하여 AWS, Azure, GCP, OpenStack, VMware vSphere 등을 포함한 여러 cloud 환경에 application을 deploy 할 수 있습니다.
- CPI는 BOSH가 infra와 상호 작용하여 stemcell, VM 및 디스크(disk)를 생성하고 관리하는 데 사용하는 API입니다.
- BOSH는 초기에 CFAR을 배포하기 위해 생성되었지만, 현재는 다른 환경에서 모든 종류의 application을 package하고 관리하기 위해 사용되고 있습니다. 

> Bare metal이란 소프트웨어가 설치되어 있지 않은 단일 테넌트(single-tenant) 물리적 하드웨어를 의미하며, 가상화 및 cloud 호스팅(hosting)과 구별하기 위해 사용됩니다.
<br /><br />
bare metal에 대한 자세한 정보는 [https://en.wikipedia.org/wiki/Bare-metal_server](https://en.wikipedia.org/wiki/Bare-metal_server){: target="\_blank"}를 확인해 주시기 바랍니다.


### 2. CFAR(cloud foundry application runtime)
- CFAR은 클라우드 환경과 개발 프레임워크, application 서비스(service)를 선택할 수 있으며, application을 빠르고 쉽게 구축하고 테스트하여 deploy 할 수 있는, 완전한 application lifecycle을 지원하는 open source multi cloud application PaaS(platform as a service)입니다.

> CFAR에 대한 자세한 설명은 [클라우드 파운드리(Cloud Foundry, CFAR) 소개](https://lindarex.github.io/cfar/cloud-foundry-cfar-introduction/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

### 3. CFCR(cloud foundry container runtime)
- CFCR은 kubernetes를 사용하여 container를 더 세밀하게 제어하고 관리할 수 ​​있는 기능을 제공합니다.
- CFCR은 과거에 Cloud Foundry Foundation 내의 incubating project인 Project Kubo였는데, container platform의 성장과 함께 container runtime인 kubo를 CFCR로 명칭을 변경했습니다.
- CFCR은 2017년 11월에 kubernetes 적합성 인증(Certified Kubernetes)을 받았고, BOSH를 사용하여 cloud platform에서 고가용성(high availability)의 kubernetes 클러스터(cluster)를 인스턴스화하고 deploy 및 관리할 수 있는 기능을 제공합니다.
- kubernetes와 BOSH의 조합으로 BOSH를 통해 deploy 및 lifecycle를 관리하면, kubernetes cluster의 스케일링(scaling)과 VM healing, 롤링 업그레이드(rolling upgrade)를 할 수 있습니다.

> Certified Kubernetes란 CNCF(Cloud Native Computing Foundation)의 Kubernetes Software Conformance Certification Program으로 kubernetes의 목표인 일관성과 휴대성을 위한 검토와 인증에 대한 적합성 시험 결과를 CNCF로 제출하여, CNCF로부터 준수 구현을 공식적으로 인증받은 것을 의미합니다.
<br /><br />
Certified Kubernetes에 대한 자세한 정보는 [https://www.cncf.io/certification/software-conformance/](https://www.cncf.io/certification/software-conformance/){: target="\_blank"}를 확인해 주시기 바랍니다.

### 4. Quarks
- Project Quarks(quarks)는 Cloud Foundry Foundation 내의 incubating project로, CFAR을 VM 대신 container로 패키징하여 kubernetes에 쉽게 deploy 할 수 있는 기능을 제공합니다. 
- container형 CFAR은 BOSH로 deploy 한 CFAR과 동일한 기능을 제공하고, 더 적은 infra 용량으로 kubernetes 운영자에게 익숙한 환경을 제공합니다.
- quarks는 기존 BOSH release를 kubernetes에 설치하기 위해 docker 이미지(image) 또는 helm charts로 변환합니다.
- kubernetes는 CFAR instance의 자동 확장 및 실패한 노드(node)를 복구하는 기능을 제공합니다. 
- kubernetes를 기본 infra로 사용하면 quarks를 모든 private 또는 public cloud에 쉽게 deploy 할 수 있습니다.

> helm은 Kubernetes의 package 관리자이며, helm을 사용하면 application 및 리소스(resources)를 Kubernetes cluster에 쉽게 설치할 수 있습니다.
<br /><br />
helm charts는 Kubernetes resources 세트를 Kubernetes cluster에 설치하기 위한 helm package이며, helm charts에는 chart.yaml, templates, values.yaml 및 의존성(dependencies)이 포함됩니다.
<br /><br />
helm에 대한 자세한 정보는 [https://helm.sh/](https://helm.sh/){: target="\_blank"}를 확인해 주시기 바랍니다.

### 5. Eirini
- Project Eirini(eirini)는 Cloud Foundry Foundation 내의 incubating project로, CFAR을 위한 플러그형 스케줄링을 가능하게 하며 application container instance를 조정하기 위해 Diego/Garden 또는 Kubernetes 중에서 선택할 수 있습니다. 
- eirini는 기존 Kubernetes cluster infra를 재사용하여 CFAR에 의해 deploy 된 application을 hosting 합니다.
- 개발자가 buildpack을 사용하여 application을 push 하면, kubernetes 내부에서 일회성 스테이징 작업이 포드(pod)로 실행되고, 드롭릿(droplet)이 동일한 방식으로 생성하여 업로드됩니다. 이 과정에서 application은 docker image로 다운로드되어 kubernetes에 deploy 됩니다.
- eirini는 CFAR 운영자가 kubernetes를 container 스케줄러로 선택할 수 있도록 하여, kubernetes에 익숙한 조직이 CFAR을 보다 쉽게 이용할 수 있도록 합니다.

### 6. Open Service Broker API
- Open Service Broker API project를 통해 독립적인 application vendor와 SaaS provider는 CFAR과 kubernetes와 같은 cloud native platform에서 실행되는 워크로드(workload)에 대한 백 앤드 service를 쉽게 제공할 수 있습니다. 
- Open Service Broker API는 많은 platform과 service provider가 채택하였고, Google, IBM, Pivotal, Red Hat, SAP 등 다수의 cloud 회사들이 기여했습니다.
- Open Service Broker API 사양의 service broker는 service lifecycle을 관리하고, platform은 service broker와 상호 작용하여 service를 provisioning 및 access하고 관리합니다.


## 마무리(CONCLUSION)
Cloud Foundry projects를 살펴보았습니다.
<br /><br />
BOSH는 Cloud Foundry projects에 기반이 되는 프로젝트이고, CFAR을 제외한 나머지 CFCR, Quarks, Eirini 등의 공통 키워드는 Kubernetes입니다. 더 말이 필요 없는 kubernetes... <br />
다음 포스트에서는 kubernetes 소개와 CFAR을 설치하는 방법을 소개하겠습니다.


## 참고(REFERENCES)
- [https://www.cloudfoundry.org/](https://www.cloudfoundry.org/){: target="\_blank"}
- [https://www.cncf.io/](https://www.cncf.io/){: target="\_blank"}
- [https://github.com/cncf/k8s-conformance](https://github.com/cncf/k8s-conformance){: target="\_blank"}
