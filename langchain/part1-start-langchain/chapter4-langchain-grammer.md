# LangChain 문법 맛보기

## 1. ChatOpenAI 주요 파라미터, invoke(), stream() 스트리밍 출력

#### API KEY 로드

```python
from dotenv import load_dotenv

# API KEY 정보 로드
load_dotenv()
```

#### ChatOpenAI 객체 생성 및 사용

> ChatOpenAI 객체를 생성하고, invoke()를 통해 질문에 대한 답변을 받을 수 있음

```python
# langchain과 openai가 협업하여 만든 패키지
from langchain_openai import ChatOpenAI

# 객체 생성
# temperature : 샘플링 온도, 0~2 사이값 설정, 값이 높을 수록 출력이 더 무작위해지고, 0에 가까울 수록 창의성이 없어짐
# max_tokens : 채팅 완성에서 생성할 토큰의 최대 개수
# model_name : gpt-3.5-turbo, gpt-4-turbo, gpt-4o, ...
llm = ChatOpenAI (
    temperature=0.1,
    model_name="gpt-4o",
)

question = "대한민국의 수도는 어디인가요?"
response = llm.invoke(question)

# response는 객체형태로 넘어옴
print(response)
```

#### 스트리밍 출력

> 스트리밍 : 실시간으로 답변을 받을 때 사용

```python
# 스트림 방식으로 질의
answer = llm.stream("대한민국의 아름다운 관광지 10곳과 주소를 알려주세요!")

# 스트리밍 방식으로 각 토큰을 출력 (실시간 출력)
for token in answer:
    print(token.content, end="", flush=True)
```

## 2. LangSmith로 GPT 추론 내용 추적

```python
from langchain_teddynote import logging # teddy님이 만든 패키지

# langsmith에서 추적할 수 있도록 함. parameter는 프로젝트 이름으로 langsmith 페이지에서 보일 이름이 됨
logging.langsmith("CH01-Basic")
```

- langsmith 페이지에 들어가면, 실행한 과정에 대한 내용이 반영되어있음

## 3. GPT-4o (멀티모달) 모델로 이미지 인식하여 답변 출력

- llm : 텍스트 입력 -> 텍스트 출력
- 멀티모달 모델 : 입력과 출력이 텍스트 뿐만 아니라, 이미지, 오디오, 비디오로 확장
- gpt-4o는 이미지를 입력으로 넣을 수 있음

```python
from langchain_teddynote.models import MultiModal

llm = ChatOpenAI(
    temperature=0.1,
    model_name="gpt-4o"
)

# 멀티모달 객체 생성, 내부 구현은 teddy_note 패키지 내용 참고
multimodal_llm = MultiModal(llm)

# 샘플 이미지 주소
IMAGE_URL = "https://..."

# 이미지 파일로 부터 질의
response = multimodal_llm.invoke(IMAGE_URL)
print(response)
```

## 4. 프롬프트 템플릿

```python
from langchain_core.prompts import PromptTemplate

# template 정의
template = "{country}의 수도는 어디인가요?"

# from_template 메소드를 이용하여 PromptTemplate 객체 생성
prompt_tempate = PromptTemplate.from_template(template)

# prompt 생성
prompt = prompt_template.format(country="대한민국")
```

## 5. LCEL로 체인 생성

- LCEL : LangChain Expression Language
- LCEL을 활용하면 다양한 구성 요소를 단일 체인(파이프라인)으로 결합할 수 있음
  - `chain = prompt | model | output_parser`

```python
from langchain_openai import ChatOpenAI

prompt = PromptTemplate.from_template("{topic}에 대해 {how} 설명해주세요")
model = ChatOpenaAI()

chain = prompt | model

# 딕셔너리 형태로 input 작성, 변수가 하나일때는 문자열만 넣어도 되는데.. 헷갈리니 딕셔너리로 넣어주자
# PromptTemplate에 정의한 변수에 대한 내용을 채워서 전달
input = {
    "topic": "인공지능 모델의 학습 원리",
    "how": "자세히"
}

# invoke
chain.invoke(input)
```

## 6. 출력파서(StrOutputParser)를 체인에 연결

- langchain_core.output_parsers : answer가 AIMessage 객체 타입인데, 여기에서 필요한 내용만 파싱하게 도와줌
  - StrOutputParser : AIMessage 중 답변 내용(string 타입)만 뽑아줌

```python
from lanchain_core.ouput_parsers import StrOutputParser

output_parser = StrOutputParser()
```

