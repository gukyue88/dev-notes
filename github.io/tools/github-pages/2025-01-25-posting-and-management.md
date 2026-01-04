---
layout: single
title: "[Github Pages] 포스팅 및 관리"
categories: tools/github-pages
---

## 기본 포스팅

- `_posts` 라는 폴더 밑에 md 파일 생성
- `2021-01-13-{url}.md` 형태로 이름 생성

```
---
layout: single
title: ‘첫 포스팅’
published : false # 이건 포스팅 비공개시에 사용
---

내용작성
```

## url 생성 규칙

`_config.yml`를 보면 permalink가 url이 생성이 어떻게 되는지 알 수 있음.

```yml
permalink: /:categories/:title/
```

- 기본은 categories와 title의 조합으로 이루어지고, categories 가 없을 경우, title만 가지고 url이 생성된다.
- title은 파일의 날짜 부분을 제외한 부분임.
- 예 1. `2025-01-24-how-to-github.md` => `{기본url}/how-to-github` (categories가 없는 경우)
- 예 2. `2025-01-24-how-to-github.md` => `{기본url}/github-pages/how-to-github-pages` (categories가 `github-pages`인 경우)

## 나의 글 관리 방법

글 관리 방법은 여러 방법이 있지만, 아래는 이 블로그에서 글들을 어떻게 관리하고 요약해서 보여줄지에 대한 내용임.

> 대분류용 카테고리를 만들고, 그 안에서 단일 포스트나 하위 카테고리(시리즈 개념)를 둔다.
> 대분류용 카테고리에 대한 아카이브 페이지를 만들고, 필요에 따라 하위 케테고리에 대한 페이지를 별도로 만든다.
> 하위 카테고리 안에서도 또 하위 카테고리를 만들 수 있도록 한다.

#### 폴더 및 파일 구조

- `_posts`
  - 대분류용 폴더 (`hardware`, `system`, `web`, `application`, `tools`)
    - 단일 마크다운 파일 또는 하위 카테고리 폴더 안에 여러 마크다운 파일 (예. github-pages, nextjs, expo...)
      - 각 포스트들은 해당 폴더의 categories를 갖도록 설정
        - 단일 파일의 경우, categories를 해당 대분류로 지정 (예. `categories: tools`)
        - 하위 카테고리가 있는 경우, cateigories를 대분류 + 하위 카테고리로 지정 (예. `categories: tools/github-pages`)
        - 하위 카테고리 안에 또 하위 카테고리가 있는 경우, 모두 합침 (예. `categories: tools/github-pages/trouble-shooting`)
      - 각 포스트의 title에는 대분류 또는 중분류에 대한 내용을 앞에 붙인다. (예. `title: [Tools] ...` 또는 `[Github Pages] ...`)

> `categories`는 포스트의 속성 지정 부분(`---`으로 감싸져 있는 부분)에 지정 가능
> categories의 띄어 쓰기를 포함하는 경우, 아카이브페이지를 만드는데 문제가 있어 `-`를 띄어쓰기 대신 사용하기로 함.

#### 아카이브 페이지 (카테고리에 해당하는 글들의 링크 모음)

1. `archive-single-category.html` 생성

> 기존 `archive-single.html`의 경우, 폰트사이즈가 크고 다른 내용도 많이 들어가 있어 여러 글의 제목을 한 번에 모아보기 어려움.

카테고리용 아카이브 페이지 `_includes/archive-single-category.html` 생성

```html
{% raw %}
{% if post.header.teaser %}
  {% capture teaser %}{{ post.header.teaser }}{% endcapture %}
{% else %}
  {% assign teaser = site.teaser %}
{% endif %}

{% if post.id %}
  {% assign title = post.title | markdownify | remove: "<p>" | remove: "</p>" %}
{% else %}
  {% assign title = post.title %}
{% endif %}
{% endraw %}

<div class="{{ include.type | default: "list" }}__item">
    <article class="archive-item">
      <div>
          <span>
            <a href="{{ post.url }}">{{post.title}}</a>
          </span>
      </div>
    </article>
</div>
```

2. `_pages` 아래 각 카테고대에 대한 아카이브 페이지 생성

> 위에서 생성한 `archive-single-categories.html`을 포맷으로 하는 아카이브 페이지를 생성

- permalink는 categories에 대한 페이지임을 명시해고 다른 포스트의 url과 중복을 방지하기 위해 `/categories/{category}` 형태로 생성함. (예. `/categories/hardware`)
- 하위 카테고리에 대한 아카이브 페이지를 만들 경우(글이 너무 많을 경우), 상위 카테고리까지 포함하여 permalink를 정의 (예. `/categories/tools/github-pages`)

예. `_pages/tools.md`

```
---
title: "Tools"
layout: archive
permalink: /categories/tools
---

{% raw %}
{% assign posts = site.categories.tools %}
{% for post in posts %} {% include archive-single-categories.html type=page.entries_layout %} {% endfor %}
{% endraw %}

#### github-pages

{% raw %}
{% assign posts = site.categories["tools/github-pages"] %}
{% for post in posts %} {% include archive-single-categories.html type=page.entries_layout %} {% endfor %}
{% endraw %}


```

#### navigation 변수 설정

위에서 만든 페이지들을 navigation 변수로 등록 (`_data/navigation.yml`)

```
# category links for sidebar
sidebar_category:
  - title: "Hardware"
    url: https://gukyue88.github.io/categories/hardware
  - title: "System"
    url: https://gukyue88.github.io/categories/system
  - title: "Web"
    url: https://gukyue88.github.io/categories/web
  - title: "Application"
    url: https://gukyue88.github.io/categories/application
  - title: "Tools"
    url: https://gukyue88.github.io/categories/tools
```

#### navigation 변수(`sidebar_category`) 등록

> 아래 2가지 모두 반영 필요

1. 루트 폴더의 `index.html`의 `---`에 아래 내용 추가

> 처음 화면인 `index` 페이지에서 sidebar를 추가하는 과정

```html
---
sidebar:
  nav: sidebar_category
---
```

2. `_config.yml`의 `defaluts`의 `values` 안에 아래 내용 추가

> 각 포스트에서 sidebar를 추가하는 과정. (각 포스트마다 이 작업을 해줘도 되지만 귀찮음)

```yml
# Defaults
defaults:
  # _posts
  - scope:
      ...
    values:
      ...
      sidebar:
        nav: sidebar_category
```
