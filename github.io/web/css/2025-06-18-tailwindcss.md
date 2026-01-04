---
layout: single
title: "[CSS] Tailwind CSS"
categories: web/css
---

#### tailwindcss?

유틸리티 우선(Utility-First) 개발 방식의 CSS 프레임워크

- 기존 방식은 하나의 클래스 안에 여러 스타일 속성을 정의하고 스타일을 적용함.
- 유틸리티 우선 개발 방식은 클래스를 아주 작게 쪼갠 후, HTML 태그 안에서 클래스들을 조합하는 방식임.
- HTML 내에서 직접 스타일링하여 개발 속도가 향상
- 클래스 이름에 대한 고민이나 스타일 구조화에 대한 고민이 없어짐.
- tailwindcss는 실제로 사용하는 클래스에 대해서만 즉성에서 스타일을 생성하여 css를 간결하고 효율적으로 운영함.
- 반응형 디자인 : `sm:`, `md:`, `lg:`등 지원
- HTML 코드가 지저분해질 수 있으나, React를 사용해 컴포넌트화하여 해결
- 스타일 재사용이 어려움. React를 사용해 컴포넌트화하여 해결

#### tailwindcss 단위

- rem 단위 기준으로 class명이 작동함
- 1rem === 16px(브라우저에 따라 달라짐)

#### tailwind box-sizing

- Tailwind CSS는 기본적으로 모든 요소에 box-sizing: border-box;를 적용함
- 간단히 말해, Tailwind CSS를 사용하면 width나 height를 지정할 때 padding과 border가 해당 너비/높이에 포함되어 계산됨

#### CheatSheet

```
// bg
bg-black
bg-pink-400

// text
text-pink
text-2xl // 크기
text-[60px] // 크기
font-bold
font-[100]

// border
border-8
border-black
rounded-full

// w, h, p, m : width / height / padding / margin
w-64
h-64
p-64
px-64 // x 축에만 적용
py-64 // y 축에만 적용
m-64
mx-64 // x 축에만 적용
my-64 // y 축에만 적용

// flex
flex
flex-col // flex-row가 default 방향임
justify-center // 주축 정렬
item-center // cross축 정렬
gap-64

// hover, transition
- hover:bg-purple
- transition
```
