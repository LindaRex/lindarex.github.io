---
title: "Oracle JDK와 OpenJDK의 차이점"
categories: 
  - concepts
tags: 
  - oraclejdk
  - openjdk
  - java
---


Java 애플리케이션 구축을 위해서는 JDK(Java Development Kit)가 필수입니다. <br />
이 포스트에서는 Oracle JDK와 OpenJDK의 차이점을 간단히 소개합니다.


## 내용(CONTENTS)
### Oracle JDK와 OpenJDK의 차이점
- Oracle JDK는 상용(유료)이지만, OpenJDK는 오픈소스기반(무료)입니다.
- Oracle JDK의 라이선스는 Oracle BCL(Binary Code License) Agreement이지만, OpenJDK의 라이선스는 Oracle GPL v2입니다.
- Oracle JDK는 LTS(장기 지원) 업데이트 지원을 받을 수 있지만, OpenJDK는 LTS 없이 6개월마다 새로운 버전이 배포됩니다.
- Oracle JDK는 Oracle이 인수한 Sun Microsystems 플러그인을 제공하지만, OpenJDK는 제공하지 않습니다.
- Oracle JDK는 OpenJDK 보다 CPU 사용량과 메모리 사용량이 적고, 응답시간이 높습니다. 

> Oracle JDK와 OpenJDK의 벤치마킹 결과는 [Comparing JDK 8 performance](https://technology.amis.nl/2018/11/23/comparing-jvm-performance-zulu-openjdk-openjdk-oracle-jdk-graalvm-ce/){: target="\_blank"} 페이지를 참고하시기 바랍니다.


## 마무리(CONCLUSION)
Oracle은 2019년 1월부터 Java를 상용으로 사용하거나 지속적인 업데이트를 받으려는 기업 사용자에게 유료로 제공하고 있으며, 
현재 2020년 2월 기준으로 Oracle JDK 8은 개인 또는 개발 용도의 사용은 무료지만, 2021년부터는 유료화로 전환될 것으로 알려져 있습니다. <br />
JDK 유료화에 따라 개인 사용자를 비롯해 기업 사용자는 OpenJDK를 적용하기 위해 호환성, 안정성, 성능 등의 검증이 필요합니다. 또한, 배포 버전에 따른 생산성 확인과 보안 이슈 등을 충분히 인지하고 테스트할 필요가 있습니다. <br />
더 자세한 내용은 아래 참고 페이지를 확인해 주시기 바랍니다.


## 참고(REFERENCES)
- [https://engineering.linecorp.com/ko/blog/line-open-jdk/](https://engineering.linecorp.com/ko/blog/line-open-jdk/){: target="\_blank"}
- [https://c10106.tistory.com/4075](https://c10106.tistory.com/4075){: target="\_blank"}
- [https://jsonobject.tistory.com/395](https://jsonobject.tistory.com/395){: target="\_blank"}
