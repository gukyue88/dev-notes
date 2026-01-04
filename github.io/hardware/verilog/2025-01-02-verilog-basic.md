---
layout: single
title: "[verilog] Digital Logic 및 Verilog 기초"
categories: hardware/verilog
published: false
---

## Combinational Logic

#### Combinatioanl Logic의 개념

- 아웃풋이 인풋과 게이트의 조합으로만 결정됨
- Sequential Logic의 경우, 피드백(이전 아웃풋값이 영향을 미침)이 들어가게 됨.

#### Gate Logic의 종류 상세 설명

- not : `assign OUT = ~A;`, `assign OUT = !A;`
- and : `assign OUT = A & B;`
- or : `assign OUT = A | B;`
- nand : `assign OUT = ~(A & B);`
- nor : `assign OUT = ~(A | B);`
- xor : `assign OUT = A ^ B;`, `assign OUR = ~(A & B) & (A | B)`
- xnor : `assign OUT = ~(A ^ B);`, `assign OUT = ~(A | B) | (A & B)`

> 실제 회로에서는 nand와 nor가 간단하고, and와 or는 nand와 nor에 not을 붙이는 형태로 제작됨. 그래서 보통 기본이 되는 단위가 nand가 됨.

#### Boolean Algebra

> 같은 동작을하는 로직은 게이트를 줄이는 형태로 최적하해야함. 수식을 통해 최적화 : Boolean Algebra, 맵을 그려 최적화 : 카르노맵

- 교환법칙 : A | B == B | A, A & B == B & A
- 결합법칙 : (A | B) | C == A | (B | C), (A & B) & C == A & (B & C)
- 분배법칙 : A & (B | C) == (A & B) | (A & C), A | (B & C) == (A | B) & (A | C)
- 합의법칙 : A == A & 1 == A | 0, 1 == A | ~A == 1 | A, 0 == A & ~A == A & 0
- 항등법칙 : A == A | A == A & A == (A & ~B) | (A & B) == (A | B) & (A | ~B)
- 드모르간법칙 : ~(A | B) == ~A & ~B, ~(A & B) == ~A | ~B

#### Karnaugh Map (카르노맵)

- 4개 이상의 인풋이 들어올 경우 좀 더 진가가 드러남
- 인풋에 대한 아웃풋 표현법
  - `F(A,B,C) = Σm(0,3,4,7) +  Σd(2,6)`
    - 2진수로 나타낸 0, 3, 4, 7이 input일 때 output이 1이 됨.
    - 2진수로 나타낸 2, 6이 input일 때, output은 dont care
      - 위 방법은 sop라는 방법이고, pos라는 방법의 경우 output이 0이 되는걸 표시함. sop가 많이 쓰임
- 카노맵 사용법
  - 00, 01, 11, 10 처럼 bit이 한 번만 바뀌도록 표를 그림
  - 같은 값끼리 묶는 사각형을 그리되 최대한 크게 그림 (dont care는 최대한 묶어서 사용)
  - 사각형을 기준으로 변하는 input을 보고 식을 작성

#### Combinatioanl Logic의 대표 예시

- half-adder : A + B = Sum + Carry
- full-adder : A + B + CarryIn = Sum + Carry
- decoder : 2자리 인풋(0,1,2,3) => 4개의 아웃풋 중 하나만 1
- mux : 인풋 중에서 선택하여 아웃풋으로 내보냄

#### Additional Logic

- tri-state buffer : output에 0, 1 외 z값을 가지는것, z는 high impedance, 쉽게 말하면 output이 z라고 하면 output의 선이 끊어진거임.
  - 자주 사용하지는 않고, 어쩔 수 없이 양방향으로 인풋, 아웃풋이 바꿔야하는 상황에서 쇼트방지를 위해 사용
- buffer : 값이 바뀌지는 않고, 죽어가는 신호를 다시 살려주는 용도로 사용함. 값이 멀리갈때 중간에 하나씩 배치
  - not 게이트 2개를 직렬로 연결하여 작성

---

## Sequential Logic

#### Sequential Logic의 개념

- 현재 output이 미래의 output에 영향을 미치는 로직
- 보통은 enable, reset 제어가 같이 사용됨
  - enable : 과거의 output을 반영할지 말지 결정하는 핀
  - reset : 처음 output 값을 결정

#### Latch 구조 및 동작

