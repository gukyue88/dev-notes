---
layout: single
title: "[임베디드레시피][핵심요약] ARM 미장센"
categories: hardware/embedded-recipe
---

## ARM Assembly : ADS vs GNU

Assembly HelloWorld
![Assembly HelloWorld](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FcodKbD%2Fbtshr6hM16K%2FAAAAAAAAAAAAAAAAAAAAAIYAK0Uh71ZVswXYmTZHycyPLUo2eczOlMgzn-CSzeIJ%2Fimg.jpg%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1756652399%26allow_ip%3D%26allow_referer%3D%26signature%3DIzr4lgn0F9nwLO%252Fim7e0%252FGwk%252Bic%253D)

#### Directives

- AREA : Assembly Block을 만들고 속성을 지정
  - 링커가 다루는 최소의 단위로 AREA 단위로 주소 할당
- AREA의 속성 (여러개 사용 가능)
  - ALIGN : default 4 byte
  - CODE : contain machine instruction
  - DATA : contain data
  - COMDEF : common section definition
  - COMMON : common data section
  - NOINIT : unitialized data section or initialized to zero
  - READYONLY : Read only section
  - READWRITE : Read/Write section
- ENTRY : AREA에 ENTRY가 있는 경우, AREA에서 수행되어야 할 첫 번째 위치
- CODE32/CODE16 : 32bit ARM code cod / 16bit Thumb code
- END : Assembly로 짜여진 file의 끝을 의미

#### Label : BEGIN, THUMB, LOOP, TEXT

- 코딩하는 사림이 붙인 이름표
- label이 symbol이 됨

#### Instruction

#### COMMENT (주석) : `;`을 통해 달 수 있음

#### 실제 helloworl.c 를 어셈블리로 컴파일한 결과 분석

> 위는 간단한 예제였고, 실제 컴파일된 어셈블리를 조금 더 복잡하긴 함!

## 초간단 Assembly와 Reverse Engineering

#### ARM Assembly 특징

- 기본 뼈대 명령어 뒤 조건이나 덧붙임 명령어를 더 붙여서 활용하는 컨셉
- 명령어 Rd, ... : 대부분의 명령어는 가장 앞쪽에 destination(오른쪽 연산 결과 저장)이 옴
  - STR은 대상이 거꾸로 되어있음
- OP 코드(instruction) + Operand 형태

#### Branch 명령어

실행 번지 변경 (주로 함수 호출에 사용)

- B 주소 : 주소로 점프
- BL 주소 : 주소로 점프 + R14(LR)에 주소 저장
- BX 주소 : 주소로 점프 + ARM 모드 toggle (ARM mode <-> Thumb mode)
- BLX 주소 : 주소로 점프 + R14(LR)에 주소 저장 + ARM 모드 toggle (ARM mode <-> Thumb mode)

#### Data Processing

여기에서 V는 레지스터 또는 값을 의미함

- MOV Rd, V : Rd = V
- MVN Rd, V : Rd = -V
- ADD Rd, V1, V2 : Rd = V1 + V2
- SUB Rd, V1, V2 : Rd = V1 - V2
- RSB Rd, V1, V2 : Rd = V2 - V1
- AnD Rd, V1, V2 : Rd = V1 & V2
- BIC Rd, V1, V2 : Rd = V1 & ~V2 (V2의 1이 들어있는 field만 0으로 set)
- CMP V1, V2 : V1 > V2 결과를 CPSR에 저장

#### Load/Store

[V]는 해당 주소(V)를 참조한 값을 의미함
!는 Write back 연산자로 연산후 write 임

> R15(PC)가 Rn이 될때는 ! 연산자 사용 불가능 (base register가 마음대로 업데이트 되는 일 방지)

- LDR Rd, [Rn] : Rd = \*Rn
- LDR Rd, 주소값(숫자) : Rd = \*주소값
- LDR Rd, [Rn, offset] : Rd = \*(Rn + offset)
- LDR Rd, [Rn, offset]! : Rd = \*(Rn), Rn = Rn + offset
- LDR Rd, [Rn], offset : Rd = \*(Rn), Rn = Rn + offset (위와 동일하고 ! 없어도 됨)

STR의 경우, LDR 대신 STR로만 변경

- STR Rd, [Rn, offset] : \*(Rn + offset) = Rd (STR의 경우 왼쪽이 결과, 오른쪽에 실제 destination이 됨)

사이즈별 Load/Store

- LDRB/STRB : byte
- LDRH/STRH : half word
- LDRSB/STRSB : signed byte
- LDRSH/STRSH : signed halfword

#### 의사 명령어(가짜 명령어) : 편의를 위해 존재하지만, 실제로 처리는 다른 방법으로 됨

