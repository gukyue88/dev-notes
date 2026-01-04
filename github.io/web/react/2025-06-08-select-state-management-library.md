---
layout: single
title: "[React] 전역 상태 관리 라이브러리 선택하기"
categories: web/react
---

> 결론 : zustand + react query

간단한 Recoil을 사용하려고 했으나, 더이상 지원이 안되는 부분이 있어 새로운 전역 상태 관리 라이브러리를 고민하게 됨.

#### 전역 상태 관리 라이브러리 후보

- redux
  - 장점 : Redux DevTools의 강력함, 방대한 커뮤니티 및 미들웨어 생태계(redux-saga, redux-thunk 등), 엄격한 상태 변화 규율(모든 상태 변화를 기록하여 상태변화의 원인과 과정을 매우 명확하게 알 수 있음), RTK Query(강력한 서버 상태 관리 기능), 큰 프로젝트에 유리
  - 단점 : 높은 복잡도, Provider 필요, 크기가 큼, 보일러플레이트 코드
- zustand + react query
  - 장점 : 간결함, 빠른 개발 속도, 쉬운 학습 곡선, Provider 불필요, DevTools 지원, 크기가 작음, 성능, 유연성, 훌륭한 typescript 지원, 훅기반
  - 단점 : redux에 비해 커뮤니티가 작음,

#### 무엇을 선택할 것인가? zustand + react query

- 최근 트렌드는 "서버 상태관리와 클라이언트 상태 관리를 분리" 하는 것이라 함.
  - 클라이언트 상태 관리는 zustand로 서버 상태 관리는 react query로 분리하여 개발
- Redux DevTools 만큼 강력하진 않지만 DevTools 지원
- 대부분의 복잡도가 높지 않은 프로젝트의 경우, zustand 만으로 충분히 효율적인 상태관리가 가능
- zustand에서도 예측 가능한 상태 변화를 구현은 가능하지만, redux처럼 이를 강력하게 제지하지는 않음
- 초대규모 애플리케이션에서는 극도로 엄격한 상태 관리 규율과 최고 수준의 디버깅 도구가 필요하겠지만, 대부분의 웹 애플리케이션에서는 zustand가 더 효율적이고 생산적인 상태관리 솔루션임
