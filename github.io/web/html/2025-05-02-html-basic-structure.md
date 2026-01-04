---
layout: single
title: "[HTML] 기본 구조 & 시맨틱 태그"
categories: web/html
---

## 기본 구조

```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>웹 페이지 제목</title>
    <link rel="stylesheet" href="style.css" />
    <script src="script.js"></script>
  </head>
  <body>
    ....
  </body>
</html>
```

- `<!DOCTYPE html>` : 이 문서는 HTML5 표준을 따른다는 것을 웹 브라우저에 알려주는 선언. 항상 문서의 맨 위에 있어야 함.
- `<html lang="ko">` : HTML 문서의 시작 태그. lang="ko"는 문서의 주 언어가 한국어임을 나타냄. (검색 엔진같은 도구들이 콘텐츠를 더 잘 이해하도록 도움)
- `<head>` : 문서의 메타데이터(문서 자체에 대한 정보)를 담는 부분. 이 안에 있는 내용은 웹 브라우저 화면에 직접 표시되지 않음.
  - `<meta charset="UTF-8">` : 문자 인코딩을 UTF-8로 설정하여 한글을 포함한 다양한 언어가 깨지지 않고 올바르게 표시되도록 함.
  - `<meta name="viewport" content="width=device-width, initial-scale=1.0">` : 모바일 기기 등 다양한 화면 크기에서 웹 페이지가 올바르게 표시되도록 뷰포트를 설정. 요즘 반응형 웹 디자인의 필수 요소.
  - `<title>` : 웹 브라우저 탭이나 즐겨찾기에 표시될 웹 페이지의 제목
  - `<link rel="stylesheet" href="style.css">` : 외부 CSS 파일을 연결할 때 사용
  - `<script src="script.js"/>` : 외부 JavaScript 파일을 연결할 때 사용
- `<body>` : 웹 브라우저 화면에 실제로 표시될 모든 콘텐츠(텍스트, 이미지, 링크, 버튼 등)가 들어가는 부분

#### 시맨틱 태그 (Semantic Tags)

> 시맨틱 태그 : 태그 자체가 의미를 가짐. (단순한 div가 아니라, 개발자 혹은 검색 엔진 같은 도구들에게 여기는 이런 부분이야 하고 알려주는 역할.)

```html
<body>
  <header>
    <h1>우리 웹사이트의 로고 또는 메인 제목</h1>
    <nav>
      <ul>
        <li><a href="#">메뉴 1</a></li>
        <li><a href="#">메뉴 2</a></li>
      </ul>
    </nav>
  </header>

  <main>
    <h1>제목</h1>
    <p>내용</p>
  </main>

  <footer>
    <p>Copyright © 2024 내 회사. 모든 권리 보유.</p>
  </footer>
</body>
```

- `<header>` : 페이지의 머리글 영역을 의미. 보통 웹사이트의 로고, 메인 제목, 내비게이션 등이 들어감
- `<nav>` : 내비게이션 링크(메뉴)들을 포함하는 영역을 의미
- `<main>` : 문서의 주요 콘텐츠 영의을 의미. 페이지당 한 번만 사용
- `<footer>`: 페이지의 바닥글 영역을 정의. 예. 저작권 정보, 연락처, 관련 링크 등이 들어감
