# Chapter 2. Agent 주요 기능

## 1. LLM에 도구 바인딩 (Binding Tools)

- llm에 tools를 바인딩한다 == function calling을 한다 == llm(뇌)에게 tool(손과 발)을 달아준다
- 모든 모델이 이걸 지원하지는 않음. llm이 학습할 때, 도구를 쓰는 방법도 학습이 되어 있어야 함
- 대부분의 최신 모델들은 도구 바인딩이 학습 되어 있음
- `.bind_tools()` 를 써서 llm에게 tools를 쥐어줌

```python
# 도구 정의
@tool
def get_word_length(word: str) -> int:
    """Returns the length of a word."""
    return len(word)

@tool
def add_function(a: float, b: float) -> float:
    """Adds two numbers together."""
    return a + b

@tool
def naver_news_crawl(news_url: str) -> str:
    """Crawls a 네이버 (naver.com) news article and returns the body content."""
    response = requests.get(news_url)
    if response.status_code == 200:
        soup = BeautifulSoup(response.text, "html.parser")
        title = soup.find("h2", id="title_area").get_text()
        content = soup.find("div", id="contets").get_text()
        cleaned_title = re.sub(r"\n{2,}", "\n", title)
        cleaned_content = re.sub(r"\n{2,}", "\n", content)
    else:
        print(f"HTTP 요청 실패. 응답 코드 : {response.status_code}")


    return f"{cleaned_title}\n{cleaned_content}"

tools = [get_word_length, add_function, naver_news_crawl]

llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

# 도구 바인딩
llm_with_tools = llm.bind_tools(tools)

chain = llm_with_tools | JsonOutputToolsParser(tools=tools)

# tool_call_results에는 어떤 툴에 어떤 args를 넣어 실행할지 리스트 형태로 정의됨
tool_call_results = chain.invoke("What is the length of the word 'teddynote'?")

# tool_call_results의 내용을 실제 실행하는 부분
for tool_call_result in tool_call_results:
    tool_name = tool_call_result["type"]
    tool_args = tool_call_result["args"]

    matching_tool = next((tool for tool in tools if tool.name == tool_name), None)

    if matching_tool:
        result = matching_tool.invoke(tool_args)
        print(f"[{tool_name}] : {result}")
    else:
        print(f"Can't find {tool_name}")
```

- docstring을 상세하게 적어 tools의 기능을 llm에게 잘 알려주기
- 결론적으로 실제로 llm이 툴을 쓰는 방법을 알긴하는데 직접 사용하는건 아니고, 이 툴을 이렇게 이렇게 쓰면 되겠다라는 답변이 나오면, 그 답변대로 우리가 실행해주는 것
- 마지막 tool_call_results를 실행해 주는 부분을 함수로 만들어 파이프라인으로 넣어줘도 됨

## 2. Agent, AgentExector 생성 방법

#### bind_tools -> Agent & AgentExecutor로 대체

- bind_tools의 복잡한 방식을 좀 더 쉽게 만들 수 있는 것이 Agent & AgentExecutor임
- bind_tools는 내부 구조를 좀 자세히 보여주기 위해 먼저 보여준 것임

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_openai import ChatOpenAI
from langchain.agents import create_tool_calling_agent
from langchain.agents import AgentExecutor

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are very powerful assistant, but don't know current events"),
        ("user", "{input}"),
        MessagesPlaceholder(variable_name="agent_scratchpad")
    ]
)

llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

tools = [get_word_length, add_function, naver_news_crawl]

agent = create_tool_calling_agent(llm, tools, prompt)

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    handle_parsing_errors=True,
)

result = agent_executor.invoke({"input": "How many letter in the word 'teddynote'?"})

print(result["output"])
```

- agent를 쓰면 반복 호출 가능 (4개를 한하는걸 물어보면 4번의 호출을 해줌)
- agent가 어떤 함수를 호출했는지도 바로 확인 가능
- agent를 쓰면 함수 호출 뒤에 다시 llm에 넣는 것도 가능해보임 (예. 네이버 기사 크롤링 후 요약까지)

#### 도구 호출 에이전트 (Tool Calling Agent)

- llm + tools 에 cycle 개념까지 들어가 있음
- cycle이 있으면 python 코드를 만들어서 오류가 나면 다시 코드를 수정하고 하는 과정 같은걸 할 수 있음
- gpt 뿐만 아니라 다른 llm들도 사용해볼 수 있음

```python
prompt = ChatPromptTemplate.from_messages(
    [
        # 이럴 때는 이런 툴을 써라라고 명시해주면 llm이 툴을 잘 사용하는 경향이 있음
        ("system", "Your are a helpful assistant. Make sure to use the 'search_news' tool for searching keyword related news."),
        # 멀티턴 구현시 메모리 기능을 활용한 chat_history를 넣어주기
        ("placeholder", "{chat_history}"),
        ("human", "{input}"),
        # 에이전트에게도 텍스트형태로 생각해놓은 끄적여놓고 저장해놓을 공간이 필요함. 그 공간이 agent_scratchpad임
        ("placeholder", "{agent_scratchpad}")
    ]
)