- SR Latch : output이 intput으로 서로 들어감
- 이 앞에 게이트와 enable을 넣으면 Gated SR Latch를 만들 수 있음.

#### D Flip-Flop의 구조 및 동작, 활용

- Flip-Flop : latch의 업그레이드 버전, 클락이 high일 떄도 값을 유지, rising 또는 falling edge 일때 output을 변경
  - Latch : enable 신호에 반응
  - Flip-Flop : clock에 의해 작동
- D Flip-Flop : verilog에서 99% 정도가 D Flip-Flop으로 이루어짐
  - clock이 rising 할때 D(input)이 Q(output)에 반영됨

#### Clock

- Clock을 결정짓는 요소
  - frequency : 1초에 몇 번 올라갔다 내려갔다 하는가?
  - duty cycle : 한 주기에서 1과 0의 비율
  - jitter : 오차 (Real clock은 주기보다 앞이나 뒤쪽으로 클락이 작동함.)
- Ideal Clock vs Real Clock
  - Ideal Clock : 올라가고 내려가는 부분이 직각임. 정해진 시간에 따라 올라가고 내려가게 됨
  - Real Clock : 올라가고 내려갈때 시간이 걸리고 기울기를 가지게 됨. 정해진 시간에서 오차를 좀 가짐.

> 클락 frequency를 결정하는 방법 : frequency > gate delay + jitter + setup time + hold time + skew

- gate delay : 피드백이 gate를 지나오면서 걸리는 시간
- jitter : 주기 오차
- setup time : rising 이전에 데이터를 미리 준비해야하는 시간
- hold time : rising 이후에도 데이터를 유지해야 하는 시간
- skew : clock source로 부터 각 flip flop의 거리가 다르기 때문에 생기는 각 flop flop마다의 clock 차이
  - 물리적인 거리가 가까운것도 길을 멀리 돌아가서 길의 길이를 맞춰 skew를 줄이는 방법도 존재함

보통 현업에서는 frequency가 먼저 정해지고, 그거에 맞춰 다른걸 줄이는 노력을고함.

- 디지털 로직에서는 보통 gate delay를 줄이는 작업을 많이 하게 되고 이게 디지털 설계 엔지니어의 역량이 됨
- 아날로그 로직에서는 jitter나 setup time, hold time 등을 잘 잡아주는게 아날로그 설계 엔지니어의 역량이 됨

> 시뮬레이션의 경우, gate delay, jitter등을 제외하고 ideal 상황에서 돌림

#### reset

종류 : async_resetn, async_reset, sync_resetn, sync_reset

- async vs sync
  - async : 클락과 무관한 reset
  - sync : 클락에 동기화된 reset
- active low vs active high
  - active low : 신호의 값이 0일 때 동작, resetn으로 표현
  - active high : 신호의 값이 1일 때 동작, reset으로 표현

#### Sequential Logic 의 대표 예시

- shift register : DFF(D flip flop)을 여러개 연결하여, 특정 clock 이후 원하는 값을 얻을 때 사용
- counter

---

## Verilog 기본 문법 및 개념

#### Verilog에서 print 하는 법

`$display()` : C언어의 printf랑 비슷함
`$monitor()` :변수가 변할때만 활성화됨

```verilog
`timescale 1ns/1ns
module say_hello;
  int test = 0;

  initial begin
    #10
    $display("[%d][display] test : %d", $time, test++);
    #10
    $display("[%d][display] test : %d", $time, test++);
    $display("[%d][display] test : %d", $time, test++);
    $finish
  end

  initial begin
    $monitor("[monitor] test : %d", test);
  end

endmodule
```

- `timescale 1ns/1ns` : 시간 단위 설정
- `#10` : 10 ns를 흘려보내라. (시간 단위가 1ns/1ns로 설정되어 있기 떄문에, #10이 10ns를 의미하게 됨)
- `module - endmodule` : 모듈의 시작과 끝
- `initial begin - end` : 프로시저 블록으로 블락 안에 있는 코드들은 sequential 하게 진행되고, 서로 다른 블락들은 parallel하게 진행됨.
- $time : 빌트인 변수 느낌, 현재 시간을 찍게 됨
- `$finish` : 시뮬레이터 종료 명령

#### Verilog에서 waveform 만드는 법

**디버그 방법**

- `$dumpfile` : waveform 저장 출력 파일 경로 지정 (\*.vcd 확장자 사용)
- `$dumpvars` : waveform dump의 범위 지정