- LDR, ADR이 의사 명령어에 해당함
- 예. LDR Rd, Label
  - Label을 [PC, #offset] 형태로 만들어 `LDR Rd, =[PC + offset]`형태로 바뀜
  - LDR Rd, 실제주소 라고 했을 때보다 한 단계 더 거치게 됨
- 의사명령어를 사용하면 메모리 접근 시간과 명령어 추가가 되기 때문에 꼭 필요할때만 사용해야함
- ADR Rd, Label 은 [PC, #offset] 이 아닌 LDR Rd, 실제주소 형태로 변환됨

#### SWAP 명령 : 메모리 영역의 값과 register의 값을 swap

> swap 명령은 인터럽트가 걸리지 않는 특성이 있어, semaphore등을 구현할 때 사용

- SWP R0, R1, [R2] : R0에 R2 값을 넣고, R2에 R1 값을 넣어줌
- 엄밀히 말하면, 값을 바꾼다기 보다 R0로 값을 뺏어오는 것임

#### PSR 명령

CPSR과 SPSR 를 일반 Register 에 값을 서로 복사해오는 명령어

- MRS Rd, CPSR : Rd = CPSR
- MSR CPSR, Rn : CPSR = Rn

> S는 PSR을 의미, R은 일반 레지스터를 의미, 방향은 <- 이라고 생각하면 MRS, MSR의 구분이 좀 쉬워짐

flag와 condition을 따로 떼어내는것도 가능함

- CPSR_c : control
- CPSR_f : flag
- CPSR_cf : control + flag

#### SWI 명령

커널이 SVC mode, 일반 Application이 user mode일 때, 일반 Application이 kernel게게 무언가를 부탁할 때 사용

- SWI 0x11 : Software Interrupt Handler에서 0x11번째 케이스 호출

#### DCD directives : 메모리역역의 data 할당 및 초기화

- DCB : 1 바이트 데이터 할당 및 초기화
- DCW : 2 바이트 데이터 할당 및 초기화
- DCD : 4 바이트 데이터 할당 및 초기화, = 로도 축약해서 쓰임
- DCQ : 8 바이트 데이터 할당 및 초기화

예) LDR R0, =0xFFFFFFFF : 0xFFFFFFFF을 메모리영역 저장하고, 그 값을 참조한 값을 R0에 Load

> ARM이 32bit 머신인데 Opcode를 제외하면, 32bit의 주소값을 하나의 명령에 담을 수 없기 때문에 따로 값을 저장해놓고 그 값을 사용해야함

#### EXPORT, IMPORT directive

- EXPORT : Assembly에서 사용된 symbol을 외부에서 사용가능하게 만들어줌
- IMPROT : 어딘가에서 EXPORT한 symbol을 가져다 쓸 때 사용

#### FIELD와 MAP : C의 구조체 값은 역할

```
MAP StudentInfo, SizeOfStudent
  FIELD StudentID, 4    ; 4바이트 정수형 학생 ID
  FIELD NameOffset, 12  ; 12바이트 문자열(이름)
  FIELD Score, 2        ; 2바이트 정수형 점수
ENDMAP
```

이건 구조만 정의한거고 인스턴스화가 된건 아님, 사용시에는 아래 처럼 인스턴스화 후 사용해야함

```
; StudentInfo 구조체에 대한 메모리 공간을 할당
StudentData DS StudentInfo ; StudentInfo의 크기만큼 메모리 할당

; StudentData의 'Score' 필드에 85점을 저장하는 예시
MOV R0, 85
MOV [StudentData + Score], R0
```

#### 덧붙임 명렁어들

- ^ (캐럿) : SPSR에 저장된 값(이전 상태 레지스터 값)을 CPSR로 복원
  - 주로 인터럽트나 예외 처리 후 원래 상태로 돌아갈 때 사용
  - 예. LDMFD SP!, {R0-R12, LR, PC}^
  - 요약: "이전 상태로 완벽하게 돌아가!"
- ! (느낌표) - 데이터 전송(로드/스토어) 후, 베이스 레지스터의 주소를 이동한 데이터의 크기만큼 자동으로 조정
  - 이 접미사는 베이스 레지스터(PC)의 주소 값을 자동으로 갱신할 때 사용
  - 예시. STMDB SP!, {R0-R3}
  - 요약: "데이터 옮기고, 주소도 알아서 바꿔줘!"
- S (Suffix) : 연산 결과에 따라 N, Z, C, V 플래그를 설정하여, 다음 명령어에서 조건 분기(branch)
  - 연산 후 CPSR의 조건 플래그를 업데이트할 때 사용
  - 예. ADDS R0, R1, R2
  - 요약: "계산하고, 결과에 따라 플래그를 바꿔줘!
