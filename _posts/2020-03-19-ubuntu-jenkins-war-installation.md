---
title: "우분투(Ubuntu) 환경에 WAR 파일로 젠킨스(Jenkins) 설치하기"
categories: 
  - jenkins
tags: 
  - jenkins
  - ubuntu
---


젠킨스(Jenkins)를 설치하는 방법은 다양합니다. 패키지(package)로 jenkins를 설치하는 방법은 [우분투(Ubuntu) 환경에 패키지(Package)로 젠킨스(Jenkins) 설치하기](https://lindarex.github.io/jenkins/ubuntu-jenkins-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다. <br />
이 포스트에서는 우분투(ubuntu) 환경에서 WAR 파일(file)로 jenkins를 설치하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- ubuntu 환경에 Java가 설치되어 있어야 합니다.
    + Java 8 or Java 11 버전(version)이 필요합니다.
- 방화벽 설정이 필요합니다.
    + TCP 8080 포트가 개방되어 있어야 합니다. 

> Java 설치 방법은 [우분투(Ubuntu) 환경에 OpenJDK(Java) 설치하기](https://lindarex.github.io/ubuntu/ubuntu-openjdk-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.

> jenkins version에 따른 Java version에 대한 자세한 정보는 [https://jenkins.io/doc/administration/requirements/java/](https://jenkins.io/doc/administration/requirements/java/){: target="\_blank"}를 확인해 주시기 바랍니다.

> 방화벽 설정 방법은 [우분투(Ubuntu) 환경에 방화벽(Firewalld) 설치 및 설정하기](https://lindarex.github.io/ubuntu/ubuntu-firewalld-installation/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- VMware® Workstation 15 Pro (15.5.1 build-15018445)
- Ubuntu 18.04.4 LTS (Bionic Beaver) Server (64-bit)
- jenkins 2.204.5
- OpenJDK 1.8.0_242


## 요약(SUMMARY)
1. 작업 공간(workspace) 생성
2. jenkins war file 내려받기
3. java 명령어로 jenkins 실행
4. 웹브라우저로 jenkins 접속


## 내용(CONTENTS)
### 1. workspace 생성
- 로그인한 사용자의 ubuntu 홈 디렉터리(directory) 아래에 'workspace' directory를 생성합니다.

```console
$ mkdir -p ${HOME}/workspace
```

### 2. jenkins war file 내려받기
- 위에서 생성한 workspace directory에 wget 명령어(command)로 jenkins war file을 내려받습니다.

```console
$ wget -P ${HOME}/workspace http://mirrors.jenkins.io/war-stable/latest/jenkins.war
--2020-03-19 04:25:52--  http://mirrors.jenkins.io/war-stable/latest/jenkins.war
Resolving mirrors.jenkins.io (mirrors.jenkins.io)... 52.202.51.185
Connecting to mirrors.jenkins.io (mirrors.jenkins.io)|52.202.51.185|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: http://ftp-nyc.osuosl.org/pub/jenkins/war-stable/2.204.5/jenkins.war [following]
--2020-03-19 04:25:53--  http://ftp-nyc.osuosl.org/pub/jenkins/war-stable/2.204.5/jenkins.war
Resolving ftp-nyc.osuosl.org (ftp-nyc.osuosl.org)... 64.50.233.100, 2600:3404:200:237::2
Connecting to ftp-nyc.osuosl.org (ftp-nyc.osuosl.org)|64.50.233.100|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 63466030 (61M) [application/x-java-archive]
Saving to: ‘/home/rex/workspace/jenkins.war’

jenkins.war                                              100%[=================================================================================================================================>]  60.53M  31.0MB/s    in 2.0s

2020-03-19 04:25:55 (31.0 MB/s) - ‘/home/rex/workspace/jenkins.war’ saved [63466030/63466030]
```

- jenkins.war file을 확인합니다.

```console
$ ll ${HOME}/workspace
total 61988
drwxrwxr-x 2 rex rex     4096 Mar 19 04:25 ./
drwxr-xr-x 5 rex rex     4096 Mar 19 04:25 ../
-rw-rw-r-- 1 rex rex 63466030 Mar  8 06:20 jenkins.war
```

### 3. java command로 jenkins 실행

```console
$ java -jar ${HOME}/workspace/jenkins.war --httpPort=8080
Running from: /home/rex/workspace/jenkins.war
webroot: $user.home/.jenkins
2020-03-19 05:28:15.555+0000 [id=1]	INFO	org.eclipse.jetty.util.log.Log#initialized: Logging initialized @516ms to org.eclipse.jetty.util.log.JavaUtilLog
2020-03-19 05:28:15.689+0000 [id=1]	INFO	winstone.Logger#logInternal: Beginning extraction from war file
2020-03-19 05:28:16.774+0000 [id=1]	WARNING	o.e.j.s.handler.ContextHandler#setContextPath: Empty contextPath
2020-03-19 05:28:16.841+0000 [id=1]	INFO	org.eclipse.jetty.server.Server#doStart: jetty-9.4.z-SNAPSHOT; built: 2019-05-02T00:04:53.875Z; git: e1bc35120a6617ee3df052294e433f3a25ce7097; jvm 1.8.0_242-8u242-b08-0ubuntu3~18.04-b08
2020-03-19 05:28:17.214+0000 [id=1]	INFO	o.e.j.w.StandardDescriptorProcessor#visitServlet: NO JSP Support for /, did not find org.eclipse.jetty.jsp.JettyJspServlet
2020-03-19 05:28:17.291+0000 [id=1]	INFO	o.e.j.s.s.DefaultSessionIdManager#doStart: DefaultSessionIdManager workerName=node0
2020-03-19 05:28:17.292+0000 [id=1]	INFO	o.e.j.s.s.DefaultSessionIdManager#doStart: No SessionScavenger set, using defaults
2020-03-19 05:28:17.295+0000 [id=1]	INFO	o.e.j.server.session.HouseKeeper#startScavenging: node0 Scavenging every 600000ms
2020-03-19 05:28:17.757+0000 [id=1]	INFO	hudson.WebAppMain#contextInitialized: Jenkins home directory: /home/rex/.jenkins found at: $user.home/.jenkins
2020-03-19 05:28:17.930+0000 [id=1]	INFO	o.e.j.s.handler.ContextHandler#doStart: Started w.@4a9e6faf{Jenkins v2.204.5,/,file:///home/rex/.jenkins/war/,AVAILABLE}{/home/rex/.jenkins/war}
2020-03-19 05:28:17.974+0000 [id=1]	INFO	o.e.j.server.AbstractConnector#doStart: Started ServerConnector@7a5ceedd{HTTP/1.1,[http/1.1]}{0.0.0.0:8080}
2020-03-19 05:28:17.977+0000 [id=1]	INFO	org.eclipse.jetty.server.Server#doStart: Started @2938ms
2020-03-19 05:28:17.981+0000 [id=20]	INFO	winstone.Logger#logInternal: Winstone Servlet Engine v4.0 running: controlPort=disabled
2020-03-19 05:28:19.005+0000 [id=25]	INFO	jenkins.InitReactorRunner$1#onAttained: Started initialization
2020-03-19 05:28:19.050+0000 [id=25]	INFO	jenkins.InitReactorRunner$1#onAttained: Listed all plugins
2020-03-19 05:28:20.675+0000 [id=25]	INFO	jenkins.InitReactorRunner$1#onAttained: Prepared all plugins
2020-03-19 05:28:20.689+0000 [id=25]	INFO	jenkins.InitReactorRunner$1#onAttained: Started all plugins
2020-03-19 05:28:20.716+0000 [id=26]	INFO	jenkins.InitReactorRunner$1#onAttained: Augmented all extensions
2020-03-19 05:28:21.991+0000 [id=26]	INFO	jenkins.InitReactorRunner$1#onAttained: Loaded all jobs
2020-03-19 05:28:22.027+0000 [id=39]	INFO	hudson.model.AsyncPeriodicWork#lambda$doRun$0: Started Download metadata
2020-03-19 05:28:22.050+0000 [id=39]	INFO	hudson.util.Retrier#start: Attempt #1 to do the action check updates server
2020-03-19 05:28:23.246+0000 [id=26]	INFO	o.s.c.s.AbstractApplicationContext#prepareRefresh: Refreshing org.springframework.web.context.support.StaticWebApplicationContext@2f650e69: display name [Root WebApplicationContext]; startup date [Thu Mar 19 04:28:23 KST 2020]; root of context hierarchy
2020-03-19 05:28:23.250+0000 [id=26]	INFO	o.s.c.s.AbstractApplicationContext#obtainFreshBeanFactory: Bean factory for application context [org.springframework.web.context.support.StaticWebApplicationContext@2f650e69]: org.springframework.beans.factory.support.DefaultListableBeanFactory@252ba0b0
2020-03-19 05:28:23.274+0000 [id=26]	INFO	o.s.b.f.s.DefaultListableBeanFactory#preInstantiateSingletons: Pre-instantiating singletons in org.springframework.beans.factory.support.DefaultListableBeanFactory@252ba0b0: defining beans [authenticationManager]; root of factory hierarchy
2020-03-19 05:28:23.532+0000 [id=26]	INFO	o.s.c.s.AbstractApplicationContext#prepareRefresh: Refreshing org.springframework.web.context.support.StaticWebApplicationContext@17aa246c: display name [Root WebApplicationContext]; startup date [Thu Mar 19 04:28:23 KST 2020]; root of context hierarchy
2020-03-19 05:28:23.533+0000 [id=26]	INFO	o.s.c.s.AbstractApplicationContext#obtainFreshBeanFactory: Bean factory for application context [org.springframework.web.context.support.StaticWebApplicationContext@17aa246c]: org.springframework.beans.factory.support.DefaultListableBeanFactory@45e8b062
2020-03-19 05:28:23.534+0000 [id=26]	INFO	o.s.b.f.s.DefaultListableBeanFactory#preInstantiateSingletons: Pre-instantiating singletons in org.springframework.beans.factory.support.DefaultListableBeanFactory@45e8b062: defining beans [filter,legacy]; root of factory hierarchy
2020-03-19 05:28:23.817+0000 [id=26]	INFO	jenkins.install.SetupWizard#init:

*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

cqqb445c1bad4f219e3737661dd2740d

This may also be found at: /home/rex/.jenkins/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************

2020-03-19 05:28:33.139+0000 [id=39]	INFO	hudson.model.UpdateSite#updateData: Obtained the latest update center data file for UpdateSource default
2020-03-19 05:28:33.770+0000 [id=26]	INFO	hudson.model.UpdateSite#updateData: Obtained the latest update center data file for UpdateSource default
2020-03-19 05:28:34.144+0000 [id=26]	INFO	jenkins.InitReactorRunner$1#onAttained: Completed initialization
2020-03-19 05:28:34.162+0000 [id=19]	INFO	hudson.WebAppMain$3#run: Jenkins is fully up and running
2020-03-19 05:28:34.796+0000 [id=39]	INFO	h.m.DownloadService$Downloadable#load: Obtained the updated data file for hudson.tasks.Maven.MavenInstaller
2020-03-19 05:28:34.797+0000 [id=39]	INFO	hudson.util.Retrier#start: Performed the action check updates server successfully at the attempt #1
2020-03-19 05:28:34.800+0000 [id=39]	INFO	hudson.model.AsyncPeriodicWork#lambda$doRun$0: Finished Download metadata. 12,771 ms
```

> 위 콘솔 로그에서 비밀번호('cqqb445c1bad4f219e3737661dd2740d')는 jenkins 초기 설정할 때 필요합니다.

### 4. 웹브라우저로 jenkins 접속
- http://[MY-IP]:8080


## 마무리(CONCLUSION)
ubuntu 환경에 war file로 jenkins 설치를 완료했습니다. <br />
jenkins 초기 설정은 [젠킨스(Jenkins) 초기 설정하기](https://lindarex.github.io/jenkins/jenkins-initial-setting/){: target="\_blank"} 포스트를 참고하시기 바랍니다. <br />
다음 포스트에서는 유용한 jenkins 플러그인(plugin)을 소개하겠습니다.


## 참고(REFERENCES)
- [https://jenkins.io/](https://jenkins.io/){: target="\_blank"}
- [https://jenkins.io/doc/pipeline/tour/getting-started/](https://jenkins.io/doc/pipeline/tour/getting-started/){: target="\_blank"}