```verilog
module dump_example;
  reg a, b;

  initial begin
    a=0; b=0;
    #10 a=1; b=0;
    #10 a=0; b=1;
    #10 a=1; b=1;
    #10
    $finish
  end

  initial begin
    $dumpfile("dump_example.vcd");
    $dumpbars(1, dump_example);
    $display("test waveform dump");
  end
endmodule
```

- `$dumpvars(1, dump_example)` : dump_example 모듈 안에 있는 모듈만 보겠다, 앞에 숫자는 계층을 나타내고 2이면 sub module 1개까지 추가되서 나옴, 0을 넣으면 해당 모듈 의 모든 sub module을 전부 보겠다는 의미가 됨
- 마지막에 `$10`은 시뮬레이션이 바로 종료되면 마지막 부분이 모니터링이 되지 않음

#### Data type

- Value Set
  - 0 : logic 0, false
  - 1 : logic 1, true
  - x : unknown
  - z : high-impedence, open connection
- Format
  - 2진수 : `'b1010`, 출력시 `%b` 사용
  - 10진수 : `'d10` 또는 `10`(생략가능), 출력시 `%d` 사용
  - 16진수, `'hA`, 출력시 `%h` 사용
    - 앞에 비트 수도 붙일 수 있음. 4비트라면? `4'b1010` 또는 `4'd10` 또는 `4'hA`
  - 모든 비트를 의미할 떈, '를 사용 -`a = '0` : a의 모든 비트에 0을 넣음 -`a = '1` : a의 모든 비트에 1을 넣음 -`a = 'x` : a의 모든 비트에 x을 넣음 -`a = 'z` : a의 모든 비트에 z을 넣음
- Nets : wire(전선), tri(tri state 연결), supply0(전원 0), supply1(전원 1)
- Variables : reg, int, real, time
- Vectors : nets나 variables는 1bit임. `wire [7:0] wire_8bit` 이라고 하면 8비트 wire가 만들어지는 것임.
- Array : ``reg [3:0] array1D [7:0]` 이라고 하면 4비트 reg를 8개 가진 1차원 배열, `reg [3:0] array2D [1:0][7:0]` 이라고 하면 4비트 reg를 [2][8]로 갖는 2차원 배열
- Vectors vs Array : Vectors에는 한 번에 값이 들어감. Array는 각각 값을 넣어줘야함.

#### Module & Port & Instantiation

- module : 설계 단위, 보통 하나의 모듈은 하나의 파일로 만들고, 이름을 똑같이 맞추어줌
- ports : input&output, module간 인터페이스, 보통 벡터형태로 표현함 (`output[3:0] output_a`)

```
module DESIGN_A (
  input clk
  input rstn,
  input [3:0] in_a,
  input [7:0] in_b,
  output[3:0] output_a
  output[7:0] output_b
);
  // Code for DESIGN_A module
endmodule
```

- instantiation : sub-module을 호출할 떄 사용, instance 화 하는 단계

> 하위모듈은 class이고, 상위모듈에서 instance화하고 이름을 붙이는 과정처럼 보임. 같은 class로 여러 instance도 생성 가능

상워 모듈인 DESIGN_C에서 하위 모듈인 DESIGN_C_A, DESIGN_C_B를 인스턴스화 하는 코드

```
module DESIGN_C (
  input clk,
  input rstn,

  input [3:0] in_a;
  input [3:0] in_b;
  input [7:0] in_c;

  output[3:0] out_a;
  output[3:0] out_b;
  output[7:0] out_c;
);

  DESIGN_CA u_C_A_1(clk, rstn, in_a, out_a);

  DESIGN_C_A u_C_A_2(
      .clk (clk),
      .rstn (rstn),
      .in (in_b),
      .out (out_b)
  );

  DESIGN_C_B u_C_B(
      .clk (clk),
      .rstn (rstn),
      .in (in_c),
      .out (out_c)
  );

endmodule
```

#### parameter 및 define

> 같은 함수 인데, bit width가 달라지는 경해 새롭게 모듈을 설계하는 일을 방지하기 위해 사용

컴파일시, 상수화가 먼저 진행됨(define 같은 전처리 느낌인듯)

```
module PRAMETER_EX #(
  parametyer N=4
) (
  input clk,
  input rstn,
  input [N-1:0] in,
  output [N-1:0] out
);
  localparam K=32; // 이건 상위 모듈에서 받지 않고, 이 모듈에서 고정으로 사용할 paramter임
  wire [N-1:0] a;
  ...
