---
layout: single
title: "[React] 클라이언트 컴포넌트 vs 서버 컴포넌트"
categories: web/react
---

## 결론 : Sever Component 우선, 이유가 없다면 모든 컴포넌트를 Sever Component로 만든다.

- 성능향상 : 초기 페이지 로딩 시 js 번들 크기가 줄어들고, 페이지 로드 시간이 빨라짐.
- 보안 강화 : 민감한 데이터(API 키, 데이터베이스 연결 정보)를 서버에만 유지할 수 있음.
- 데이터 패칭 용이 : 서버에서 직접 데이이베이스나 외부 API에 접근, 클라이언트에서 데이터를 패칭할 때 발생하는 추가적인 네트워크 roundtrip을 줄일 수 있음
- SEO 개선 : 서버에서 HTML이 미리 렌더링되므로 검색 엔진 크롤러가 콘텐츠를 더 쉽게 인덱싱 가능

## Client Component로 만들어야 하는 경우

#### 상호작용이 필요한 경우 (Interactivity):

- onClick, onChange, onSubmit 등과 같은 이벤트 핸들러를 사용하는 경우.
- useState, useEffect와 같은 React Hooks를 사용하여 상태를 관리하거나 사이드 이펙트를 처리하는 경우.
- 사용자 입력에 따라 UI가 동적으로 변경되어야 하는 경우. (예: 카운터, 토글 버튼, 폼 입력 필드)

#### 브라우저 전용 API에 접근해야 하는 경우:

- window, document, localStorage, navigator 등과 같은 브라우저 전용 객체에 접근해야 하는 경우. (예: window.innerWidth를 사용하여 반응형 UI를 만들거나 localStorage에 사용자 설정을 저장하는 경우)

#### Third-Party 라이브러리가 Client-side 렌더링을 요구하는 경우:

- React Context API를 사용하는 라이브러리.
- 특정 DOM 조작이나 애니메이션 라이브러리 (예: Framer Motion, GSAP).
- 브라우저 환경에 의존하는 UI 라이브러리 (예: 일부 지도 라이브러리, 차트 라이브러리).

#### 클라이언트에서만 사용되는 컴포넌트:

- 서버에서 미리 렌더링될 필요 없이, 클라이언트에서만 필요할 때 로드되어야 하는 컴포넌트.
