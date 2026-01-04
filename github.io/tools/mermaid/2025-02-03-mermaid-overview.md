---
layout: single
title: "[mermaid] mermaid Overview"
categories: tools/mermaid
---

텍스트와 코드를 사용해 다이어그램과 시각화를 하게 해줌

- 주요 목적 : 개발 속도에 맞는 문서화
  - 다이어그램 작성 및 문서화는 많은 비용이 발생함
  - 다이어그램이나 문서가 없으면 생산성 저하와 조직의 학습의 어려움을 겪게됨.
  - mermaid는 다이어그램을 쉽게 수정하고 생성할 수 있고, 프로덕션 스크립트의 일부가 될 수도 있다.

#### 다이어그램 타입

여러 그래프와 시각화가 가능하고, 아래는 주로 사용할 것으로 생각됨.

- Block Diagram : 아키텍처
- class diagram : SW 구조
- sequnce diagram : sequnce
- Entity Relationship Diagram : 데이터 베이스 관계
- flowchart : 분기 구조
- 기타 : 여러 차트(파이 차트, XY chart, Radar등), Mindmaps, timeline,

#### Mermaid API

- mermaid js를 base로 동작함.
- mermai를 지원하는 플랫폼을 사용해도 되지만, js를 직접 import 해서 사용하는 방법도 있음.