endmodule
```

define : c의 define과 같음. 너무 많으면 헷갈려서 최대한 자제를 권유한다고 함

```
`define SEQ_LOGIC

`ifdef SEQ_LOGIC
  ...
`endif

`ifndef SEQ_LOGIC
  ...
`else
  ...
`enqdif
```

#### Operator

Bit-part : 일부 bit만 취할 떄 사용

```
wire [3:0] a;
wire [1:0] b, c;

b = a[3:2];
c = a[1:0];
```

Concatenation : bit을 합칠 때 사용

```
a = {b,c}; // b랑 c를 합쳐 a를 만듦
a = {2{b},c} // b를 2번 받복한것과 c를 합쳐 a를 만듦
```

Conditional

```
a = b ? c: d;
```

Arithmetic(산술연산) : 곱셈, 나눗셈은 회로가 많이 들어가기 때문에 필요한 상황에만 사용하기! (나눗셈과 %는 거의 아예 안쓰면 좋겠다고 함.)

```
a+b;
a-b;
a*b;
a/b;
a%b;
a**b;
```

Relationl

```
a<b;
a>b;
a<=b;
a>=b;
```

Equality : `===`, `!==`는 0, 1, z, x를 모두 포함하여 계산

보통은 z, x를 설계에 넣지 않기 때문에, `==`, `!=`를 많이 사용함

```
a===b;
a!==b;
a==b;
a!=b;
```

Logical

```
a&&b;
a||b;
!a;
```

Bitwise

```
~a;
a&b;
a|b;
a^b;
```

Unary

```
&a; // a안의 모든 비트를 &
|a; // a안의 모든 비트를 |
^a; // a안의 모든 비트를 ^
```

Shift

```
a>>3;
a<<3;
a>>>3; // sign 부호까지 같이 끌고 내려옴
a<<<3; // 이거는 왼쪽으로 미는거라 큰 차이 없음
```

---

## Verilog Modeling

모델링엔 3가지 타입의 모델링 방법이 있음

- Gate-level Modeling : 게이트로 한땀 한땀 표시, 잘 쓰지 않음
- Data flow Modeling : 와이어가 어떻게 연결되었는지를 묘사
- Behavior Modeling : 어떤 동작을 하는지 묘사

#### Gate-level Modeling

```
module mux_gatelevel(
  input a, b
  input A, B, C, D,
  output Q
);

  wire a_not, b_not;
  wire A_pick, B_pick, C_pick, D_pick;

  not(a_not, a);
  not(b_not, b);

  and(A_pick, a_not, b_not, A);
  and(B_pick, a, b_not, B);
  and(C_pick, a_not, b, C);
  and(D_pick, a, b, D);

  or(Q, A_pick, B_pick, C_pick, D_pick);

endmodule
```

- gate는 첫 번째가 아웃풋이고, 뒤에 가변인자로 input을 받음

#### Data flow Modeling

assign 이 있어야 data flow라고 부를 수 있음

- 게이트 레벨보단 사람들이 이휘하기 쉽고, 더 많이 사용됨.
- 데이터가 흘러가는게 눈에 잘보임(?)
- assign 은 반드시 wire 또는 net에만 써야함.

```
module mux_dataflow(
  input a, b,
  input A, B, C, D,
  output Q
);
  wire sel_0, sel_1, sel_2, sel_3;

  assign sel_0 = ~a & ~b
  assign sel_1 =  a & ~b
  assign sel_2 = ~a &  b
  assign sel_3 =  a &  b

  assign Q = sel_0 ? A :
             sel_1 ? B :
             sel_2 ? C :
             sel_3 ? D : 0

endmodule
```

#### Behavior Modeling

- procedure 블락 안에 동작을 묘사
  - procedure 블락 : begin 부터 end까지 묘사되는 블락
- 보통은 always 블록을 많이 사용함.

동작을 이해하고, 동작 중심으로 코드를 작성함

```
module mux_behavior (
  input a, b,
  input A, B, C, D,
  output Q
);
  reg Q;

  always @* begin
    case ({b,a})
      `b00 : Q=A;
      `b01 : Q=B;
      `b10 : Q=C;
      `b11 : Q=D;
      default : Q=0;
    endcase
  end

