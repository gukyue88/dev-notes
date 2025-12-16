# Chapter 4. Ouput Parser

## 1. 출력 파서

- 출련 파서란? 출력을 구조화된 형태(예. JSON)로 변환하는 컴포넌트
- 자동화 측변에서 일반적인 text 답변에서는 우리가 원하는 정보를 쪽쪽 뽑기 힘든데, 이걸 뽑아서 구조화된 형태로 바꿔주는게 파서의 역할임

#### 주요 특징

- 다양성 : langchain은 많은 종류의 출력 파서를 제공함
- 스트리밍 지원 : 많은 출력 파서들이 스트리밍을 지원함
- 확장성 : 최소한의 모듈부터 복잡한 모듈까지 확장 가능한 인터페이스 제공

#### 출력 파서의 이점

- 구조화 : LLM의 자유 형식 텍스트 출력을 구조화된 데이터로 변환
- 일관성 : 출력 형식을 일관되게 유지하여 후속 처리 용이
- 유연성 : 다양한 출력 형식 (JSON, 리스트, 딕셔너리 등)으로 변환 가능

## 2. PydanticOutputParser

- Pydantic은 Python에서 가장 널리 사용되는 데이터 유효성 검사 라이브러리임
- PydanticOuputParser는 언어 모델의 출력을 더 구조화된 정보로 변환하는데 도움이 되는 클래스임
- 대부분의 OutputParser들은 두 가지 핵심 메서드가 구현 되어야 함
  - `get_format_instructions()` : 지침(형식)을 알려주는 역할
  - `parse()` : 구조화된 객체로 변환

```python
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import PydanticOutputParser
from langchain_core.pydantic_v1 import BaseModel, Field

llm = ChatOpenAI(temperatyure=0, model_name="gpt-4o")

class EmailSummary(BaseModel):
    person: str = Field(description="메일을 보낸 사람")
    email: str = Field(description="메일을 보낸 사람의 이메일 주소")
    subject: str = Field(description="메일 제목")
    summary: str = Field(description="메일 본문을 요약한 텍스트")
    date: str = Field(description="메일 본문에 언급된 미팅 날짜와 시간")

# PydanticOutputParser 생성
parser = PydanticOutputParser(pydantic_object=EmailSummary)

prompt = PromptTemplate.from_template(
    """
You are a helpful assistant. Please answer the following questions in Korean.

Question:
{question}

EMAIN CONVERSATION:
{email_conversation}

FORMAT:
{format}
"""
)

# format에 PydanticOutputParser의 부분 포매팅(partial) 추가
prompt = prompt.partial(format=parser.get_format_instructions())

chain = prompt | llm

response = chain.stream(
    {
        "email_conversation": email_conversation,
        "question": "이메일 내용 중 주요 내용을 추출해 주세요."
    }
)

output = stream_response(response, return_output=True)

# parser를 사용하여 결과를 파싱하고 EmailSummary 객체로 변환
email_summary = parser.parse(output)
print(email_summary.email)
print(email_summary.person)
```

> chain을 구성할 때, parser를 같이 넣어주면 get_format_instructions, parse 과정을 생략할 수 있음

```python
chain = prompt | llm | parser

response = chain.invoke(
    {
        "email_conversation": email_conversation,
        "question": "이메일 내용 중 주요 내용을 추출해 주세요."
    }
)

# response 가 EmailSummary 객체로 바로 나옴
```

## 3. with_structured_output() 바인딩

> `with_structed_output(Pydantic)`을 사용하여 출력 파서를 추가하면, 출력을 Pydantic 객체로 변환할 수 있음(?)

```python
llm_with_structured = ChatOpenAI(
    temperature=0, model_name="gpt-4o"
).with_structured_output(EmailSummary)

answer = llm_with_structured.invke(email_conversation)
```

- with_structured_output 함수는 stream 을 지원하지 않음

## 4. LangSmith에서 OutputParser 흐름 확인

- llm 입력을 보면 format에 EmailSummary의 get_format_instructions() 내용이 채워져서 들어간 것을 볼 수 있음

## 5~9 기타 Parsers

#### CommaSeperatedListOutputParser

> 답변을 List를 받고 싶을 때 사용

```python
from langchain_core.output_parsers import CommaSeperatedListOutputParser

output_parser = CommaSeparatedListOutputParsers()

prompt = PromptTemplate(
    template="List five {subject}.\n{format_instructions}",
    input_variables=["subject"],
    partial_variables={"format_instructions": output_parser.get_format_instructions()}
)

model = ChatOpenAI(temperature=0)

chain = prompt | model | output_parser
answer = chain.invoke({"subject": "대한민국 관광명소"})
print(answer)
# ["경복궁", "님산타워", "부산 해수욕장", "제주도 성산일출봉", "경주 불국사"]
```

- answer의 type은 List임

#### StructuredOutputParser

- StructuredOutputParser는 llm에 대한 답변을 `dict` 타입으로 구성함
- GPT는 PydanticOutputParser와 잘 동작하지만, 외부 llm의 경우 PydanticOutputParser가 잘 맞지 않은 경우가 있음.
  - 그럴때 대안으로 StructuredOutputParser를 사용
  - 일반적으로 Pydantic/JSON Parser가 더 강력하다는 평가를 받긴 함
- 쓰는 방법은 따로 참고

#### JsonOutputParser

- llm에 대한 답변을 `didt` 타입으로 구성
- 모델이 똑똑하면 잘 동작하나, 낮은 성능의 모델에서는 잘 동작하지 않는 경향이 있음

#### PandasDataFrameParser

- llm에 대한 답변을 PandasDataFrame 타입으로 구성
- 요건 또 상위모델이랑 오히려 상성이 안좋은 경향이 있음

#### DatetimeOutputParser

- llm에 대한 답변을 DataTime 타입으로 구성

#### EnumOutputParser

- llm에 대한 답변을 Enum 값 중 하나로 파싱하는 도구
- 답변을 enum 중에 하나로 받음
