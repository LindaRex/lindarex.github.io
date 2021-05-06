---
title: "JAVA HttpSession(javax.Servlet.Http) 소개"
categories: 
  - concepts
tags: 
  - httpsession
  - session
  - "open source software"
  - "open source"
  - oss
  - java
---


HttpSession은 Java의 public interface이며, 이를 사용하여 세션(session)을 제어할 수 있습니다.
<br /><br />
session은 쿠키(cookie)의 트래픽(traffic) 이슈(issue)와 cookie 변경으로 인한 보안 issue를 해결하기 위해 등장했습니다.
<br /><br />
이 포스트에서는 HttpSession(session)을 소개합니다.


## 요약(SUMMARY)
1. session 소개
2. session 동작 방식(mechanics)
3. HttpSession 소개


## 내용(CONTENTS)
### 1. session 소개

> 이 포스트에서 session은 http session을 의미합니다.

- session은 사전적 의미로 서버(server)와 클라이언트(client) 간에 반영구적으로 상호 작용하는 정보 교환입니다.
- session은 server로 요청(request) 하는 client를 구별하기 위해 server에 저장되는 정보입니다.
    + session은 client에 저장되는 쿠키(cookie)와 다르게 server에 저장되므로 관리가 용이하고 효율적이며 보안에 강합니다.

- server는 client request에 session-id를 생성하여 server와 client 브라우저(browser) 메모리(memory)에 cookie로 저장합니다.
    + 위 cookie는 일반적인 cookie가 아닌 session cookie이며, 인 메모리(in-memory) cookie 또는 임시(transient) cookie, 반영구(non-persistent) cookie로 불립니다.
    + session cookie는 server가 종료되거나, 유효기간이 만료하거나, client browser를 종료하면 삭제됩니다.

    > session cookie에 대한 자세한 정보는 [https://en.wikipedia.org/wiki/HTTP_cookie#Session_cookie](https://en.wikipedia.org/wiki/HTTP_cookie#Session_cookie){: target="\_blank"} 페이지를 참고하시기 바랍니다.

- session의 단점은 server resource를 사용하므로 server에 부담을 줄 수 있으며, 로드 밸런싱(load balancing) 시스템(system)에서 session 처리가 쉽지 않다는 것입니다.

### 2. session mechanics

1. client가 server에 리소스(resource)를 request 합니다.

2. server는 client가 request 한 request-header의 session cookie를 통해 session-id를 확인합니다.

3. session-id가 존재하면, server는 session-id가 유효한지 확인 후 client의 request를 처리하고 응답(response) 합니다.

4. session-id가 존재하지 않으면, server는 set-cookie를 통해 session-id를 생성한 후 response-header에 추가하여 client에 response 합니다.

5. 위 4번 항목 처리 후 client는 server에 request 시, server로부터 response 한 session-id를 request-header에 추가하여 request 합니다.

- session 종료 시기
    1. 타임아웃
    2. session 객체(object)의 invalidate() 호출
    3. 애플리케이션 다운

- session timeout
    + DD(Deployment Descriptor :: web.xml)에서 설정하며, 단위는 분입니다.
    + 아래 설정은 모든 session object에 setMaxInactiveInterval()를 호출하는 것과 같습니다.  

    ```
    <wep-app....>
      <Servlet>
       ...
      </Servlet>
    
      <Session-config>
        <Session-timeout>15</Session-timeout> 
      </Session-config>
    </wep-app>
    ```

### 3. HttpSession 소개

> 이 포스트에서 HttpSession은 javax.Servlet.Http 패키지(package)의 인터페이스(interface)인 HttpSession을 의미합니다.

- HttpSession은 client request에서 client를 식별하고, 해당 client 정보를 저장하는 방법을 제공하는 JAVA의 public interface입니다.

- HttpSession interface를 사용하는 서블릿 컨테이너(servlet container)는 server와 client 간의 session을 제어합니다.
    + servlet container는 웹(web) container라고도 불리며, Tomcat, JBoss(현재 WildFly), Jetty 등이 대표적입니다.
    + servlet container와 servlet은 다릅니다.
        - servlet container는 server에서 servlet 생명 주기(life cycle) 관리, request에 따른 스레드(thread) 생성, 동적(dynamic) resource(JSP, servlet 등) 생성 등 servlet과 상호 작용하는 web server의 일부입니다.
        - servlet은 javax.servlet package에 정의된 interface이며, JVM(java virtual machine) 내에서 실행되는 web 애플리케이션(application)의 작은 조각입니다.

    > servlet container에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/웹_컨테이너](https://ko.wikipedia.org/wiki/%EC%9B%B9_%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88){: target="\_blank"} 페이지를 참고하시기 바랍니다.

    > servlet에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/자바_서블릿](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%B0%94_%EC%84%9C%EB%B8%94%EB%A6%BF){: target="\_blank"} 페이지를 참고하시기 바랍니다.

    > JVM에 대한 자세한 정보는 [https://ko.wikipedia.org/wiki/자바_가상_머신](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%B0%94_%EA%B0%80%EC%83%81_%EB%A8%B8%EC%8B%A0){: target="\_blank"} 페이지를 참고하시기 바랍니다.

- HttpSession object 생성
    + {HttpServletRequestObject}.getSession() : // 기존 session이 있으면 기존 session object를, 없으면 새로 생성 후 반환
    + {HttpServletRequestObject}.getSession(false) // 기존 session이 있으면 기존 session object를, 없으면 null 반환

- HttpSession Method
    + setAttribute(String, Object)
    + getAttribute(): Object
    + getCreationTime(): long
    + getLastAccessedTime(): long
    + setMaxInactiveInterval(int second) // client가 설정 시간 동안 request가 없으면 session 만료
    + getMaxInactiveInterval(): int
    + invalidate() : // session 종료(session에 속한 속성들도 함께 제거)
    + getId() : String // jSessionId 반환


## 마무리(CONCLUSION)
session과 HttpSession에 대해 간단히 소개했습니다.
<br />
더 자세한 내용은 아래 참고 페이지를 확인해 주시기 바랍니다.


## 참고(REFERENCES)
- [https://javaee.github.io/javaee-spec/javadocs/javax/servlet/http/HttpSession.html](https://javaee.github.io/javaee-spec/javadocs/javax/servlet/http/HttpSession.html){: target="\_blank"}
