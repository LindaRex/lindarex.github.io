---
title: "Java StringUtils(org.apache.commons.lang3) 소개"
categories: 
  - concepts
tags: 
  - stringutils
  - apache
  - commons
  - "apache commons"
  - "open source software"
  - "open source"
  - oss
  - java
---


Java StringUtils는 Apache Commons Lang 패키지(package)에 포함되어 있으며, 이를 통해 java.lang API를 위한 유틸리티(utility)를 사용할 수 있습니다.
<br /><br />
이 포스트에서는 Java StringUtils를 소개합니다.


## 요약(SUMMARY)
1. StringUtils 소개
2. StringUtils 예제(example)


## 내용(CONTENTS)
### 1. StringUtils 소개
- StringUtils는 Java의 String class가 제공하는 문자열 관련 기능을 강화한 클래스입니다.
    + 표준 Java 라이브러리(library)는 핵심(core) 클래스(class) 제어를 위한 메소드(method)가 부족한데, 이를 Apache Commons Lang으로 보완할 수 있습니다.

- StringUtils는 Apache 소프트웨어 재단에서 제공하며, 최신 안정화(stable) 버전은 2021년 3월 1일 기준 3.12.0(Java 1.8)입니다.
    + Apache Commons Lang는 java.lang class와 java.util.Date class를 위한 표준으로 간주하는 Java utility class package입니다.
    + Apache Commons Lang 3.0 및 이후 버전은 이전 버전(org.apache.commons.lang)과 다른 패키지(org.apache.commons.lang3)를 사용하므로 동시에 사용할 수 있습니다.

- StringUtils는 대부분의 문자열 처리를 수행할 수 있으며, 파라미터(parameter)값으로 null 입력 시에도 NullPointException을 발생시키지 않습니다.

### 2. StringUtils example

- test code

```java
package com.lindarex.test.user;

import org.apache.commons.lang3.StringUtils;

public class StringUtilsTest {

    public static void main(String[] args) {

        String testString;
        String testString2;
        Boolean testBoolean;
        

        testString = "hello java.";

        // testString이 java를 포함하고 있으면 true를 반환합니다.
        testBoolean = StringUtils.contains(testString, "java");
        System.out.println("contains : " + testBoolean);

        
        // testString이 null이면 "", 아니면 testString을 반환합니다.
        testString2 =  StringUtils.defaultString(testString);
        System.out.println("defaultString : " + testString2);


        testString = "h e l l o j a v a .";

        // 문자열 중 공백 문자가 있으면 모두 제거한다.
        testString2 = StringUtils.deleteWhitespace(testString);
        System.out.println("deleteWhitespace : " + testString2);
        

        testString = "lindarex";
        testString2 = "lindarex";

        // testString과 testString2의 동일 유무를 반환합니다.
        testBoolean = StringUtils.equals(testString, testString2);
        System.out.println("equals : " + testBoolean);


        testString = "JAVA";
        testString2 = "java";

        // 대소문자를 무시하고 testString과 testString2를 비교합니다.
        testBoolean = StringUtils.equalsIgnoreCase(testString, testString2);
        System.out.println("equalsIgnoreCase : " + testBoolean);
        

        testString = "lindarex lindarex";

        // testString에서 첫 번째 "rex"의 index를 반환합니다. (index는 0부터 시작)
        int i = StringUtils.indexOf(testString, "rex");
        System.out.println("indexOf : " + i);

        // testString에서 마지막 "linda"의 index를 반환합니다.
        i = StringUtils.lastIndexOf(testString, "linda");
        System.out.println("lastIndexOf : " + i);
        
        // testString이 null이거나 길이가 0이면 true를 반환합니다.
        testBoolean = StringUtils.isEmpty(testString);
        System.out.println("isEmpty : " + testBoolean);

        // testString이 null이 아니거나 길이가 0이 아니면 true를 반환합니다.
        testBoolean = StringUtils.isNotEmpty(testString);
        System.out.println("isNotEmpty : " + testBoolean);


        String[] testStringArray1 = {"java", "javascript", "jQuery", "json"};
        testString = " | ";

        // array에서 문자열을 읽어와 " | "를 구분자로 연결합니다.
        testString2 = StringUtils.join(testStringArray1, testString);
        System.out.println("join : " + testString2);


        testString = "LINDAREX";

        // testString을 소문자로 변환합니다.
        testString2 = StringUtils.lowerCase(testString);
        System.out.println("lowerCase : " + testString2);
        

        testString = "lindarex";

        //testString을 대문자로 변환합니다.
        testString2 = StringUtils.upperCase(testString);
        System.out.println("upperCase : " + testString2);


        testString = "HELLO java";

        // 대문자는 소문자로, 소문자는 대문자로 변환합니다.
        testString2 = StringUtils.swapCase(testString);
        System.out.println("swapCase : " + testString2);

        //문자열의 앞뒤 순서를 바꿉니다.
        testString2 = StringUtils.reverse(testString);
        System.out.println("reverse : " + testString2);


        testString = "c++, java, c#, javascript, jQuery";

        // ','를 구분자로 사용하여 분리합니다.
        String[] testStringArray2 = StringUtils.split(testString, ',');

        for(int j = 0; j < testStringArray2.length; j++) {
            System.out.println("split testStringArray2[" + j + "] : " + testStringArray2[j]);
        }


        testString = "    java    ";

        // 문자열 좌우에 있는 공백 문자를 제거합니다.(=trim())
        testString2 = StringUtils.strip(testString);
        System.out.println("strip : " + testString2);

        // 문자열 좌우 공백 문자를 제거합니다.
        testString2 = StringUtils.trim(testString);
        System.out.println("trim : " + testString2);

    }
}
```

- test result

```java
contains : true

defaultString : hello java.

deleteWhitespace : hellojava.

equals : true

equalsIgnoreCase : true

indexOf : 5

lastIndexOf : 9

isEmpty : false

isNotEmpty : true

join : java | javascript | jQuery | json

lowerCase : lindarex

upperCase : LINDAREX

swapCase : hello JAVA

reverse : avaj OLLEH

split testStringArray2[0] : c++

split testStringArray2[1] :  java

split testStringArray2[2] :  c#

split testStringArray2[3] :  javascript

split testStringArray2[4] :  jQuery

strip : java

trim : java
```


## 마무리(CONCLUSION)
StringUtils과 StringUtils example을 간단히 소개했습니다.
<br />
더 자세한 내용은 아래 참고 페이지를 확인해 주시기 바랍니다.


## 참고(REFERENCES)
- [https://commons.apache.org/proper/commons-lang/index.html](https://commons.apache.org/proper/commons-lang/index.html){: target="\_blank"}