llm = ChatOpenAi(model="gpt-4o-mini", temperature=0)

agent = create_tool_calling_agent(llm, tools, prompt)

# AgentExcutor는 agent를 제어해주는 역할을 함. agent를 실제 구동해주는 역할
# 예) agent의 최대 실행 루프를지정, 나중에 멀티 에이전트를 구성하거나 할때도 사용함
# agent와 agentExcutor는 짝으로 다님.

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    max_iterations=10,
    max_execution_time=10,
    handle_parsing_errors=True,
)

result = agent_executor.invoke({"input": "AI 투자와 관련된 뉴스를 검색해 주세요"})

print(result["output"])
```

## 3. Agent의 중간단계 스트리밍(stream), AgentStreamParser

- Agent의 결과만 출력하지 않고, 스트리밍 형태로 중간 과정을 보고 싶을 때 사용
- 조금 복잡하긴 함

```python
result = agent_executor.stream({"input": "AI 투자와 관련된 뉴스를 검색해 주세요"})
for step in result:
    print(step)
    print("-------------------------------------")
```

- AgentExecutor의 `stream()` 메서드를 사용하면 agent의 중간 단계를 스트리밍 함
- return은 Action, Observation 단계가 번갈아 나타나며, 최종적으로 에이전트가 목표를 달성했다면 답변으로 마무리됨
  - Action 단계 : `actions` : AgentAction 또는 하위 클래스, `messages` : 액션 호출에 해당하는 채팅 메시지
  - Observation 단계 : `steps` : 현재 액션과 그 관착을 포함한 에이전트가 지금까지 수행한 작업의 기록, `messages` : 함수 호출 결과(즉, 관찰)을 포함한 채팅 메시지
  - Final Answer 단계 : `output` : AgentFinish, `messages` : 최종 출력을 포함한 채팅 메시지

## 4. Agent에 메모리 추가 (멀티턴 구현)

- `RunnableWithMessageHistory` 를 가지고 `agent_executor`를 감싸서 사용

```python
def get_session_history(session_ids):
    if session_ids not in store:
        store[session_ids] = ChatMessageHistory()
    return store[session_ids]

agent_with_chat_history = RunnableWithMessageHistory(
    agent_executor,
    get_session_history,
    input_messages_key="input",
    history_message_key="chat_history",
)
```

## 5. 다양한 LLM을 활용한 에이전트 생성

- OpenAI 외에도 Google Gemini, Ollama 등 더 광범위한 공급자 구현을 지원함

## 6. iter() 함수로 단계별 출력과 Human-in-the-loop

- agent의 실행 과정에서 개입하고 싶을때!

```python

@tool
def add_function(a: float, b: float) -> float:
    """Adds two numbers together."""
    return a + b

tools = [add_function]
gpt = ChatOpenAI(model="gpt-4o-mini")

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant."),
        ("human", "{input}"),
        MessagesPlacehodler(variable_name="agent_scratchpad"),
    ]
)

gpt_agent = create_tool_calling_agent(gpt, tools, prompt)

agent_excutor = AgentExecutor(
    agent=gpt_agent,
    tools=tools,
    verbose=False,
    max_iterations=10,
    handle_parsing_errors=True,
)

question = "11.2 + 13.5 + 123.1 의 계산 결과는?"

# AgentExecutor의 `iter()`를 사용하여 중간 단계를 처리하자
for step in agent_executor.iter({"input": question}):
    if output := step.get("intermediate_step"):
        action, value = output[0]
        if action.tool == "add_function":
            print(f"{action.tool}'s result : {value}")
        _continue = input("계속 진행? (y/n)?")
        if _continue.lower() != "y":
            break

if "output" in step:
    print(step["output"])
```
