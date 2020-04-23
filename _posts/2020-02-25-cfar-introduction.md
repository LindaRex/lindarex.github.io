---
title: "클라우드 파운드리(Cloud Foundry, CFAR) 소개"
categories: 
  - cfar
tags: 
  - "cloud foundry"
  - cfar
  - paas
---


클라우드 파운드리(Cloud Foundry)는 과거에 PaaS(platform as a service)와 재단을 가리키는 용어였지만, 현재는 PaaS를 뜻하는 Cloud Foundry는 CFAR(cloud foundry application runtime)로 대체되었습니다. <br />
CFAR은 개방형 기여(contribution) 및 개방형 거버넌스 모델(governance model)을 갖춘 Apache License 2.0으로 배포된 오픈소스 클라우드 애플리케이션 플랫폼(open source cloud application platform)입니다. <br />
사용자가 업체(vendor)에 종속되지 않도록 유연성을 제공하고, 애플리케이션(application)을 더 빠르고 쉽게 구축(build)하여 테스트(test)하고 배포(deploy)하며 확장할 수 있는 기능을 제공합니다. <br />
이 포스트에서는 CFAR을 소개합니다.


## 내용(CONTENTS)
- CFAR은 클라우드(IaaS) 환경, 개발 프레임워크(Buildpack) 및 application 서비스(Service)를 선택할 수 있으며, application을 빠르고 쉽게 build, test하고 deploy 하며 확장할 수 있는 오픈소스(open source) 멀티 클라우드(cloud) application PaaS입니다.
- CFAR의 컨테이너(container) 기반 아키텍처는 아마존 웹 서비스(AWS), 마이크로소프트 애저(Azure), 구글 컴퓨트 플랫폼(GCP), 오픈스택(OpenStack), VM웨어(VMware) vSphere 등의 다양한 cloud 환경 위에서 다양한 언어로 application을 실행할 수 있도록 합니다.
- CFAR은 초기 개발부터 test, 그리고 deploy에 이르는 완전한 application 수명 주기(lifecycle)를 지원하기 때문에 지속적 배포(CD, continuous delivery)에 적합합니다. 
- CFAR은 루비(Ruby), 고언어(Go), 자바(Java)로 개발되었으며, BOSH deployment 스크립트(script)를 이용하여 IaaS에 deploy 합니다.

