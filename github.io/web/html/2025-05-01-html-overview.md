---
layout: single
title: "[HTML] Overview"
categories: web/html
---

#### 요소

```
<태그>내용(contents)</태그>
```

#### 부모 자식 관계

```
<태그>
	<태그>
		내용
	</태그>
</태그>
```

- 부모 요소 : 나를 감싸고 있는 요소
- 자식 요소 : 내가 감싸고 있는 요소
- 상위 요소 : 부모 요소를 포함한 모든 상위 요소
- 하위 요소 : 자식 요소를 포함한 모든 하위 요소

#### 빈태그

내용(content)이 없는 태그

```
<태그 />
```

- 슬래시를 안붙여도 되지만, 슬래시를 붙이는 것으로 권장
- 예) meta, img, input 등

#### 글자와 상자

요소가 화면에 출력되는 특성을 크게 2가지로 구분함.

- inline 요소 : 글자를 만들기 위한 요소들
  - 예) span
  - 기본적으로 요소가 수평방향으로 쌓임, 줄바꿈을 하면 띄어쓰기로 해석됨
  - 가로 사이즈와 세로 사이즈가 콘텐츠의 크기만큼 줄어듬. (width, height가 안먹힘)
  - padding, margin은 위,아래는 사용할 수 없고, 좌우는 적용됨
  - 기본적으로 인라인 요소 안에는 블록 요소를 넣을 수 없음.
- block 요소 : 상자(레이아웃)을 만들기 위한 요소들
  - 예) div
  - 기본적으로 요소가 수직방향으로 쌓임.
  - width는 부모요소의 크기만큼 자동으로 늘어남.
  - height는 콘텐츠 크기 만큼 줄어듬
  - width, height 속성이 지정됨.
  - padding, margin 위,아래,좌,우 모두 적용됨
  - 블록요소 안에 블록 요소를 넣을 수 있음.
- inline-block 요소 : inline 요소이긴 하나, 몇 가지 block 요소의 특성을 가짐.
  - 예) input
  - 요소가 수평방향으로 쌓임
  - width, height 설정 가능
  - padding, margin 상하좌우 가능

## 핵심 정리

#### 주요 요소

- 블록요소
  - div : 블록특별한 의미가 없는 구분을 위한 요소
  - h1 ~ h6 : 제목을 의미하는 요소
  - p : 문장을 의미하는 요소
  - ul : unordered list. 순서가 필요없는 목록의 집합을 의미. li 태그가 하나 이상 포함
  - li : list item. 목록 내 각 항목.
  - table : 표 요소
    - tr(table row)로 행을 먼저 정의하고, 그 안에 td(table data == 하나의 셀을 의미)가 들어가는 형태
- 인라인요소
  - img : 이미지를 삽입하는 요소. 필수 속성 : src, alt
  - a : 다른/같은 페이지로 이동하는 하이퍼링크를 지정. 필수 속성 : href
  - span : 특별한 의미가 없는 구분을 위한 요소
    - 보통 글자들의 범위를 잡아낼 때 사용
    - `<p>동해물과 <span>백두산이</span> 마르고 닳도록</p>`에서 `백두산이`를 뭔가 따로 처리하고 구분해내고 싶을 떄 사용.
  - br : break, 줄바꿈 요소
    - `<p>동해물과 백두산이 </br>마르고 닳도록</p>`
  - label : 라벨링이 가능한 요소(예. input)에 제목을 포함시키고 싶을 때 사용
    - `<label><input type="checkbox"/> Apple </label>`
    - input 태그 자체는 라벨이 없고, Apple 이라는 label을 붙이기 위해 `label` 태그 안에 input과 Apple을 같이 넣어줌.
    -
- inline-block 요소
  - input : 사용자가 데이터를 입력하는 요소. 필수 속성 : type
    - type : 입력 타입
      - "text"
        - value : 미리 입력된 값
        - placeholder : 입력할 값의 힌트
        - disabled : 비활성화
      - "checkbox"
        - checked : 체크됨
      - “radio" : 그룹에서 한 개만 입력 받을 때 사용
        - name : 그룹의 이름을 지정. 같은 그룹임을 나타낼 때 사용

```
<label>
	<input type="radio" name="fruits" /> Apple
</label>
<label>
	<input type="radio" name="fruits" /> Banana
</label>
```

#### 전역 속성

- 요소들은 각자 자신들이 사용할 수 있는 속성들이 있음. (예. a태그: href, img태그:src)
- 전역 속성은 html의 body에서 사용할 수 있는 모든 태그에서 사용할 수 있는 속성

```
title=“설명” : 마우스를 올렸을 때 설명이 Tooltip처럼 나옴

style="스타일" : 요소에 적용할 스타일(css)를 지정

class="이름" : 요소를 지칭하는 중복 가능한 이름

id="이름“ : 요소를 지칭하는 고유한 이름
```
