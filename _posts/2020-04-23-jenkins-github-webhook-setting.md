---
title: "GitHub Webhook으로 젠킨스(Jenkins) Job을 실행(자동화)하는 방법"
categories: 
  - jenkins
tags: 
  - jenkins
  - github
  - webhook
  - ci
  - "continuous integration"
  - "open source software"
  - "open source"
  - oss
---


이 포스트에서는 GitHub Webhook으로 젠킨스(Jenkins) Job을 실행(자동화)하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- 외부 통신이 가능한 jenkins가 설치되어 있어야 합니다.
- github 계정이 필요합니다.

> jenkins 설치 방법은 [우분투(Ubuntu) 환경에 패키지로 젠킨스(Jenkins) 설치하기](https://lindarex.github.io/jenkins/jenkins-initial-setting/){: target="\_blank"} 포스트를 참고하시기 바랍니다.


## 테스트 환경(TEST ENVIRONMENT)
- jenkins 2.222.3
- Chrome v81.0.4044.129(공식 빌드) (64비트)
- Firefox Browser DEVELOPER v76.0b8 (64-비트)


## 요약(SUMMARY)
1. github personal access token 생성
2. (선택사항) github repository 생성
3. jenkins 설정
4. github webhook 설정
5. 테스트


## 내용(CONTENTS)
### 1. github personal access token 생성

- github에 접속 후 'Settings' 페이지 왼쪽 아래의 'Developer settings' 메뉴를 선택합니다.

![lindarex-jenkins-github-webhook-setting-001]

- 왼쪽 아래의 'Personal access tokens' 메뉴를 선택합니다.

![lindarex-jenkins-github-webhook-setting-002]

- 오른쪽 위의 'Generate new token' 버튼을 선택합니다.

![lindarex-jenkins-github-webhook-setting-003]

- github 계정의 비밀번호를 입력합니다.

![lindarex-jenkins-github-webhook-setting-005]

- 'New personal access token' 페이지에서 아래와 같이 설정합니다.
    + Note :: lindarex-github-access-token
    + repo :: 체크
    + admin:repo_hook :: 체크

> Note는 임의로 입력합니다.

![lindarex-jenkins-github-webhook-setting-007]

- 아래의 'Generate token' 버튼을 선택하여 github personal access token을 생성합니다.

![lindarex-jenkins-github-webhook-setting-008]

- 생성한 github personal access token을 확인하고, token 값을 보관합니다.

![lindarex-jenkins-github-webhook-setting-009]

### 2. (선택사항) github repository 생성

- 'Repositories' 페이지로 이동하여 오른쪽 위의 'New' 버튼을 선택합니다.

![lindarex-jenkins-github-webhook-setting-009A]

- repository 생성 페이지에서 아래와 같이 설정하고, 아래의 'Create repository' 버튼을 선택하여 repository를 생성합니다.
    + Repository name :: lindarex-jenkins-webhook-test
    + Initialize this repository with a README :: 체크

> Repository name은 임의로 입력합니다.

> README 생성 설정은 테스트의 편의를 위함이니 체크하지 않아도 무관합니다.

![lindarex-jenkins-github-webhook-setting-010]

- 생성한 repository를 확인합니다.

![lindarex-jenkins-github-webhook-setting-011]

### 3. jenkins 설정
#### 3.1. (선택사항) jenkins 플러그인 설치

> github webhook 연동을 위해 'Github Integeration Plugin'이 필요합니다.

- jenkins에 로그인하여 왼쪽의 'Jenkins 관리' 메뉴를 선택합니다.

![lindarex-jenkins-github-webhook-setting-012]

- '플러그인 관리' 메뉴를 선택합니다.

![lindarex-jenkins-github-webhook-setting-013]

- '설치 가능' 탭을 선택합니다.

![lindarex-jenkins-github-webhook-setting-014]

- 'Github Integeration Plugin'을 체크하고, 아래의 '지금 다운로드하고 재시작 후 설치하기'를 선택합니다.

![lindarex-jenkins-github-webhook-setting-017]

- '플러그인 설치/업그레이드 중' 페이지로 이동된 후, 설치 진행을 확인합니다.

![lindarex-jenkins-github-webhook-setting-018]

- 아래와 같이 내려받기를 완료하면, 아래의 '설치가 끝나고 실행중인 작업이 없으면 Jenkins 재시작'을 체크합니다.

![lindarex-jenkins-github-webhook-setting-019]

- jenkins가 재시작됩니다.

![lindarex-jenkins-github-webhook-setting-020]

#### 3.2. jenkins credential 설정
##### 3.2.1. github personal access token을 위한 jenkins credential 생성

- jenkins에 로그인하여 왼쪽의 'Credentials' 메뉴를 선택합니다.

![lindarex-jenkins-github-webhook-setting-012]

- 왼쪽의 'System' 메뉴를 선택합니다.

![lindarex-jenkins-github-webhook-setting-021]

- 'Global credentials (unrestricted)'를 선택하여 'Add Credentials' 버튼을 선택합니다.

![lindarex-jenkins-github-webhook-setting-023]

- credential 생성 페이지로 이동합니다.

![lindarex-jenkins-github-webhook-setting-024]

- 아래와 같이 설정하고, 아래의 'OK' 버튼을 선택하여 저장합니다.
    + Kind :: Secret text
    + Scope :: Global (Jenkins, nodes, items, all child items, etc)
    + Secret :: {GITHUB PERSONAL ACCESS TOKEN}
    + ID :: lindarex-github-access-token
    + Description :: lindarex github access token

> {GITHUB PERSONAL ACCESS TOKEN} 값은 위에서 생성한 후 복사한 lindarex-github-access-token 값입니다.

> ID와 Description은 임의로 입력합니다.

![lindarex-jenkins-github-webhook-setting-025]

- 생성한 jenkins credential을 확인합니다.

![lindarex-jenkins-github-webhook-setting-026]

##### 3.2.2. github account를 위한 jenkins credential 생성

- '3.2.1.'을 참고하여 Jenkins > Credentials > System > Global credentials (unrestricted) > Add Credentials 순서로 jenkins credential을 추가 생성합니다.
- 아래와 같이 설정하고, 아래의 'OK' 버튼을 선택하여 저장합니다.
    + Kind :: Username with password
    + Scope :: Global (Jenkins, nodes, items, all child items, etc)
    + Username :: {GITHUB ACCOUNT NAME}
    + Password :: {GITHUB ACCOUNT PASSWORD}
    + ID :: lindarex-github-account
    + Description :: lindarex github account

> Username과 Password는 github 계정 정보입니다.

> Username은 github 로그인 시 입력하는 이메일 주소가 아닌, github profile의 'Name' 값입니다.

> ID와 Description은 임의로 입력합니다.

![lindarex-jenkins-github-webhook-setting-027]

- 생성한 jenkins credential을 확인합니다.

![lindarex-jenkins-github-webhook-setting-028]

#### 3.3. github server 설정

- jenkins 첫 페이지에서 왼쪽의 'Jenkins 관리' 메뉴를 선택합니다.

![lindarex-jenkins-github-webhook-setting-012]

- '시스템 설정' 메뉴를 선택합니다.

![lindarex-jenkins-github-webhook-setting-013]

- GitHub 설정 영역의 'Add GitHub Server'를 선택합니다.

![lindarex-jenkins-github-webhook-setting-031]

- 아래와 같이 설정하고, 아래의 '저장' 버튼을 선택하여 저장합니다.
    + Name :: lindarex-github-server
    + API URL :: https://api.github.com
    + Credentials :: lindarex-github-access-token
    + Manage hooks :: 체크

> Name은 임의로 입력합니다.

> Credentials는 github personal access token을 위해 생성한 jenkins credential 값입니다.

![lindarex-jenkins-github-webhook-setting-032]

#### 3.4. jenkins job 설정

- jenkins 첫 페이지에서 왼쪽의 '새로운 Item' 메뉴를 선택합니다.

![lindarex-jenkins-github-webhook-setting-012]

- 아래와 같이 설정하고, 아래의 'OK' 버튼을 선택하여 새로운 item(job)을 생성합니다.
    + Item name :: github-webhook
    + Freestyle project 선택

> Item name은 임의로 입력합니다.

![lindarex-jenkins-github-webhook-setting-033]

- '소스 코드 관리'와 '빌드 유발' 영역을 아래와 같이 설정하고, 아래의 '저장' 버튼을 선택하여 저장합니다.
    + '소스 코드 관리' > 'Git' :: 체크
        - Repositories > Repository URL	:: https://github.com/lindarex/lindarex-jenkins-webhook-test
        - Repositories > Credentials :: lindarex-github-account
        - Branches to build > Branch Specifier (blank for 'any') :: '\*/master'
        - Repository browser :: '(자동)'
    + '빌드 유발' > 'GitHub hook trigger for GITScm polling' :: 체크

> Repositories > Repository URL은 위에서 생성한 github repository 주소입니다.

> Repositories > Credentials는 github account를 위해 생성한 jenkins credential 값입니다.

![lindarex-jenkins-github-webhook-setting-035]

### 4. github webhook 설정

- 위에서 생성한 github repository로 이동한 후, 오른쪽 위의 'Settings' 메뉴를 선택합니다.

![lindarex-jenkins-github-webhook-setting-036]

- 왼쪽의 'Webhooks' 메뉴를 선택합니다.

![lindarex-jenkins-github-webhook-setting-037]

- 오른쪽 위의 'Add webhook' 버튼을 선택합니다.

![lindarex-jenkins-github-webhook-setting-038]

- 아래와 같이 설정하고, 아래의 'Add webhook' 버튼을 선택하여 저장합니다.
    + Payload URL :: [JENKINS_URL]/github-webhook/
    + Content type :: application/json

> Payload URL 입력 시, '[JENKINS_URL]/github-webhook/'과 같이 마지막 '/'를 꼭 입력하시기 바랍니다.

![lindarex-jenkins-github-webhook-setting-039]

- 저장 후 정상적으로 연동되면, 목록의 Payload URL 앞에 녹색 아이콘이 표시됩니다.

> Payload URL 앞에 녹색 아이콘이 아니라면 오류가 발생한 것이며, Payload URL 오타 또는 github과 jenkins 간의 통신 오류가 원인입니다.

![lindarex-jenkins-github-webhook-setting-040]

### 5. 테스트

- 위에서 생성한 github repository로 이동합니다.

![lindarex-jenkins-github-webhook-setting-036]

- 테스트를 위해 github repository의 README.md 파일을 수정합니다.

![lindarex-jenkins-github-webhook-setting-041]

- 'Commit changes' 버튼을 선택하여 수정 사항을 적용(PUSH)합니다.

![lindarex-jenkins-github-webhook-setting-042]

- jenkins에 접속하면, 왼쪽 아래에 '빌드 실행 상태'에 github webhook 설정을 한 job이 실행하는 것을 확인할 수 있습니다.

![lindarex-jenkins-github-webhook-setting-043]

- 빌드가 완료된 후 해당 job을 선택합니다.

![lindarex-jenkins-github-webhook-setting-044]

- 왼쪽 아래의 'Build History' 영역에서 완료한 빌드 번호를 선택한 후, 'Console Output'을 선택합니다.

![lindarex-jenkins-github-webhook-setting-045]

- README.md 파일을 수정한 내역과 함께 정상적으로 github webhook이 연동된 것을 확인할 수 있습니다.

![lindarex-jenkins-github-webhook-setting-046]


## 마무리(CONCLUSION)
github webhook으로 jenkins job을 실행(자동화)하는 방법을 소개했습니다.
<br /><br />
현업에서는 github에서 제공하는 webhook을 사용해서 'Pull Request' 발생에 따라 빌드뿐만 아니라 테스트, 코드 분석, 리뷰, 알람 등을 적용하여 개발 프로세스를 최대한 자동화합니다.
<br /><br />
하지만 어디까지나 개발 서버와 연동하는 수준이며, 운영 서버는 보안 및 승인 등의 이유로 github webhook을 연동하는 경우는 거의 없습니다.
<br /><br />
그리고 github webhook을 적용하기 위해서는 github와 jenkins 간의 통신이 가능해야 하므로 jenkins는 외부망에 위치해야 합니다. 그래서 내부망 또는 폐쇄망에서는 github webhook을 사용할 수 없는 단점이 있습니다.
<br /><br />
하지만 상당수의 많은 프로젝트에서, 특히 DevOps를 지향하는 사이트에서는 자동화된 CI/CD(continuous integration/continuous delivery 또는 continuous deployment) 구축을 위해 github webhook 연동을 필수로 설정합니다.


[lindarex-jenkins-github-webhook-setting-001]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-001.png
[lindarex-jenkins-github-webhook-setting-002]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-002.png
[lindarex-jenkins-github-webhook-setting-003]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-003.png
[lindarex-jenkins-github-webhook-setting-005]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-005.png
[lindarex-jenkins-github-webhook-setting-007]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-007.png
[lindarex-jenkins-github-webhook-setting-008]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-008.png
[lindarex-jenkins-github-webhook-setting-009]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-009.png
[lindarex-jenkins-github-webhook-setting-009A]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-009A.png
[lindarex-jenkins-github-webhook-setting-010]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-010.png
[lindarex-jenkins-github-webhook-setting-011]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-011.png
[lindarex-jenkins-github-webhook-setting-012]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-012.png
[lindarex-jenkins-github-webhook-setting-013]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-013.png
[lindarex-jenkins-github-webhook-setting-014]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-014.png
[lindarex-jenkins-github-webhook-setting-017]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-017.png
[lindarex-jenkins-github-webhook-setting-018]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-018.png
[lindarex-jenkins-github-webhook-setting-019]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-019.png
[lindarex-jenkins-github-webhook-setting-020]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-020.png
[lindarex-jenkins-github-webhook-setting-021]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-021.png
[lindarex-jenkins-github-webhook-setting-023]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-023.png
[lindarex-jenkins-github-webhook-setting-024]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-024.png
[lindarex-jenkins-github-webhook-setting-025]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-025.png
[lindarex-jenkins-github-webhook-setting-026]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-026.png
[lindarex-jenkins-github-webhook-setting-027]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-027.png
[lindarex-jenkins-github-webhook-setting-028]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-028.png
[lindarex-jenkins-github-webhook-setting-031]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-031.png
[lindarex-jenkins-github-webhook-setting-032]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-032.png
[lindarex-jenkins-github-webhook-setting-033]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-033.png
[lindarex-jenkins-github-webhook-setting-035]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-035.png
[lindarex-jenkins-github-webhook-setting-036]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-036.png
[lindarex-jenkins-github-webhook-setting-037]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-037.png
[lindarex-jenkins-github-webhook-setting-038]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-038.png
[lindarex-jenkins-github-webhook-setting-039]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-039.png
[lindarex-jenkins-github-webhook-setting-040]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-040.png
[lindarex-jenkins-github-webhook-setting-041]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-041.png
[lindarex-jenkins-github-webhook-setting-042]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-042.png
[lindarex-jenkins-github-webhook-setting-043]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-043.png
[lindarex-jenkins-github-webhook-setting-044]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-044.png
[lindarex-jenkins-github-webhook-setting-045]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-045.png
[lindarex-jenkins-github-webhook-setting-046]:/assets/images/2020-04-23-jenkins-github-webhook-setting/lindarex-jenkins-github-webhook-setting-046.png


## 참고(REFERENCES)
- [https://developer.github.com/webhooks/](https://developer.github.com/webhooks/){: target="\_blank"}
- [https://devcenter.bitrise.io/webhooks/adding-a-github-webhook/](https://devcenter.bitrise.io/webhooks/adding-a-github-webhook/){: target="\_blank"}
- [https://dzone.com/articles/adding-a-github-webhook-in-your-jenkins-pipeline](https://dzone.com/articles/adding-a-github-webhook-in-your-jenkins-pipeline){: target="\_blank"}
