---
title: "Travis CI로 Jekyll 블로그를 GitHub Pages에 자동 배포하기"
categories: 
  - travis-ci
tags: 
  - "travis ci"
  - "github pages"
  - jekyll
---


이 포스트에서는 Travis CI(이하 travis)를 이용해 Jekyll 블로그(이하 jekyll)를 GitHub Pages(이하 github)에 자동 배포하는 방법을 소개합니다.


## 선행조건(PREREQUISITE)
- GitHub 계정이 필요합니다.
- GitHub Pages의 브랜치(이하 branch)는 빌드(이하 build) 후 배포되는 master branch, jekyll code를 올릴 sources branch로 구성합니다.

## 테스트 환경(TEST ENVIRONMENT)
- Chrome v80.0.3987.116(공식 빌드) (64비트)
- Firefox Browser v74.0b5 (64-비트)


## 요약(SUMMARY)
1. GitHub Personal access token 생성
2. Travis CI 설정
3. travis.yml 파일 작성
4. 배포 확인


## 내용(CONTENTS)
### 1. GitHub Personal access token(이하 token) 생성
> [https://github.com/](https://github.com/){: target="_blank"}에 접속합니다.

- 우측 상단의 Profile 아이콘을 선택하여 'Settings' 페이지로 이동합니다.

![lindarex-travis-github-pages-jekyll-setting-001]

- 좌측 중간에 Personal access tokens 메뉴를 클릭합니다.

![lindarex-travis-github-pages-jekyll-setting-002]

- GitHub 계정의 비밀번호를 입력합니다.

![lindarex-travis-github-pages-jekyll-setting-003]

- token 이름을 입력하고, token으로 접근 가능한 Scope(repo 전체)를 체크하고, 화면 하단의 'Generate token'을 클릭합니다.

![lindarex-travis-github-pages-jekyll-setting-004]

![lindarex-travis-github-pages-jekyll-setting-004-1]

- 생성된 token 값은 travis 설정에 필요하기 때문에 메모해 둡니다.

![lindarex-travis-github-pages-jekyll-setting-005]

### 2. Travis CI 설정

> [https://github.com/apps/travis-ci](https://github.com/apps/travis-ci){: target="_blank"}에 접속합니다.

- 우측 상단의 'Install'을 클릭합니다.

![lindarex-travis-github-pages-jekyll-setting-006]

- 'Only select repositories' 영역의 'Select repositories'를 클릭하여 travis를 연결할 레파지토리(이하 repository를 선택하고 'Install'을 클릭합니다.

![lindarex-travis-github-pages-jekyll-setting-007]

- travis가 GitHub repository에 접근할 수 있도록 권한 부여에 승인합니다.

![lindarex-travis-github-pages-jekyll-setting-008]

![lindarex-travis-github-pages-jekyll-setting-009]

- [https://travis-ci.com/](https://travis-ci.com/){: target="_blank"}으로 이동하여 승인이 진행됩니다.

![lindarex-travis-github-pages-jekyll-setting-010]

- 우측 하단의 'Settings'를 클릭합니다.

![lindarex-travis-github-pages-jekyll-setting-011]

- 'Environment Variables' 영역에서 아래와 같이 입력하고 'Add'를 클릭합니다.
    + Name :: LINDAREX_GITHUB_TOKEN
    + Value :: 메모해 두었던 token

> 위 Name은 임의로 입력해도 무관합니다. 단, 이후 travis 설정과 동일해야 합니다.

![lindarex-travis-github-pages-jekyll-setting-013]

### 3. travis.yml 파일 작성

> 실제 생성하는 파일명은 '.travis.yml'입니다.

- 아래와 같이 '.travis.yml' 파일을 작성하여 jekyll 최상위 디렉터리(index.html 파일이 있는 경로)에 저장합니다.
- travis 설정 시 Environment Variable에 입력한 name과 아래 github_token 값이 동일해야 합니다.

```ruby
language: ruby
rvm:
- 2.3.3

script: bundle install && bundle exec jekyll build

exclude: [vendor]

deploy:
  provider: pages
  skip_cleanup: true
  github_token: $LINDAREX_GITHUB_TOKEN
  keep_history: true
  local-dir: ./_site
  target-branch: master
  on:
    branch: sources

branches:
  only:
  - sources

```

### 4. 배포 확인

- 설정 완료 후, [travis 첫 페이지](https://travis-ci.com/){: target="_blank"}으로 이동합니다.

![lindarex-travis-github-pages-jekyll-setting-014]

- 로컬의 sources branch를 원격 저장소로 Push 하면, travis에서 build하여 master branch로 자동 배포됩니다.

![lindarex-travis-github-pages-jekyll-setting-015]

- build 중입니다.

![lindarex-travis-github-pages-jekyll-setting-016]

- 정상 배포되었습니다.

![lindarex-travis-github-pages-jekyll-setting-017]


## 마무리(CONCLUSION)
travis로 jekyll을 github에 자동 배포하는 방법을 완료했습니다. <br />
다음 포스트에서는 travis로 다른 언어를 배포하는 방법을 소개하겠습니다.


## 참고(REFERENCES)
- [https://travis-ci.com/](https://travis-ci.com/){: target="_blank"}


[lindarex-travis-github-pages-jekyll-setting-001]:/assets/images/2020-02-21-travis-github-pages-jekyll-setting/lindarex-travis-github-pages-jekyll-setting-001.png
[lindarex-travis-github-pages-jekyll-setting-002]:/assets/images/2020-02-21-travis-github-pages-jekyll-setting/lindarex-travis-github-pages-jekyll-setting-002.png
[lindarex-travis-github-pages-jekyll-setting-003]:/assets/images/2020-02-21-travis-github-pages-jekyll-setting/lindarex-travis-github-pages-jekyll-setting-003.png
[lindarex-travis-github-pages-jekyll-setting-004]:/assets/images/2020-02-21-travis-github-pages-jekyll-setting/lindarex-travis-github-pages-jekyll-setting-004.png
[lindarex-travis-github-pages-jekyll-setting-004-1]:/assets/images/2020-02-21-travis-github-pages-jekyll-setting/lindarex-travis-github-pages-jekyll-setting-004-1.png
[lindarex-travis-github-pages-jekyll-setting-005]:/assets/images/2020-02-21-travis-github-pages-jekyll-setting/lindarex-travis-github-pages-jekyll-setting-005.png
[lindarex-travis-github-pages-jekyll-setting-006]:/assets/images/2020-02-21-travis-github-pages-jekyll-setting/lindarex-travis-github-pages-jekyll-setting-006.png
[lindarex-travis-github-pages-jekyll-setting-007]:/assets/images/2020-02-21-travis-github-pages-jekyll-setting/lindarex-travis-github-pages-jekyll-setting-007.png
[lindarex-travis-github-pages-jekyll-setting-008]:/assets/images/2020-02-21-travis-github-pages-jekyll-setting/lindarex-travis-github-pages-jekyll-setting-008.png
[lindarex-travis-github-pages-jekyll-setting-009]:/assets/images/2020-02-21-travis-github-pages-jekyll-setting/lindarex-travis-github-pages-jekyll-setting-009.png
[lindarex-travis-github-pages-jekyll-setting-010]:/assets/images/2020-02-21-travis-github-pages-jekyll-setting/lindarex-travis-github-pages-jekyll-setting-010.png
[lindarex-travis-github-pages-jekyll-setting-011]:/assets/images/2020-02-21-travis-github-pages-jekyll-setting/lindarex-travis-github-pages-jekyll-setting-011.png
[lindarex-travis-github-pages-jekyll-setting-013]:/assets/images/2020-02-21-travis-github-pages-jekyll-setting/lindarex-travis-github-pages-jekyll-setting-013.png
[lindarex-travis-github-pages-jekyll-setting-014]:/assets/images/2020-02-21-travis-github-pages-jekyll-setting/lindarex-travis-github-pages-jekyll-setting-014.png
[lindarex-travis-github-pages-jekyll-setting-015]:/assets/images/2020-02-21-travis-github-pages-jekyll-setting/lindarex-travis-github-pages-jekyll-setting-015.png
[lindarex-travis-github-pages-jekyll-setting-016]:/assets/images/2020-02-21-travis-github-pages-jekyll-setting/lindarex-travis-github-pages-jekyll-setting-016.png
[lindarex-travis-github-pages-jekyll-setting-017]:/assets/images/2020-02-21-travis-github-pages-jekyll-setting/lindarex-travis-github-pages-jekyll-setting-017.png

