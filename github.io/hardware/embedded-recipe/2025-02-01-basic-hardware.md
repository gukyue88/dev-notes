---
layout: single
title: "[임베디드레시피][핵심요약] 하드웨어 기초"
categories: hardware/embedded-recipe
---

## Bus

> 버스 : 데이터를 운송하는 이동 수단, 사실은 연결된 선이고 공유가 되어있어 허가 및 사용 절차가 필요함

- 아비터 : 어느 장치가 버스를 사용할 지 결정해주는 역할
- 버스 사용 가능 여부 선을(예. S0, S1) 연결해놓고, 각 장치가 자기에 맞는 S0, S1 상태일때 버스(wire)를 사용하는 형태
- CPU 내부에서는 CU가 아비터 역할을 함

## Timing 및 Spec 읽기

> 실제로 입력신호와 출력신호의 시간의 차이가 발생하고 이런 부분이 타이밍도에 반영되어있음

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FbbjJgT%2FbtsgE7XiFh3%2FAAAAAAAAAAAAAAAAAAAAAIrztkzRFBhgvkFZsc4MwORyEPWLwZ0CtIcbSYxTjHh6%2Fimg.jpg%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1756652399%26allow_ip%3D%26allow_referer%3D%26signature%3DeTSV8k2VazYkJBuYlCo6UAAKgwY%253D)

- Transient : 신호가 변화하는 구간 또는 시스템 안정화 구간
- Impedance : 저항이 무한대 상태이며, 아무 신호도 없는 상태
- Heavy line : 타이밍 다이어그램 중 의미가 있는 구간

## 메모리

![메모리의종류](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fyc0xA%2FbtsgC4fMPDV%2FAAAAAAAAAAAAAAAAAAAAAG7cQmcgWrbkSDWJjzZ3-8Ek0HGe0YVtvZ6x8vwL9MR-%2Fimg.jpg%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1756652399%26allow_ip%3D%26allow_referer%3D%26signature%3DEW7bvxn1ineY3udv1uoJi9PpnA8%253D)

- RAM : Random Access Memory, 휘발성
  - SRAM : Static RAM, 가장 비쌈
  - DRAM : Dynamic RAM, 가장 쌈, recharging 직접 필요
    - SDRAM : Sychronous DRAM, 시스템 클락과 동기를 맞춰 동작
    - DDR SDRAM : Double Data Rate SDRAM, bus clock의 rising edge, falling edge를 모두 사용하여 시스템 클락의 두 배로 데이터를 전송 가능
  - PSRAM : SRAM 처럼 보이는 DRAM, 중간 가격, recharging을 내부적으로 함
- ROM : Read Only Memory, 발성
  - Flash Memory : ROM이긴 하지만 쓰고 지울 수 있는 메모리
    - NOR Memory : 랜덤 억세스 가능, 구조로 인해 비쌈, Write/Erase 느림, Read 빠름
    - NAND Memory : 페이지 단위 억세스 가능(512 바이트, 2KB 등), Write/Erase 빠름, Read 느림

최근 추세는 NAND + DDR SDRAM 을 조합하여 사용함

## RAM 구조

![RAM구조](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FbAUMKG%2FbtsgJZ5eIVM%2FAAAAAAAAAAAAAAAAAAAAAPFlbTVvuB2_QNldRAmX6H-ULsxYB2ViaVHhx6m-3Wg7%2Fimg.jpg%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1756652399%26allow_ip%3D%26allow_referer%3D%26signature%3DvbX84fx0RR09jRlcC5Om4hBx0Bg%253D)

> RD/WR, Address Lines, Data Lines로 구성됨

## 일반적인 CPU 동작과 Pipe Line

![CPU](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FkCxv1%2FbtsgMarXi2X%2FAAAAAAAAAAAAAAAAAAAAAAL3fbIlSAs8RfEZSacXaxRRTbEMo8OeHc_xskBPxmyP%2Fimg.jpg%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1756652399%26allow_ip%3D%26allow_referer%3D%26signature%3Dp9FrXsEu3c5mQ04XDF%252FTxqYiMWc%253D)

- PC : fetch할 instruction이 담긴 주소
- IR(Instruction Register) : fetch된 instruction을 담는 공간
- Address Register : 현재 사용중인 데이터의 주소를 담는 공간
- Data Register : Address Register가 실제 가리키는 값
- ACC : 연산에 사용된 값을 임시 저장하는 공간, 외부 접근이 아닌 CPU 내부적으로만 사용
- Decoder : IR에서 가져온 Instruction을 해석해서 CU에게 넘김
- CU(Central Unit) : Decode된 Instruction을 각종 제어 신호로 변환하여 제어신호 발생
- ALU : 산술 연산 담당

![파이프라인](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FdfPMrR%2FbtsgG5EHIZv%2FAAAAAAAAAAAAAAAAAAAAAEAg9S_cT3tySsrcJs7Gp_VRbddjngmV_P2o88SPomhl%2Fimg.jpg%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1756652399%26allow_ip%3D%26allow_referer%3D%26signature%3DDlXRKtgQE1uWairdzD7ch%252FUYAeM%253D)

동일 시간내 많은 instruction을 처리하기 위해 아래와 같이 동작을 구분하고 파이프라인 형태로 운영함

- Fetch : 메모리에서 Instruction을 가져옴
- Decode : Instruction을 석하고, Register 값들도 확인
- Execute : Decode된 Instruction을 실행