> 국내에서는 한국정보화진흥원(NIA, National Information Society Agency)에서 2014년부터 CFAR을 이용한 PaaS 아키텍처와 기능 분석을 시작했고, 2016년에 CFAR과 개발 도구 및 운영 도구를 패키징한 [PaaS-TA](https://paas-ta.kr/){: target="\_blank"}라는 open source PaaS 플랫폼(platform)을 공개했습니다. 2016년에 버전 1.0을 시작으로 다양한 서비스(service)와 버전 업그레이드를 통해 코스콤, KT, LG CNS, SK 등 민간 기업과 협력하면서 2019년에 버전 5.0까지 배포되었습니다.

> open source project인 CFAR을 상용화한 Pivotal Cloud Foundry(PCF)와 IBM Cloud(IBM Bluemix) 등은 전 세계적으로 TOP 10 안에 드는 PaaS platform이며, Public, Private 또는 하이브리드(Hybrid) IaaS 환경에서 모두 활용 가능합니다.

> IaaS(infrastructure as a service)란 서버(server) 자원(resource), 네트워크(network), 스토리지(storage) 등 인프라(infra) resources을 쉽고 편하게 이용할 수 있는 cloud service 형태로 제공하는 것을 의미하며, PaaS와 SaaS(software as a service)의 기반입니다. 
server 가상화, 데스크톱 가상화 등의 기술로 구현되며, 대표적으로 AWS EC2(Elastic Cloud Compute), MS Azure, Google compute engine, OpenStack 등이 있습니다.

> PaaS란 platform as a service의 약자로 application 개발을 위한 infra를 구축하고 유지보수하는 복잡함 없이, application을 개발하고 실행 및 관리할 있는 platform을 제공하는 service를 의미합니다. 
대표적인 상용 PaaS로는 Pivotal Cloud Foundry, Google App Engine, Oracle Cloud Platform, Heroku 등이 있습니다.

> SaaS는 cloud 환경에서 동작하는 application을 service 형태로 제공하는 것으로 software as a service의 약자입니다. 
주문형 소프트웨어(on-demand software)라고도 하며, 대표적으로 Google Gmail, Google docs, Dropbox 등이 있습니다.

> Buildpack이란 application을 CFAR로 배포(push)할 때, application에 대한 프레임워크(framework) 및 런타임(runtime) 지원을 제공합니다. CFAR은 push 되는 application을 검사하여 적합한 buildpack을 감지하고, 이 buildpack은 내려받을 종속성(dependencies)을 결정하고 바인딩(binding) 된 service와 통신하도록 application 구성을 설정합니다. buildpack에 대한 자세한 정보는 [https://docs.cloudfoundry.org/buildpacks/](https://docs.cloudfoundry.org/buildpacks/){: target="\_blank"}를 확인해 주시기 바랍니다.

> 바인딩(binding)이란 일반적으로 하나를 다른 것으로 매핑시키는 것을 의미합니다. Cloud Foundry에서 binding은 service instance를 push 된 application에 연결 또는 연관(association)시키는 것이며, 반대는 unbinding입니다.

### 1. 역사(history)
- CFAR은 VMware에 의해 개발되어 Pivotal Software로 넘어간 후, 2015년 1월에 Cloud Foundry 재단이 비영리 독립 리눅스(linux) 재단 협업 project의 하나로 설립되어, Cloud Foundry 소프트웨어(소스 코드 및 모든 관련 상표)는 open source 소프트웨어 재단의 소유가 되었습니다.
- 현재 CFAR 개발은 Cloud Foundry 재단이 관리 및 통제하고 있습니다.

### 2. 아키텍처(architecture)
![lindarex-cfar-architecture-001]

### 3. 지원되는 런타임 및 프레임워크(runtimes and frameworks)

| 언어(language) | 런타임(runtimes) | 프레임워크(frameworks) |
|:--------|:--------|:--------|
| 자바(Java) | Java 6 ~ 8	 | 스프링 프레임워크(Spring Frameworks) 3.x, 4.x |
| 루비(Ruby) | Ruby 1.8 ~ 2.2 | 레일즈(Rails), Sinatra |
| Node.js | V8 자바스크립트 엔진 (구글 크롬) | Node.js |
| 스칼라(Scala) | Scala 2.x	 | 플레이(play) 2.x, 리프트(lift) |
| 파이썬(Python) | Python 2.7.10 ~ 3.5.1 | Python |
| PHP | PHP 5.5 ~ 7.0 | PHP |
| Go | Go 1.1.1 ~ 1.4.2 | Go |

> 출처 :: [https://ko.wikipedia.org/w/index.php?title=클라우드_파운드리&action=edit&section=4](https://ko.wikipedia.org/w/index.php?title=%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C_%ED%8C%8C%EC%9A%B4%EB%93%9C%EB%A6%AC&action=edit&section=4){: target="\_blank"}

### 4. 구성요소(component)
- 각 구성요소에 대한 최신 배포(release) 버전과 저장소(repository), 개별 설치 방법은 생략합니다.

#### 4.1. Router
- 라우터(router)는 들어오는 트래픽을 클라우드 컨트롤러(Cloud Controller) 컴포넌트(component) 또는 디에고 셀(Diego Cell)에서 실행되는 호스팅 된 application으로 라우팅합니다.
- router는 주기적으로 Diego BBS(bulletin board system)를 조회하여 현재 application이 실행 중인 셀(cell)과 container를 결정하고, 이 정보로 각 cell 가상 머신(VM, virtual machine)의 IP 주소(address)와 cell container의 호스트 측 포트(port) 번호를 기반으로 새로운 라우팅 테이블을 다시 계산합니다.

> router에 대한 자세한 정보는 [https://docs.cloudfoundry.org/concepts/architecture/router.html](https://docs.cloudfoundry.org/concepts/architecture/router.html){: target="\_blank"}을 확인해 주시기 바랍니다.

#### 4.2. OAuth2 UAA server
- OAuth2 UAA(user account and authentication) server와 로그인(login) server는 함께 작동하여 ID 관리 기능을 제공합니다.
- UAA는 OAuth2 제공자(provider)로서 CFAR 사용자를 대신하여 클라이언트(client) application이 사용할 수 있는 토큰(token)을 발행합니다. 
- UAA는 login server와 연동하여 CFAR 자격 증명(credentials)으로 사용자를 인증 할 수 있으며, 이러한 credentials나 다른 credentials를 사용하여 SSO service를 제공할 수 있습니다.
- UAA에는 사용자 계정을 관리하고 OAuth2 client를 등록하기 위한 엔드 포인트(endpoints)와 다양한 관리 기능이 있습니다.
- CFAR에는 기본적으로 두 개의 UAA 인스턴스(instance)가 있는데, 하나는 BOSH Director 용이며 다른 하나는 BOSH 배포(CFAR도 BOSH 배포에 포함) 용도로 사용됩니다.
- 하나의 runtime 또는 service에 login 하면, UAA를 사용하여 인증하는 다른 runtime 및 service에는 login 되지 않고, 각 runtime 또는 service에 별도로 login 해야 합니다.

> UAA에 대한 자세한 정보는 [https://docs.cloudfoundry.org/concepts/architecture/uaa.html](https://docs.cloudfoundry.org/concepts/architecture/uaa.html){: target="\_blank"}을 확인해 주시기 바랍니다.

#### 4.3. Cloud Controller
- 클라우드 컨트롤러(CC, cloud controller)는 application push를 지시합니다. 
- application을 CFAR로 push 하기 위해 CC를 대상(target)으로 하고, CC-Bridge components를 통해 Diego Brain에 지시하여 개별 Diego Cell을 조정한 후, application을 준비(stage)하고 실행합니다.
- CC는 client가 시스템(system)에 접근할 수 있도록 REST API endpoints를 제공하고, 조직(orgs), 공간(spaces), 서비스(services), 사용자 역할(user roles) 등의 데이터베이스(database)를 유지하고 관리합니다.

> CC에 대한 자세한 정보는 [https://docs.cloudfoundry.org/concepts/architecture/cloud-controller.html](https://docs.cloudfoundry.org/concepts/architecture/cloud-controller.html){: target="\_blank"}을 확인해 주시기 바랍니다.

> Diego cell에 대한 자세한 정보는 [https://docs.cloudfoundry.org/concepts/architecture/#diego-cell](https://docs.cloudfoundry.org/concepts/architecture/#diego-cell){: target="\_blank"}을 확인해 주시기 바랍니다.

#### 4.4. Blobstore
- Blobstore는 큰 바이너리(binary) 파일(file)을 저장할 수 있는 repository입니다.
- application code packages, buildpacks, droplets를 포함하고 있으며, 내부 server 또는 외부 S3, S3 호환 endpoints로 구성할 수 있습니다.

> droplets이란 application을 CFAR로 push하고 buildpack을 사용하여 deploy 하면 생성되는 CFAR의 실행 단위입니다. 

#### 4.5. Diego Cell
- application instances와 application tasks, staging tasks는 모두 Diego Cell VM에서 Garden container로 실행됩니다.
- Diego cell rep component는 container의 lifecycle과 container에서 실행되는 프로세스(process)를 관리하고, container 상태를 Diego BBS에 보고하며, 로그(log)와 측정지표(metrics)를 Loggregator로 보냅니다.

> Garden에 대한 자세한 정보는 [https://docs.cloudfoundry.org/concepts/architecture/garden.html](https://docs.cloudfoundry.org/concepts/architecture/garden.html){: target="\_blank"}을 확인해 주시기 바랍니다.

#### 4.6. Service Brokers
- application은 database나 third-party SaaS provider와 같은 service에 의존하는데, service를 프로비저닝(provisioning)하고 application에 binding 할 경우, 해당 service의 service broker는 service instance를 제공하는 역할을 수행합니다.

> 프로비저닝(provisioning)이란 사용자의 요구에 맞게 system resource를 할당하고 배치하여 배포해 두었다가, 필요할 때 system을 즉시 사용할 수 있는 상태로 미리 준비해 두는 것을 의미합니다.

> CFAR의 service에 대한 자세한 정보는 [http://docs.cloudfoundry.org/services/index.html](http://docs.cloudfoundry.org/services/index.html){: target="\_blank"}을 확인해 주시기 바랍니다.

#### 4.7. Internal networking
- CFAR component VM은 HTTP와 HTTPS 프로토콜(protocol)을 통해 내부적으로 서로 통신하며, Diego BBS에 저장된 임시 메시지(message)와 데이터(data)를 공유합니다.
- BOSH Director는 deploy 된 모든 VM에 BOSH DNS server를 설치하고, 모든 VM은 다른 모든 VM에 대한 최신 DNS 레코드(record)를 같은 기반으로 유지하여 VM 간의 service 검색을 가능하게 합니다.
- BOSH DNS는 여러 VM을 사용할 수 있을 때, 상태가 양호한 VM을 임의로 선택하여 client 측 로드 밸런싱(load balancing)을 제공합니다.
- route-emitter component는 NATS protocol을 사용하여 최신 라우팅 테이블을 router로 전송합니다.
- Diego BBS는 cell과 application 상태, 할당되지 않은 작업, heartbeat messages 등과 같이 자주 업데이트되는 data와 일회성 data, 수명이 긴 distributed locks를 저장하며, Go MySQL 드라이버를 사용하여 MySQL에 data를 저장합니다.

> BBS(Bulletin Board System)에 대한 자세한 정보는 [https://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#bbs](https://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#bbs){: target="\_blank"}를 확인해 주시기 바랍니다.

> BOSH DNS에 대한 자세한 정보는 [https://bosh.io/docs/dns/](https://bosh.io/docs/dns/){: target="\_blank"}를 확인해 주시기 바랍니다.

#### 4.8. Loggregator
- Loggregator(log aggregator) system은 application log를 스트리밍하여 개발자에게 전송합니다.

> Loggregator에 대한 자세한 정보는 [https://docs.cloudfoundry.org/loggregator/architecture.html](https://docs.cloudfoundry.org/loggregator/architecture.html){: target="\_blank"}을 확인해 주시기 바랍니다.

#### 4.9. Metrics Collector
- Metrics Collector는 component로부터 metrics와 통계치(statistics)를 수집하고, 운영자는 이 정보를 사용하여 Cloud Foundry deployment를 모니터링할 수 있습니다.


## 마무리(CONCLUSION)
CFAR에 대한 소개와 architecture, component 등을 살펴보았습니다. <br />
CFAR을 비롯해서 Cloud, PaaS, IaaS, SaaS 등의 기술 트랜드는 선택이 아닌 필수가 되었습니다. 대기업 및 금융권 등 현업에서는 Cloud 전환 프로젝트가 빠르게 진행 중이고, 공공 기관에서도 위에 소개한 PaaS-TA를 이용한 다양한 과제가 진행되고 있습니다. CFAR에 대한 더 자세한 내용은 앞으로 다양한 포스트로 소개할 예정입니다. <br />
다음 포스트에서는 [클라우드 파운드리(Cloud Foundry) 프로젝트(projects)](https://lindarex.github.io/cfar/cloud-foundry-projects-introduction/){: target="\_blank"}를 소개하겠습니다.


## 참고(REFERENCES)
- [https://www.cloudfoundry.org/](https://www.cloudfoundry.org/){: target="\_blank"}
- [https://docs.cloudfoundry.org/concepts/architecture/](https://docs.cloudfoundry.org/concepts/architecture/){: target="\_blank"}
- [https://ko.wikipedia.org/wiki/클라우드_파운드리](https://ko.wikipedia.org/wiki/%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C_%ED%8C%8C%EC%9A%B4%EB%93%9C%EB%A6%AC){: target="\_blank"}
- [http://www.ciokorea.com/news/37345](http://www.ciokorea.com/news/37345){: target="\_blank"}
- [https://www.trustradius.com/platform-as-a-service-paas](https://www.trustradius.com/platform-as-a-service-paas){: target="\_blank"}
- [https://www.updatedreviews.in/top-10-best-paas-providers](https://www.updatedreviews.in/top-10-best-paas-providers){: target="\_blank"}
- [https://www.slant.co/topics/3478/~best-platform-as-a-service-providers-that-have-a-free-plan](https://www.slant.co/topics/3478/~best-platform-as-a-service-providers-that-have-a-free-plan){: target="\_blank"}
- [https://www.devteam.space/blog/10-top-paas-providers/](https://www.devteam.space/blog/10-top-paas-providers/){: target="\_blank"}


[lindarex-cfar-architecture-001]:/assets/images/2020-02-25-cfar-introduction/lindarex-cfar-architecture-001.png
