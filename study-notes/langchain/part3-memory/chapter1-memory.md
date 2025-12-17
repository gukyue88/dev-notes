# Chapter 1. Memory

## 1. 메모리란?

- Memory : 메시지들을 기억하고 있는 역할
- GPT 자체가 이전의 내용을 기억하는걸 가지고 있지는 않고, 별도로 메모리를 구현한 다음 gpt에 내용을 같이 넣어주는 형태
- 대화를 기록할 때는 항상 질문과 답변에 대한 쌍을 저장함

## 2. ConversationBufferMemory

- 메시지를 저장한 다음 변수에 메시지를 추출하게 해줌
- 가장 기본 메모리
- `save_context`를 통해 대화 내용을 저장하고, `load_memory_variables()`함수를 통해 대화 내용을 load 할 수 있음

## 3. ConversationBufferWindowMemory

- ConversationBufferMemory는 무한정으로 메시지를 저장하다보니 llm의 input이 넘칠 수 있음
- window 크기를 제약할 수 있도록한 ConversationBufferMemory

## 4. ConversationTokenBufferMemory

- 대화의 개수가 아닌 토큰 길이를 사용하여 대화내용을 flush할 시기를 결정함

## 5. ConversationEntityMemory

- 과거의 대화 내용 중 Entity(핵심) 정보를 추출하여 저장함

## 6. ConversationKGMemory

- 지식 그래프(Entity 메모리와 느낌은 비슷함) 형태로 내용을 저장하는 메모리

## 7. ConversationSummaryMemory

- 테디님이 가장 좋아하는 메모리
- 요약본을 저장하는 메모리
- ConversationSummaryMemory : 바로 바로 요약본을 만듦
- ConversationSummaryBufferMemory : 일정 수준이 넘어가게 되면 그때부터 요약본을 만듦
  - 이걸 보통 가장 많이 추천

## 8. VectorStoreRetrieverMemory

- 대화 내용을 VectorStore라는 DB에 저장하고, 호출될 때마다 가장 '눈에 띄는' 상위 K개의 문서를 쿼리함
- 과거의 대화 내용을 시간 순이 아닌 검색을 통해 가져오는 것이 특징

## 9. LCEL Chain에 메모리 추가

```python
model = ChatOpenAI()
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful cahtbot"),
        MessagesPlaceholder(variable_name="chat_history"),
        ("human", "{input}"),
    ]
)

memory = ConversationBufferMemory(return_messages=True, memory_key="chat_history")

# 메모리 변수를 빈 dict로 초기화
meemory.load_memory_variables({})

# memory의 load_memory_variables 콜백함수를 전달
# {'chat_history': []} 에서 [] 만 꺼내서 'chat_history'에 매칭
runnable = RunnablePassthrough.assign(
  chat_history=RunnableLambda(memory.load_memory_variables)
  | itemgetter("chat_history")
)

# runnable.invoke({"input": "hi})
# {'input': 'hi', 'chat_history': []}

chain = runnable | prompt | model
# runnable의 input과 chat_history가 prompt를 채움
```

- MessagePlaceholder로 prompt에 공간을 잡아두었다가, 대화 history 를 넣는데 사용
- 대화 history는 메모리에서 꺼냄

## 10. SQLite에 대화내용 (사용자 및 대화별) 저장

- InMemory를 사용하면 유저가 떠나면 대화내역이 사라짐
- SQLChatMessageHistory를 사용하면 SQLite DB에 대화내용을 저장할 수 있음

## 11. 일반 변수에 대화내용 저장(휘발성 메모리)

- InMemory를 사용하는 형태 (휘발성 메모리)
- 여러 방법 중 `RunnableWithMessageHistory`를 사용하는 방법에 대해 설명함
