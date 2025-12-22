# Chapter 1. LangGraph 개요

## 1. LangGraph 개요

#### LangGraph의 탄생 배경

- RAG(Retrieval-Augmented Generation)이라는 기능이 아래의 문제점은 없는가?
  - LLM이 생성한 답변이 Hallucination이 아닐까?
  - RAG를 적용하여 받은 답변이 문서에는 없는 '사전지식'으로 답변한건 아닐까?
  - 문서 검색에서 원하는 내용이 없을 경우, '인터넷' 혹은 '논문'에서 부족한 정보를 검색해서 지식을 보강할 수는 없을까?
- 위 문제를 해결하려면 체인이 덕지덕지 붙게 되고, RAG 파이프라인의 복잡도가 올라감

#### 기존 RAG의 문제점

- 사전 정의된(이미 정해진) 데이터 소싱(PDF, DB, Table등) 자원
- 사전 정의된 Fixed Size Chunk
- 사전 정의된 Query 입력
- 사전 정의된 검색 방법
- 신뢰하기 어려운 LLM 혹은 Agent
- 고정된 프롬프트 형식
- LLM의 답변 결과에 대한 문서와의 관련성/신뢰성

> RAG 파이프 라인이 단방향 구조이기 때문에 생기는 문제점임. 모든 단계를 한 번에 잘해야하고, 이전 단계로 되돌아가기 어려움

#### LangGraph 제안

- 각 과정(예. Load, Split, Embed, store, retrieve, prompt, llm, ...)을 Node 라고 정의
- Node를 연겷하는 것을 edge라고 정의
- 조건부 edge를 통해 분기 처리

#### 예제

- 평가자를 두거나
- 웹검색을 더 넣거나
- 평가자를 두 명 이상 두거나

#### LangGraph

- Node(노드) : 함수로 이루어져 있음.
- Edge(엣지) : 노드 간 연결
- State(상태관리) : 노드 간 메시지 전달자 역할
- Conditional Edge : 조근부 흐름 제어
- Human in the loop : 필요시 중간 개입하여 다음 단계를 결정
- Checkpointer : 과거 실행 과정에 대한 '수정' & '리플레이' 가능

## 2. LangGraph 세부 기능

#### State

- 노드와 노드간 정보를 저장할 때 GraphState라는 객체에 정보를 담아 전달함
- GraphState는 TypedDict(Dictionary 개념에 타입 힌팅을 추가한 개념)를 상속 받음

#### Reducer

- `GraphState`를 보면 `messages: Annotated[list, add_messages]` 처럼 되어있음
- `from langgraph.graph.message import add_messages`라고 해서 langgraph에서 제공해줌
- question과 messages는 `=`를 실행하면 append가 동작되도록 `add_messages`로 구현됨

#### 노드

- 함수로 정의
- 입력 인자 : State 객체
- 반환 : 대부분 State 객체, Conditional Edge의 경우 다를 수 있음

```python
from langgraph.graph import END, StateGraph
from langgraph.checkpoint.memory import MemorySaver

workflow = StateGraph(GraphState)

# 노드 정의 : add_node
# 노드 이름은 가능하면 영어로 작성할 것!
workflow.add_node("retriever", retrieve_document)
workflow.add_node("llm_answer", llm_answer)
```

#### 엣지

- 노드간 연결 : `add_edge("from노드", "to노드")`

```python
workflow.add_edge("retreive", "llm_answer")
```

#### 조건부 엣지

- 조건부 엣지 생성 : `add_conditional_edges("노드이름", 조건부판단함수, 매핑dict)`

```python
workflow.add_conditional_edges(
  "relevance_check",
  is_relevant,
  {
    "grounded": END, # 관련성이 있으면 종료 (is_relevant의 결과가 grounded이면)
    "notGrounded": "llm_answer", # 관련 없으면 다시 답변 생성 (is_releavant의 결과가 notGrounded이면)
    "notSure": "llm_answer" # 모호하면 다시 답변 생성 (is_relevant의 결과가 notSure이면)
  }
)
```

#### 시작점 지정

```python
workflow.set_entry_point("retrieve")
```

#### 그래프 생성 및 시각화

```python
# 그래프 컴파일
app = workflow.compile()

# 메모리 (체크포인트 or 스냅샷)을 사용할 경우, 아래와 같이
# 뒤로 되돌리기 가능
memory = MemorySaver()
app = workflow.compile(checkpointer=memory)

# 그래프 시각화 : 인터넷이 연결되어 있어야 함서버
display(
  Image(app.get_graph(xray=True).draw_mermaid_png())
)
```

#### 그래프 실행

```python
from langchain_core.runnables import RunnableConfig

# recursion_limit : 최대 반복 횟수, 최대 노드 실행 개수 지정
# thread_id: 실행 ID (구분용), 그래프 실행 ID를 기록하고, 추후 추적하기 위한 목적으로 활용
config = RunnableConfig(recursion_limit=13, configurable={"thread_id": "SELF-RAG"})

# GraphState 객체 활용
inputs = GraphState(question="삼성전자가 개발한 생성형 AI의 이름은?")
output = app.invoke(inputs, config=config)

print(output)
```
