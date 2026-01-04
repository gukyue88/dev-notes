---
layout: single
title: "[CSS] Tailwind CSS primary color 변경 및 적용"
categories: web/css
---

#### primary color 변경

> app/globals.css의 `primary` 변경

```css
@layer base {
  :root {
    --primary: 42 100% 55%;
  }
}
```

- hsl 기준으로 작성 필요
- hex to hsl 변경 : https://convertacolor.com/

> 사용시 `xxx-primary` 사용 (예. `bg-primary`)

```tsx
export default function IndexPage() {
  return (
    <main className="min-h-screen flex flex-col items-center justify-center bg-primary"></main>
  );
}
```
