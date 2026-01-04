---
layout: single
title: "[verilog] Verilog Overview"
categories: hardware/verilog
---

## What is Verilog?

#### Verilog란?

- HDL (Hardware description language) : 코드 형태로 디지털 시스템과 회로를 표현하는 언어
- 계층 구조로 디지털 회로를 묘사함.
- 여러 모델링 방법을 지원함. 추상화 정도에 따라 모델링 방법이 달라짐
  - Gate-level Modeling : 추상화 정도 == 없음, 그냥 게이트를 일일이 한땀 한땀 표시, 잘 쓰지 않음
  - Data flow Modeling : 추상화 정도 == 중간, **assign**과 연산자를 사용하여 입력과 출력 간의 관계를 정의, 불 대수 식이나 진리표를 코드로 옮기는 수준
  - Behavior Modeling : 추상화 정도 == 높음, C언어와 비슷한 프로그래밍 구성 요소 (if-else, case, for, while 루프)와 **절차적 할당(always, initial 블록 내부)**를 사용하여 시간의 흐림에 따른 신호의 변화를 표현, 기능적 동작에 초점을 맞춤

#### VHDL vs Verilog

- Verilog가 나오기 전에 HDL로 `VHDL`이 사용됨.
- Verilog는 VHDL보다 더 심플한 문법 + 높은 추상화 + 더 좋은 툴 을 지원함.

#### 일반 프로그래밍 언어 vs Verilog

> 일반 프로그래밍 언어는 '컴퓨터 프로세서가 실행할 명령어들을 작성'하는 언어이지만, Verilog는 '디지털 회로를 묘사' 하는 언어임.

- 문법 : Verilog는 '디지털 회로를 묘사'하도록 문법이 만들어짐. (예. wires, registers, logic gate 등)
- 컴파일 : Veriolog는 디지털 회로의 `실행`이 아닌 `묘사`하는데 사용되므로, 직접 실행을 하지 못함. 대신 물리적 회로나 FPGA에서 구현될 하드웨어 Configuration으로 컴파일 됨. 일반 프로그래밍 언어는 컴퓨터 프로세서가 실행할 머신 코드로 컴파일 됨

#### Verilog를 대체할만한 미래의 HDL

- HLS (High-Level Synthesis) : C, C++, SystemC와 같은 언어로 작성된 고급 설명에서 하드웨어 디자인을 자종으로 생성해주는 기술
  - 지금보다 더 높은 수준의 추상화로 설계 의도 및 기능을 표현할 수 있음
- 이외에도 Verilog 및 VHDL의 한계를 해결하ㅕ고 하는 여러 시도들이 이어지고 있음.

#### 참고 자료

[chipverify.com/tutorials/verilog](chipverify.com/tutorials/verilog)
