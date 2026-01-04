---
layout: single
title: "[verilog] Verilog 소개 및 concept"
categories: hardware/verilog
published: false
---

## Verilog로 묘사하고자 하는 것

#### verilog의 계층 구조

- Module Top : Behavior + Connection Description
  - Module A : Behavior + Connection Description
    - Module A_A : Behavior Description
    - Module A_B : Behavior Description
    - Module A_C : Behavior Description
  - Module B : Behavior Description
  - Module C : Behavior Description

Behavior Description : 동작 묘사
Connection Description : 연결 묘사 (자식 모듈이 어떻게 되결 되었는지 묘사)

#### cycle-based results

- clock, cycle 을 가지고, 각 시간별로 어떤 사이클에 어떤 동작을 하는지가 각각의 결과로 나타내지는 언어임.
- 일반 프로그래밍 언어가 한 번에 한 라인이 실행되는 것과는 다르게 각갹의 라인들이 동시에 실행됨

## Verilog와 다른 설계 방법과의 비교

- 모듈 : 기능별로 나누는 단위
- 함수처럼 어딘가 들어가서 return한다는 개념은 없음
- verilog는 하드웨어를 만들기위한 언어임 : 칩/회로가 어떻게 구성되어있는가를 나타냄
- 동시에 라인이 수행됨
- 묘사방식 차이
  - c : 동작이 어떤식으로 수행되는지 묘사하는 것임, 예) z = a + b , a + b를 할고 z에 넣는 동작을 수행함
  - 베릴로그/vhdl : 동작이 수행되는 로직이 어떻게 구성되었는지를 묘사, 예) z = a + b, a와 b를 input으로 받아 더해 z로 output하는 회로를 구현함

#### 아날로그 vs 디지털

- 아날로그 설계는 각 시그널의 움직임에 예민하고, 마우스로 그림을 한땀 한땀 그리는 형태로 보통 제작됨. 메모리에서 자주 쓰임
- 디서털 설계는 0,1을 가지고 복잡한 로직을 구성할 때 사용. verilog/vhdl을 사용하고 verilog를 많이 사용하는 추세

## Free tool 소개 (회사에서 쓰는 툴은 좀 비쌈)

- Simulatore와 debugger가 필요함!
- Synopsys/Cadence/Siemens EDA 업체에서 제공함 -> 비싸고 개인이 사용이 어려움
- Free Tools
  - Icarus Verilog
  - edaplayground (이걸 더 추천함) : 웹사이트임

#### 언어

- system verilog는 verilog를 업데이트한 버전정도로 보면됨
- vhdl 은 verilog는 옛날 verilog라고 보면됨

#### uvm

- 검증시 사용
