---
layout: single
title: "[Next.js] Route 방법"
categories: web/nextjs
---

#### `<Link>` 컴포넌트를 사용하는 방법

```ts
import Link from "next/link";

function HomePage() {
  return <Link href="/about">About Us</Link>;
}
```

#### `useRouter` 훅을 사용하는 방법

> AppRouter(NextJS 13이상) import 시 `next/navigation`을 해야함.

- 'next/router'를 사용할 경우, `Error: NextRouter was not mounted.` 에러가 남

```ts
"use client"; // useRouter is a client-side hook
import { useRouter } from "next/navigation";

function MyComponent() {
  const router = useRouter();

  const handleClick = () => {
    router.push("/products/123");
  };

  return <button onClick={handleClick}>Go to Product</button>;
}
```
