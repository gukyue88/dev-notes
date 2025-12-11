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
