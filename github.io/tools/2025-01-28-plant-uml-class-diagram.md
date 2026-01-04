---
layout: single
title: "[Tools] PlantUML 클래스 다이어그램"
categories: tools
---

## 용어정리

- uml : SW 개념을 다이어그램으로 표현하기위한 표준
- 클래스 다이어그램 : 클래스간 관계를 표현하기 위한 uml의 한 종류
- plant uml : text를 기반으로 uml 그릴 수 있음.
  - vscode plantuml 익스텐션 활용
  - java, graphviz 가 설치되어 있어야함.
  - .puml 확장자를 사용함.

## PlantUML 클래스 다이어그램

> 클래스 다이래그램은 요소(Element)와 관계(relation)으로 이루어짐

#### 간단 예제

```
@startuml
package Group1 {
	class Car {
		- String brand
		- String model
		- Car car
		+ void start()
		+ void stop()
	}

	class Engine {
		- int horsepower
		- int cylinders
		+ void start()
		+ void stop()
		+ void drive(Car car)
	}
}

Engine --* Car : contains in

note top of Engine : Engine is included in Car

@enduml
```

#### 기본 포맷

```
@startuml
// uml 내용 작성
@enduml
```

#### 주요 요소

```
class
interface
struct
enum
```

#### 접근 제한자

- `-` : private
- `+` : public
- `#` : protected

#### 관계

> Element 사이에 관계를 표현할 때 사용

```
# A --|> B : Extension
- A가 B를 상속 받음
- 실선과 빈삼각형으로 표시

# A ..|> B : Implementation
- A가 인터페이스 B를 실체화함
- 점선과 빈삼각형으로 표시

# A --* B : Composition
- A가 B에 속해있는 관계, A안에 B타입의 멤버변수가 있고, 라이프사이클을 같이함.
- 실선과 속이찬마름모로 표시

# A --o B : Aggregation
- 정의가 애매해서 uml2.0에서는 사라짐.
- A가 B에 속해있으나, B가 없어진다고 해서 A가 없어지지는 않는 관계(?)
- 예) 선수 --o 팀, 이건 다른 걸로 표현이 가능함.
- 실선과 빈마름모로 표시

# A -- B : 2 ways Association
- A와 B가 서로를 참조하는 멤버변수로 가지고 있는 관계
- 실선으로 표시

# A --> B : one way Association
- A만 B를 참조하는 멤버변수를 갖는 관계
- 실선과 화살표로 표시

# A ..> B : 의존관계 (Dependency)
- A가 B를 멤버변수로 가지고 있지는 않고, 지역변수나 파라미터&리턴값으로 사용하는 경우
- 점선과 화살표로 표시
```

#### 관계 레이블

> 1 대 다 매칭, contains 등 관계선상에 label을 넣을 때 사용

```
A "slabel" -- "elabel" B : mlabel
```

- slabel : 시작 레이블
- elabel : 끝 레이블
- mlabel : 중간 레이블

#### 노트

> 메모를 하고 싶을 떄 사용

```
class A {}
class B {}
class C {}

note top of A : this is a note top of A
note bottom of A : this is a note bottom of A
note left of A : this is a note left of A
note right of A : this is a note right of A

note top of B
note can be written
in several lines
like this!
end note

note top of C
It's possible to use few HTML tags.
<font color="#AAAAAA">test</font>
<size:18>size</size>
end note
```

#### package

> 그룹을 표시할 때 사용

```
package 패키지명 {
	// 패키지 안에 내용
}
```
