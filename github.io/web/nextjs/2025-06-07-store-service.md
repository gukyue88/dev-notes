---
layout: single
title: "[Next.js] Service와 Store 분리 구조"
categories: web/nextjs
---

> Service(데이터소스 접근) 코드과 Store(상태 관리) 코드를 분리하자!

```
data sources <-> services <-> stores <-> components
```

#### services

- 역할 : 데이터 소스(supabase)와 직접적인 통신
- 구현 내용 : 데이터소스 CRUD 로직

#### stores

- 역할 : 서비스 레이어에서 가져온 데이터를 전역 상태로 저장하고 관리
- 구현 내용 : 현재 상태, 컴포넌트 입장에서 필요한 상태 변경 인터페이스