endmodule
```

- `always @*` : always가 언제 반응할건지를 묘사, `*`은 변화가 있을 때 무조건 반응한다. `forceEdge clock`(?)이면 clock에 반응하도록함

#### Stimulus

- 베릴로그는 디자인 + 테스트 벤치로 돌아감
- 테스트 벤치가 input을 흔들어서 output이 원하는 대로 나오는걸 보는 과정을 stimulus(자극)라고 함

```
module mux_testbench;
  reg a, b;
  reg A, B, C, D;
  wire Q;

  mux_dataflow u_mux_dataflow(a, b, A, B, C, D, Q);

  initial begin
    A='0; B='1; C='1; D='0;

    a=`0; b=`0;
    #10 a='1; b='0;
    #10 a='0; b='1;
    #10 a='1; b='1;
  end

  initial begin
    $monitor("dataflow : a %d, b %d, Q %d", a, b, Q);
  end

endmodule
```

#### Behaviro Modeling의 다양한 구성

- TODO : 일단 나중에

#### Behavior Modeling 연습문제

- TODO : 일단 나중에

---

## FSM

#### FSM의 쓰임새 및 정의

- FSM : Finite State Machine (유한한 state를 가진 체계)
- 시퀀셜 로직의 경우, output은 현재 상태에 의해서도 결정되는데, 이 상태의 개수가 무한하지 않고 유한한 개수를 가질 떄, 이 시스템을 FSM 이라고 부름
- status를 중점으로 state 다이어그램을 그리고 각 state가 어떨때 변경되는지 확인

#### moore vs Mealy machine

FSM을 구현하는 방법

- Moore machine
  - 현재 상태에 의해서만 output이 바뀜(input value에 의해 당장 output이 바뀌지 않음)
  - output은 클락 엣지에 의해서만 동장
  - mealy machine 보다 안전함 (input이 async 하더라도, output이 sync 동작이기 때문에)
  - 실무에서는 이걸 더 많이 씀, 또 섞어서 쓰기도 함
- Mealy machine
  - output이 현재 상태와 input 모두에 영향을 받음
  - moore machine 보다 빠른 반응 속도 (클락 엣지를 기다릴 필요 없음)
  - moore machine 대비 상태를 좀 더 적게 쓸 수 있음. (상태 대신 input으로 넣으면 되기 때문에)

#### FSM Coding 하는 법

next state와 current state를 따로 나누는 스타일의 코딩

```
// 0,1,2 대신 S0, S1, S2라는 이름을 사용하기 위해 선언
localparam [1:0] S0=0, S1=1, S2=2;

// clk 엣지마다 next_state를 cur_state에 반영하겠다
always @(posedge clk, negedge rstn)
if (!rstn) curr_state_ <= 0;
else       curr_state <= next_state;

// state 다이어그램에 따른 next_state 정의
always @* begin
  next_state = curr_state;
  case(curr_state)
    S0:                   next_state = S1;
    S1: if      (go==1)   next_state = S2;
    S2: if      (back==1) next_state = S1;
        else if (done==1) next_state = S0;
  endcase
end

// state에 따른 status(output) 정의
always @* begin
  status = 0;
  case (curr_state)
  S0: status = 0;
  S1: status = 1;
  S2: status = 2;
end
```

next state와 current state를 따로 나누지 않는 스타일의 코딩

- next_state 대신 그냥 바로 curr_state를 변경함

```
localparam [1:0] S0=0, S1=1, S2=2;

always @(posedge clk, negedge rstn)
if (!rstn) curr_state_ <= 0;
else
  case(curr_state)
    S0:                   curr_state <= S1;
    S1: if      (go==1)   curr_state <= S2;
    S2: if      (back==1) curr_state <= S1;
        else if (done==1) curr_state <= S0;
  endcase
end

// state에 따른 status(output) 정의
always @* begin
  status = 0;
  case (curr_state)
  S0: status = 0;
  S1: status = 1;
  S2: status = 2;
