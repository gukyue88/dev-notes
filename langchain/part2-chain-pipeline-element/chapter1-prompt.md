# Chapter 1. 프롬프트

## 1. PromptTemplate, 부분변수(partial_variables)

#### from_template() 메소드를 사용하여 PromptTemplate 객체 생성

```python
template = "{country}의 수도는 어디인가요?"

# from_template 메소드를 이용하여 PromptTemplate 객체 생성
prompt = PromptTemplate.from_template(template)

chain = prompt | llm
chain.invoke({"country": 대한민국"})
```

#### PromptTemplate 객체 생성과 동시에 prompt 생성

```python
template = "{country}의 수도는 어디인가요?"

prompt = PromptTemplate(
    template=template,
    input_variables=["country"],
)
```

- from_template이 아닌 PromptTemplate 생성자를 사용해서 prompt를 만들어줄 수 있음
- partial_variables를 사용하면 부분 변수를 미리 채울 수 있음
- partial_variables에 함수를 넣어줘서 format이 실행되는 시점에서 함수를 호출하여 채우기도 함

## 2. yaml 파일로부터 프롬프트 템플릿 로드

prompt를 yaml 형태로 저장하고 load 할 수 있음

```yaml
_type: "prompt"
template: "{fruit}의 색깔이 뭐야?"
input_variables: ["fruit"]
```

```python
from langchain_core.prompts import load_prompt

prompt = load_prompt("prompts/fruit_color.yaml")
# PromptTemplate(input_variables=['fruit'], template='{fruit}의 색깔이 뭐야?')
```

## 3. ChatPromptTemplate

대화를 주고 받는 형식의 챗봇을 쓸 때는 ChatPromptTemplate를 사용함

```python
from langchain_core.prompts import ChatPromptTemplate

chat_template = ChatPromptTemplate.from_messages(
    [
        # role, message
        ("system", "당신은 친절한 AI 어시스턴트 입니다. 당신의 이름은 {name} 입니다."),
        ("human", "반가워요!"),
        ("ai", "안녕하세요! 무엇을 도와드릴까요?"),
        ("human", "{user_input}"),
    ]
)

chain = chat_template | llm
chain.invoke({"name": "Teddy", "user_input": "당신의 이름은 무엇입니까?"})
```

- ChatPromptTemplate은 messages 리스트를 관리
- 리스트 안에 요소는 'system', 'human', 'ai'의 PromptTemplate 객체로 구성됨
  - 여기에서 system은 전역설정과 관련된 내용을 담음. 대화 전체에 적용됨 (예. 페르소나)

## 4. MessagesPlaceHolder

아직 확정되지 않은 메시지나 대화를 위해 위차만 잡아주는 역할(?)

```python
from langchain_core.prompts import ChatPromptTemplate, MessagePlaceHolder

chat_prompt = ChatPromptTemplate.from_messages(
    [
        MessagesPlaceHolder(variable_name="conversation"),
        ("human", "지금까지의 대화를 {word_count} 단어로 요약합니다.")
    ]
)

chain = chat_prompt | llm

chain.invoke(
    {
        "word_count": 5,
        "conversation": [
            ("human", "안녕하세요. 저는 오늘 새로 입사한 테디입니다. 만나서 반갑습니다."),
            ("ai", "반가워요! 앞으로 잘 부탁드립니다.")
        ]
    }
)
```

- word_count에는 5가 들어가고, MessagePlaceHolder에 conversation 내용이 들어감

## 5. FewShotPromptTemplate

- prompt 기법 중 답변의 예시를 보여주는 기법으로 one shot, few shot 등이 존재함
- 하나의 답변 예시를 보여주는 것이 one shot, 두 개의 답변 예시를 보여주는 것이 few shot임

```python
prompt = FewShotPromptTemplate(
    ...
)
```

- FewShotPromptTemplate 사용법은 나중에 확인
- 요점은 One Shot, Few Shot 기법을 활용하기 위해 prompt 객체를 만들 때 예시를 넣을 수 있는 FewShotPromptTemplate을 사용한다는 것임

## 6. Example Selector

- FewShotPromptTemplate을 사용할 때의 문제는 모든 example이 다 입력으로 들어간다는 거임
- 입력으로 넣어주는건 다 돈이기 때문에, 질문과 유사도가 높은 애들만 입력으로 넣기 위해 사용

```python
example_selector = SemanticSimilarityExampleSelector.from_examples(
    examples, # 예시
    OpenAIEmbeddings(), # 문장을 숫자로 변경해줄 객체
    Chroma, # DB
    k=1, # 생성할 예시의 수
)
```

- selector는 FewSHotPromptTemplate 객체 생성시 넣어줌

## 7. MaxMarginalRelevance (MMR) 알고리즘

- SemanticSimilaryty는 문맥 유사도만 고려한 Example Selector이고
- MaxMarginalRelevance는 관련성과 문서간의 차별성을 모두 고려한 Example Selector임
- MaxMarginalRelevanceExampleSelector를 사용

## 8. FewShotChatMessagePromptTemplate

- FewShotPromptTemplate에서 chat을 추가한 객체라고 보면됨

## 9. 목적에 맞는 예제 선택기 (CustomExampleSelector)

- 위에서 사용한 ExampleSelector는 전체 내용을 보고 판단하는데, 특정 키 값에 대한 내용의 유사도만 비교하고 싶을 때 사용
- teddy님이 key값을 받아 해당 key값에 대한 값만 비교하도록 따로 객체를 구현함

## 10. LangChain Hub

- 프롬프트를 잘 작성하는게 llm으로 부터 좋은 답변을 얻는 지름길임
- 잘 작성된 Prompt를 공유한 것이 LangChain Hub임
- Lang Smith 사이트에서도 확인 가능함
- Top Viewed나 Top Download를 눌러 정렬해서 위쪽에 올라온걸 보면 좋음
- rlm/rag-prompt는 다운로드수가 많긴 하지만, 평균적인 프롬프트로 그 이후에 것을 이용하는 것이 좋음
- langchain에서 코드로 프롬프트를 가져오는 것도 제공함

```python
from langchain import hub

prompt = hub.pull("rlm/rag-prompt")
```

내가 작성한 프롬프트를 개인적으로 등록하는 것도 가능함

```python
from langchain import hub

hub.push("teddynote/simple-summary-korean", prompt)
```
