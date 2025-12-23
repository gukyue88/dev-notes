# Chapter 1. LangGraph 핵심 모듈

## 1. LangGraph에서 자주 등장하는 파이썬 문법 (TypedDict, Annotated, Reducer)

#### TypedDict

- dict와 비슷한데, 약간의 차이가 있음
  - 타입 검사 : dict는 타입 검사가 없는 반면, TypedDict는 정적 타입 검사를 제공함
  - 키와 갑의 타임 : dict는 키와 값의 타입을 일반적으로 지정, TypedDict는 각 키에 대한 구체적인 타입을 지정
  - 유연성 : dict는 런타임에 키를 추가/제거 가능, TypedDict는 불가
- TypedDict가 dict 대신 사용되는 이유
  - 타입 안정성 : 버그 미리 방지
  - 코드 가독성 : 딕셔너리 구조를 명확하게 정의하여 가독성 향상
  - IDE 지원 : 자동 완성 및 타입 힌트 지원
  - 문서화 : 코드 자체가 문서의 역할을 하여 딕셔너리의 구조를 명확히 보여줌

```python
from typing import Dict, TypedDict

class Person(TypedDict):
    name: str
    age: int
    job: str

typed_dict: Person = {"name": "teddy", "age": 25, "job": "designer"}
```

#### Annotated

- Annotated를 사용하는 주요 이유 : 타입 힌트에 추가 정보 제공 + 문서화 + 유효성 검사 + 프레임워크 지원(langGraph에서 특별한 동작 정의)
- Annotated는 Python typing 모듈에서 제공하는 특별한 타입 힌트로, 기존 타입에 메타데이터를 추가할 수 있게 해줌
- `variable: Annotated[Type, metadata1, metadata2, ...]`

```python
from typing import Annotated

name: Annotated[str, "사용자 이름"]
age: Annoatated[int, "사용자 나이 (0-150)"]
```

Pydantic과 함께 사용

```python
from typing import Annotated, List
from pydantic import Field, BaseModel, ValidationError

# Field의 ...은 필수 항목이라는 의미임
class Employee(BaseModel):
    id: Annotated[int, Field(.., description="직원 ID")]
    name: Annotated[str, Field(..., min_length=3, max_length=50, description="이름"]
    age: Annotated[int, Field(gt=18, lt=65, description="나이 (19-64세)")]
    salary: Annotated[float, Field(gt=0, lt=10000, description="연봉 (단위: 만원, 최대 10억ㅇ)")]
    skilles: Annotated[List[str], Field(min_items=1, max_items=10, description="보유 기술 (1-10개)")]

try:
  valid_employee = Employee(
    id=1, name="teddy", age=30, salary=1000, skills=["python", "cpp"]
  )
except ValidationError as e:
  print(f"유효성 검사 오류 : {e}")
```

- 생성시 조건에 맞지 않으면 Exception 발생함

#### Reducer

- `add_messages`는 LangGraph에서 메시지를 리스트에 추가하는 함수

```python
from langchain_core.messages import AIMessage, HumanMessage
from langgraph.graph import add_messages

msg1 = [HumanMessage(content="hi", id="1")]
msg2 = [AIMessage(content="hello", id="2")]

# msg1과 msg2 의 원소가 모두 분리하여 다시 새로운 하나의 리스트를 새엇ㅇ
# id가 같으면 대체됨
result1 = add_messages(msg1, msg2)
```

## 2. LangGraph 챗봇 만들기

- LangGraph를 사용한 간단한 챗봇 만들기
- 전체적인 Step 구조는 나중에 다른 프로젝트도 비슷할 것

#### Step 1. 상태 (State) 정의

```python
from typing import Annotated, TypedDict
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages

class State(TypedDict):
  # 메시지 정의 : list type 이며, add_messages 함수를 사용하여 메시지를 추가
  messages: Annotated[list, add_messages]
```

#### Step 2. 노드(Node) 정의

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

# 챗봇 노드 (함수) 정의
def chatbot(state: State);
  # 새로운 State안에 messages는 add_messages에 의해 대입이 아닌 기존 리스트에 답변이 추가되는 형태로 작동하게 됨
  return State(messages=[llm.invoke(state["messages"])])
```

#### Step 3. 그래프(Graph) 정의, 노드 추가

```python
graph_builder = StateGraph(State)

# chatbot 노드 추가
graph_builder.add_node("chatbot", chatbot)
```

#### Step 4. 그래프 엣지(Edge) 추가

```python
graph_builder.add_edge(START, "chatbot")
graph_builder.add_edge("chatbot", END)
```

#### Step 5. 그래프 컴파일

```python
graph = graph_builder.compile()
```

#### Step 6. 그래프 시각화

```python
from langchain_teddynote.graphs import visualize_graph

visualize_graph(graph)
```

#### Step 7. 그래프 실행

```python
question = "서울의 유명한 맛집 TOP 10 추천해줘"

for event in graph.stream({"messages": [("user", question)]}):
  for value in event.values():
      print("Assistant:", value["messages"][-1].content)
```

> langGraph에서 streaming은 노드 단위의 출력물을 말하는 거임. langchain과 다름. langchain에서의 stream같은건 따로 있음

## 3. Function Calling LLM과 도구 호출 노드, Conditional Edge 구축 방법

- LangGraph와 Agent를 함께 썻을 때 시너지가 좋음
- llm이 적절한 도구를 쓰면 좋은데, 자유도가 높아 컨트롤이 어려울 때가 있는데, LangGraph를 사용하면 컨트롤러빌리티를 높일 수 있음

```
__start__ -> chatbot <-|     -> __end__
             |-> tools -
```

- messages에 HumanMessage()가 담김
- 챗봇이 도구호출 내용(도구 이름, args) 내용인 AIMessage()를 messages에 담아 tools에 넘김
- tools가 해당 내용을 실행하고 실행한 내용을 ToolMessage()에 내용을 담아 messages에 담아 다시 chatbot에 넘김
- chatbot에서 **end**로 갈지 tools로 갈지 판단은 ConditionalEdge(add_conditional_edges)에서 path를 사용
  - 여기서는 chatbot의 message의 tool_calls가 있는 경우 tools로 가게끔 설정

> LangGraph에서 제공하는 ToolNode를 사용하면 사전에 정의된 tool들을 사용해볼 수 있음

## 4. LangGraph 에 메모리 추가

- 멀티턴 대화 기능 구현을 위한 메모리 추가
- LangGraph의 persistant chckingpointing을 사용

## 나머지 가능한 것

- 이전 상태로 되돌리는거 가능
- 중간 단계에서 State를 임의로 수정하는 것 가능
- 전체 실행이 완료된 후 특정 노드로 되돌려 수정하는 것 가능
- 병렬 노드 처리 가능
- 토큰 단위 스트리밍 가능
- subgraph(하나의 노드로 간주)를 추가하여 복잡한 시스템을 구축 가능
- 여러 subgraph를 사용하는 것 가능
