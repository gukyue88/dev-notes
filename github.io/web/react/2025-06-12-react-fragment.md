---
layout: single
title: "[react] Fragment"
categories: web/react
---

> Fragment : DOM 노드 추가 없이, 여러 요소를 반환하는 방법

- 컴포넌트가 반환하는 요소는 하나의 부모 요소로 감싸져 있어야 함.
- 여러 요소를 반환하는 경우, 부모 요소로 감싸야하는데, DOM 트리의 깊이가 깊어짐.
- Fragment(`<></>`)를 사용하면 DOM 노드 추가 없이, 여러 요소를 묶어서 반환 가능함.

```tsx
function MyComponent() {
  return (
    <>
      <h1>title</h1>
      <p>content</p>
    </>
  );
}
```