#### 위 chain과 연결

```python
chain = prompt | model | output_parser

chain.invoke(input) # return이 AIMessage가 아닌, metadata만 나옴
```

## 6. Chain 추론 과정을 LangSmith에서 확인

- LangSmith에서 위 예제 실행 내용을 확인함ㅁ

## 7. LCEL 인터페이스 - 일괄처리 batch()

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser

model = ChatOpenAI()
prompt = PromptTemplate.from_template("{topic} 에 대하여 3문장으로 설명해줘")
chain = prompt | model |  StrOutputParser()

data = {"topic" : "멀티모달"}

# stream : 실시간 출력
for token in chain.stream(data):
    print(token, end="", flush=True)

# invoke : 호출
answer = chain.invoke(data)
print(answer)

# batch : 단위 실행
# config={"max_concurrency": 3} 를 사용하면 동시 요청수를 3으로 제한할 수 있음
batch_data = [{"topic": "멀티모달"}, {"topic": "Instagram"}]
answer = chain.batch(batch_data)
print(answer[0])
print(answer[1])
```

## 8. 비동기 호출 방법

```python
# astream
async for token in chain.astream(data):
  print(token, end="", flush=True)

# ainvoke
await chain.ainvoke(data)

# abatch
await chain.abatch(batch_data)
```

## 병렬체인 구성 (RunnableParallel)

- prompt, llm, parser 등이 모두 Runnable 프로토콜로 구현됨
- chain은 Runnable 을 묶을 수 있도록 되어있음
- chain 또한 Runnable이고, 그렇기 때문에 chain끼리 연결도 가능함
- `RunnableParallel`을 사용하면 chain을 Parallel 하게 수행할 수 있음

```python
combined = RunnableParallel(capital=chain1, area=chain2)
answer = combined.invoke({"country":"대한민국"})
print(answer)
# combined는 'capital', 'area' 키를 가진 딕셔너리 형태의 답변을 가짐

# batch도 가능함
combined.batch([{"country":"대한민국"}, {"country": "미국"}])
```

## 11. 값을 전달해주는 RunnablePassthrough

```python
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI

prompt = PromptTemplate.from_template("{num} 의 10배는?")
llm = ChatOpenAI(temperature=0)

chain = prompt | llm

# invoke를 사용한 chain 실행
response = chain.invoke({"num": 5})
```

##### RunnablePassthrough를 사용한 예제

```python
from langchain_core.runnables import RunnablePassthrough

runnalbe_chain = {"num": RunnablePassthrough()} | prompt | ChatOpenAI()
runnable_chain.invoke(10)
```

- prompt에는 dict 형태만 들어감
- 이전에는 invoke할때 prompt에 전달하기 위핸 {} 를 전달함
- RunnablePassthrough()를 사용하면, invoke시 {}가 아닌 값을 넣을 수 있음

## 12. 병렬로 Runnable을 실행하는 RunnableParallel

```python
from langchain_core.runnables import RunnableParallel

runnable = RunnableParallel(
  passed=RunnablePassthrough(),
  extra=RunnablePassthrough.assign(mult=lambda x: x["num"] * 3),
  modified=lambda x: x["num"] + 1,
)

result = runnable.invoke({"num": 1})
# result : {'passed': {"num": 1}, "extra": {"num": 1, "mult": 3}, "modified": 1}
```

## 13. RunnableLambda

RunnableLambda를 사용하여 사용자 정의 함수를 매핑할 수 있음?

```python
# a 파라미터가 존재하나, 실제 사용하지 않음
# RunnableLambda에 사용되는 함수는 무조건 dict가 넘어갈 매개변수 하나를 가져야함
def get_today(a):
  return datetime.today().strftime("%b-%d")

# RunnableLambda가 get_today 함수를 호출해서 today에 매칭, get_today의 파라미터 a의 값에도 3이 들어가긴함
# RunnablePasstrough는 사용자가 준 파라미터 3이 그대로 감
chain = (
  {"today": RunnableLambda(get_today)}, "n": RunnablePassthrough()}
  | prompt
  | llm
  | StrOutputParser()
)

chain.invoke(3)
```

#### invoke 안에 여러 파라미터를 주는 방법?

- dict나 list를 사용
- RunnablePassthrough()는 그대로 dict나 list가 전달됨
- 만약 dict의 특정 아이템을 얻고 싶다면 itemgetter를 사용
