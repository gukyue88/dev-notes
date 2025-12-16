# Chapter 6. model

## 1. RAG 에서의 LLM

- GPT-4o, Claude3 Sonnet, llama3-8b 등 종류가 많음
- 보안 걱정, 성능 측면, 금액 측면등을 고려하여 선택

## 2. 다양한 LLM과 활용방법

- 모델 종류에 따라 API키를 받거나, 사용 방법이 조금씩 다름

#### 종류

- OpenAI's GPT
- Anthoropic's Claude (버전 Opus, Sonnet, Haiku)
- Cohere's Cohere

## 3. LLM의 답변을 캐싱

- 동일한 질문이 빈번한 케이스에서 이전 질문에 대한 답변을 새로 생성하지 않고 캐시 정보를 활용함
- 비용절감 효과와 빠른 응답 속도
- GPT한테 물어보지 않고, 우리 서비스의 메모리에 캐싱을 해놓는 원리임
- `InMemoryCache`를 사용하면 휘발성 cache를 사용
- `SQLiteCache`를 사용하면 비휘발성 cache를 사용

## 4. 모델의 저장 및 로드 (직렬화, 역직렬화)

- serialization : 모델을 저장 가능한 형식으로 변환하느 과정
- chain을 JSON등의 형식으로 serialization 하여 저장
- JSON등의 형식으로 저장된 chain을 deserialization 하여 로드
- 모든 모델이 직렬화 되지는 않고, 되는 애들이 있음
- `is_lc_serailizable()` 를 사용하면 직렬화 가능한지 체크할 수 있음
- `dumpd`로 dict 타입으로 저장, `Pickle`을 통해 파이썬 dict를 저장하고 load

## 5. 토큰 사용량 확인

코드 상으로 토큰 사용량 확인하는 방법

```python
with get_openai_callback() as cb:
    result = ll.invoke("대한민국의 수도는 어디야?")
    print(cb.total_tokens)
    print(cb.prompt_tokens)
    print(cb.completion_tokens)
    ..
```

## 6. Gemini

```bash
pip install langchain-google-genai
```

```python
from langchain_google_genai import ChatGoogleGenerativeAI

llm = ChatGoogleGenerativeAI(model="gemini-1.5-pro-latest")

answer = llm.stream("자연어 처리에 대해서 간략히 설명해줘")
stream_response(answer`)
```

- batch 기능도 활용할 수 있음
- MultiModal을 활용해서 이미지도 같이 넣어줄 수 있음

## 7~10. Hugging Face

AI 개발자들이 서로 협력하고, 모델과 데이터셋을 공유하며, ML 애플리케이션을 만들 수 있도록 돕는 중앙 허브 역할을 합니다.

- 모델 공유
- 데이터셋 공유
- 호스팅 서비스 제공
- python 코드 라이브러리도 제공

## 11~12. Ollama

- 로컬 환경에서 **대규모 언어 모델(LLM)**을 쉽고 효율적으로 실행하고 관리할 수 있도록 해주는 오픈 소스 플랫폼 또는 도구
- 쉽게 말해, 복잡한 설정 없이도 **사용자의 개인 PC(Mac, Linux, Windows)**에서 Llama 3, Mistral, Gemma 같은 유명한 오픈 소스 AI 모델들을 직접 다운로드하고 실행해 볼 수 있게 해주는 '완성차' 같은 역할을 합니다.

#### Ollama의 핵심 기능과 장점

1. 쉬운 실행과 관리
   - 간단한 명령어: 복잡한 설치 과정 없이, ollama run llama3 같은 명령어 하나로 모델 다운로드부터 실행까지 모두 처리할 수 있습니다.
   - 모델 허브 역할: 다양한 오픈소스 LLM을 손쉽게 검색하고 다운로드하며 관리할 수 있습니다.
2. 로컬 실행의 이점
   - 개인 정보 보호(프라이버시): 데이터가 클라우드가 아닌 사용자 자신의 컴퓨터에 남아있기 때문에 보안 및 프라이버시 측면에서 매우 유리합니다.
   - 비용 절감: 클라우드 서비스의 API 사용료를 지불할 필요가 없어 비용 효율적입니다.
   - 오프라인 사용: 인터넷 연결 없이도 AI 모델을 실행할 수 있습니다.
   - 빠른 응답 속도: 클라우드 서버와의 통신 지연 없이 바로 처리되므로 응답 속도가 빠릅니다.
3. 개발 편의성
   - API 지원: RESTful API를 제공하여, 개발자들이 자신이 만든 애플리케이션에 로컬에서 실행되는 LLM 기능을 쉽게 통합할 수 있습니다.
   - 커스터마이징: Modelfile이라는 간단한 설정 파일을 통해 모델의 매개변수나 시스템 프롬프트를 조정하는 등 커스터마이징이 용이합니다.

#### gguf

- llm 모델을 하나의 파일 형태로 만들어서 공유할 수 있도록한 확장자

#### 13. GPT4ALL로 로컬 모델 실행

- Ollama랑 비슷한데, Ollama가 좀 더 빠름
