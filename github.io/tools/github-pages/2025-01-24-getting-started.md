---
layout: single
title: "[Github Pages] 시작하기 (feat. minimal-mistakes)"
categories: tools/github-pages
---

## 프로젝트 생성

- [minimal-mistakes repository](https://github.com/mmistakes/minimal-mistakes) fork 하기
- 해당 repository 이름을 "{username}.gihub.io" 로 변경

## 배포

`_config.yml` 파일 수정후 커밋

```yml
url: “https://{username}.github.io“
```

## Configuration

> `_config.yml` 파일 수정후 커밋

#### 지역 설정

```yml
locale: "ko-KR" # 블로그의 기본 UI 텍스트(예. "Read more", "Comments" 등)이 한국어로 표시됨
timezone: Asia/Seoul # 블로그 내 시간 관련 요소(예. 게시물 작성 시간)들을 한국 시간에 맞춤
```

#### 블로그 기본 정보

```yml
title: "" # 사이트의 이름, <title> 태그에 사용
description: "" # 사이트의 description에 사용, SEO 향상에 도움, 직접적으로 보여지는 부분은 아님
logo: "/assets/images/logo.png" # 로고 이미지
```

### 저자 정보

```yml
author:
  name: "Your Name"
  avatar: # path of avatar image, e.g. "/assets/images/bio-photo.jpg"
  bio: "I am an **amazing** person."
  location: "Somewhere"
  email: # 아래 이메일 링크를 활용, 여기에 설정하면 중복됨
  links:
    - label: "Email"
      icon: "fas fa-fw fa-envelope-square"
      # url: "mailto:your.name@email.com"
    - label: "Website"
      icon: "fas fa-fw fa-link"
      # url: "https://your-website.com"
    - label: "Twitter"
      icon: "fab fa-fw fa-twitter-square"
      # url: "https://twitter.com/"
    - label: "Facebook"
      icon: "fab fa-fw fa-facebook-square"
      # url: "https://facebook.com/"
    - label: "GitHub"
      icon: "fab fa-fw fa-github"
      # url: "https://github.com/"
    - label: "Instagram"
      icon: "fab fa-fw fa-instagram"
      # url: "https://instagram.com/"
```

#### 기타 설정

```yml
enable_copy_code_button: true # 코드 블록 복사 가능 여부 설정
search: true # 사이트에서 post 검색 기능 활성화 여부 설정
```

## favicon 설정

- favicon 생성 후, assets 폴더에 포함
- `_include/head/custom.html` 에 아래 내용 추가

```html
<link rel="icon" href="/assets/favicon.ico" type="image/x-icon" />
```
