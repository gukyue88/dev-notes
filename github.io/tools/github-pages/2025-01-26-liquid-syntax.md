---
layout: single
title: "[Github Pages] liquid syntax"
categories: tools/github-pages
---

> markdown 코드 블록안에 있는 내용이 해석이 잘못되면서 build가 안되거나, 이상한 내용이 페이지에 들어올 때가 있음.

github pages는 동적인 웹페이지를 생성하기 위해 liquid 구문을 사용함.

#### liquid 구문 예

```
{% raw %}

# 변수
{% assign name = "뤼튼" %}
Hello, {{ name }}!

# 루프
{% for post in site.posts %}
  - {{ post.title }}
{% endfor %}

# 필터
{{ "hello" | upcase }}  // 결과: HELLO

{% endraw %}
```

> 그런데 이런 부분 때문에 내 일반적인 코드도 liquid 구문으로 해석하면서, build 에러가 발생하는 경우가 발생함.

liquid 구문으로 해석되지 않게 하는 방법 (escape 방법)

```
{% raw %}
{% r a w %}
 // 이 부분에 내용 작성
{% e n d r a w %}
{% endraw %}
```

> r a w 는 raw로, e n d r a w는 endraw로 붙여써야 함!
