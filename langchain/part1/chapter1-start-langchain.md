# Chapter1. LangChain 시작하기

## 1. 강의를 보는 방법

- 강의 목적 : RAG 를 마스터 하가는 아니고, 밑거름을 마련하는데 목적을 둠
- 디스코드 : 테디노트의 RAG 비법 노트
- 모두 다 보려면 시간이 오래걸리기 떄문에 알려주는 주요 내용 먼저 보면서 RAG 파이프라인을 먼저 만들고, 필요한걸 선택해서 보는 컨셉으로 진행할 것
- 전자책 교재 : [랭체인LangChain 노트](https://wikidocs.net/book/14314)
  - 업데이트가 빠르기 떄문에 전자책으로 공유함

## 2. RAG (Retrieval-Augmented Generation) 기술

- RAG란?
  - Retrieval : 검색
  - Augmented : 증강
  - Generation : 생성

#### RAG를 사용해야 하는 이유

- ChatGPT의 문제점
  - 최신 정보에 대하여 학습되어 있지 않다.
  - 나(개인) 혹은 우리 회사에 제한되어 있는 내부데이터에 대한 학습이 되어 있지 않다
  - 따라서, 특정 도메인(나의 개인정보, 회사의 내부 정보)에 대한 질문을 하면 기대하는 답변을 얻을 수 없다
  - 문서화 시켜 업로드를 ChatGPT에서 질의할 수 있지만, 기대하는 답변을 받을 수 없거나, 할루시네이션(환각) 현상이 발생한다.
    - 문서의 양이 많아지면 더욱 더 이러한 현상은 심해진다.
- RAG를 적용했을 때
  - 최신 정보를 기반으로 답변할 수 있으며, 정보를 찾을 수 없는 경우 "검색" 기능을 활용하여 답변할 수 있다.
  - 나(개인) 혹은 우리 회사에 제한되어 있는 내부데이터를 참고하여 답변할 수 있다.
  - 문서를 내부 DB에 저장할 수 있고, DB에 내용을 축적해 나갈 수 있으며, 저장된 DB에서 원하는 정보를 검색하여 검색된 정보를 바탕으로 답변할 수 있다. (우리는 DB만 쌓고, 챗봇은 DB에 기반하여 최신 정보를 기반으로 답변)
  - 답변에 대한 출처를 역으로 저장되어 있는 DB에서 검색 후 검증하느 방식으로 할루시네이션 현상을 줄일 수 있다.

> 다른 말로 RAG는 GPT가 사전 학습한 정보에 더해 도메인 특화 정보를 던져주고 답변하게 하는 것

예시) 서울특별시에 사는 "테디"의 아버지 이름은 뭐야? -> 내부 문서에서 가족관계증명서를 참고하여 답변

> 고도화된 RAG 시스템은 정보를 잘 정제해서 gpt가 정제된 정보를 잘 활용할 수 있게 만드는 것

#### ChatGPT 안 RAG?

- ChatGPT에 문서를 업로드한 후, 질문을 하게되면 업로드 된 문서를 기반으로 답변하게 됨
- 다만, ChatGPT는 RAG의 전반적인 과정을 블랙박스로 공개하고 있지 않아 내부적으로 어떤 과정으로 RAG가 일어나는지 알 수 없음
- ChatGPT에 문서를 넣고 답변을 시켜보면 제대로 된 답변을 안하는 경우도 많고, 우리가 컨트롤 할 수 없음
- 우리가 할 수 있는 최선은 문서를 ChatGPT가 잘 검색할 수 있는 형태로 변경하는 것임

> ChatGPT가 문서 검색을 잘 하도록 우리가 가지고 있는 문서의 형태를 모두 변경하는 것은 사실상 어려운 일임

#### RAG 프로세스

```
                         Question
                             ↓
 -------------------------- RAG --------------------------
| Document -> Embedding -> Vector store (DB) -> Retriever |
 ---------------------------------------------------------
                             ↓
                           prompt
                             ↓
                            llm
                             ↓
```

- Document : 어떤 문서인가? (PDF, Word, Website, 논문, HWP, ...)
- Embedding : 한글/영어?, 의료용어?, 특정 도메인?
- Vector Store : 큰 용량의 데이터?, 개인이 사용하는 정도?
- Retriever : 키워드 검색?, 의미 유사도 검색?
- Prompt : 프롬프트 엔지니어링
- llm : GPT 3.5, GPT 4, Claude, Llama 3

#### 할 것

- 처음 기본적인 RAG 시스템을 구축하고 나면 생각보다 답변이 잘 안나옴
- RAG 시스템 안에 내용을 하나 하나 튜닝해가면서 RAG 시스템을 고도화 해야함

#### optimization flow

![optimazation flow](https://www.google.com/url?sa=t&source=web&rct=j&url=https%3A%2F%2Fmedium.com%2F%40luvverma2011%2Foptimizing-llms-best-practices-prompt-engineering-rag-and-fine-tuning-8def58af8dcc&ved=0CBUQjRxqFwoTCMD-jNnXr5EDFQAAAAAdAAAAABAH&opi=89978449)

- Prompt Engineering에는 한계가 있음
- RAG 적용 : Context Optimization을 끌어올릴 수 있음
- Fine-tuning : LLM Optimization을 끌어올릴 수 있음 (GPT는 범용 모델이고, 우리가 볼 데이터셋에 답변을 잘하도록 미세조정)
- Prompt Engineering + RAG + Fine-tuning : 세개 모두를 적용하면 원래 모델이 뿜어낼 수 있는 potential을 올릴 수 있음

#### RAG 특징

- Prompt Engineering을 배우는 것 만큼 쉬움
  - Fine-tuning이 어려움
- 어떤 질문을 했을 때, 답변이 어떤 과정을 거쳐나오는지 볼 수 있어서 분석에 용이함
- 유효한 정보 기반으로 답변을 강제할 수 있어서, 답변에 대한 출처를 다시 주어진 문서로부터 찾게 함으로서 할루시네이션을 줄일 수 있음

## 3. 위키독스 전자책 활용 바업

- 링크 : https://wikidocs.net/book/14314
- 다 하나씩 보는게 아니라 필요한 내용이 있을 떄, 검색해서 사용하자

## 4. 실습코드 (Github)

- 링크 : https://github.com/teddylee777/langchain-kr
- 업데이트가 자주 일어나다보니, 코드가 안되는 상황이 나올 수 있음
- 디스코드로 피드백 필요