end
```

#### FSM 설계 연습문제

- 신호등 예제 (초가 지나면 신호가 바뀌게, 특정 버튼을 누르면 바로 신호가 바뀌게)

#### FSM 실제 사용 예

- 그냥 순환만 하거나, 인풋이 거의 없을 때는 FSM이 아닌 카운팅을 하면 됨. (비효율적으로 설계가 됨)

---

## Testbench의 개념 및 활용

#### Testbench 란

- DUT (Design under test) : 반도체 회로 디자인
- TB (TestBench)
  - DUT를 검증하는 환경을 제공하는 것
  - 검증 모델로서 DUT 만큼 중요하고, DUT 만큼 복잡함
  - 최근에는 TB로 System-Verilog가 추천됨
  - DUT의 clock, reset, logic등을 input으로 넣고, monitor와 checker를 통한 검증 구조

#### fork-join

fork로 parrallel한 구문들을 실행해로고

- join : 구문들이 모두 끝나야 다음으로 넘어감
- join_any : 구문들 중 하나라도 끝나면 다음으로 넘어감
- join_none : 바로 다음으로 넘어감

> join은 verilog이고, join_any와 join_none은 system verilog임

#### event-wait

- event
  - testbench를 위한 다른 타입의 변수
  - 고정된 사간 이후 이벤트를 발생 시킴
  - `#20 -> event_a;` : 20초 후, event_a 발생
  - `@(event_a); : event_a 감지되기까지 기다림
  - event 대신 쓸 수 있는 것
    - `@(negedge rstn)` : rstn이 일어난 다음
    - `repeat(5) @(posedge clk);`posedge 5클락 지난 후
- wait
  - `wait(a==1)` : a가 1이 될 때까지 기다림

#### force-release

원하는 state까지 올리시데 시간이 오래걸릴 때 주로 사용함

- force : 그냥 다 무시하고, 강제로 해당 값을 세팅함
- release : force로 할당된 값을 해제하고, 회로에 의해 자연스레 값이 세팅되도록 함

> force 한 값은 release로 풀어줘야 함

#### Verilog system function

- File I/O
  - $fopen : 파일 열기
  - $fscanf : 파일에서 값을 읽어와 변수에 할당 (fread 개념)
  - $fdisplay : 변수의 값을 파일에 씀 (fwrite 개념)
  - $fclose : 파일 닫기

> 보통 파일에 테스트 케이스를 저장해놓고 load 해서 사용함

- $random : 랜덤 값 생성
- $readmemh, $readmemb
  - 파일의 텍스트나 데이타를 읽어올 때 쓰임
  - $fscanf와 비슷하나, 더 빠르고 간단함? (fopen, fscanf, fclose 없이 그냥 하나로 다 돌아감)

---

## Task & Function

테스트 벤치를 짜는 스킬임

#### Task 문법 및 사용

- C에서 함수 역할
- 테스트 시퀀스를 짜야하는 테스트 벤치쪽에서 많이 쓰임
- input/output을 가질 수 있고, task, endtask로 감싼다

```
task my_task(
  input [3:0] in_a, in_b,
  output[3:0] out_a, out_b
);
begin
  out_a = in_a + 1;
  #1 out_b = int_b + 1;

  @(posedge clk);
  out_a = out_a + 1;
  out_b = out_b + 1;

  @(posedge clk);
  out_a = out_a + 1;
  out_b = out_b + 1;
end
endtask
```

- task 는 time 개념을 가질 수 있음. 예) #10, @(posedge clk) 등
- task가 실행되는 동안에 output의 상태 변화를 바깥쪽에서 실시간으로는 볼 수는 없고, output만 나오게 됨.
- 만약 꼭 알아야한다고 하면 task 안쪽에서 찍어야함. 바깥쪽에서는 보지 못함.
- 하나의 task는 다른 task를 부를 수 있음.
- task는 기본적으로 static임. -> 변수가 변하면 그걸 유지하고 있음.
- automatic 키워드를 같이 사용하면, 각 task가 독립적으로 동작함. 변수를 공유하지 않음
- task에서는 blocking assignment(`=`)와 non-blocking assignment(`<=`)를 모두 사용 가능함

#### Functtion 문법 및 사용

- procedure 블락 뿐만 아니라, data-flow 에서도 사용할 수 있는 서브루틴임
- task의 경우는 procedure 블락에서만 부를 수 있음.
- function의 이름 자체가 output으로 사용됨
- time 컨셉을 가질 수 없음
- automatic, static 두 가지로 동작 가능
- function는 회로가 구성되는 거임. DUT에서 복잡한 회로를 대체용으로 쓰임

```
function [4:0] my_function (
  input [3:0] in_a, in_b
);
  reg [4:0] sum;
  reg [4:0] shift;

  sum = in_a + in_b;
  my_function = sum << 1;

endfunction
```
