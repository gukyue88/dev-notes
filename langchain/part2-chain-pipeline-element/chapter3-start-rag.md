# Chapter 3. RAG 시작하기

## 1. RAG 프로세스 이해하기 (Pre-processing)

- RAG와 GPT의 차이 : RAG는 참고할 만한 정보를 줌
- 참고할만한 정보(context)를 어떻게 주는가? => RAG Process

#### RAG Process (Pre-Processing)

1. Document load : word, pdf, excel, csv, text 등 load
2. Text Split : 단락을 구분하는 단계, 나중에 질문과 유사한 단락을 몇 개를 뽑아 그걸 기준으로 답변함
   - 자르는 덩이를 chunk라고 부르고, 중간에 잘리는 것을 없애기 위해 어느정도 overlap을 함
3. Embedding : 문자열을 수학적 표현으로 바꾸는 단계, Vector 형태로 표현됨 (예. OpenAi, HuggingFace), 질문과 유사도 계산을 위해 쓰임
4. DB (Vector DB) : FAISS, Chroma, Milvus, Pinecone
   - Embedding을 할 때 OpenAI를 사용하면 돈이 들어감
   - Embedding 데이터를 저장해놔서 재사용

> Pre-processing 단계는 한 번만 해주면 됨!

## 2. RAG 프로세스 이해하기 (Run time 단계)

1. Question : 유저가 질문
2. Retrieve(검색) : 유저의 질문과 유사도가 높은 단락들을 뽑아내는 단계, 위 DB (Vector DB)에서 유사도가 높은 단락을 검색하여 참고정보(context)를 뽑아냄
3. Prompt : Queustion + Context를 기반으로 prompt로 넣어줌
4. LLM
5. Answer

#### RAG Prompt 구조

> 지시사항(instruction) + 사용자 질문 + 문맥(검색된 정보)로 이루어짐

```
당신은 질문-답변 (Question-Answer) Task를 수행하는 AI 어시스턴트 입니다.
검색된 문맥(context)를 사용하여 질문(Question)에 답하세요.
만약, 문맥(context) 으로부터 답을 찾을 수 없다면 '모른다'고 말하세요.
한국어로 대답하세요.

#Question:
{이곳에 사용자가 입력한 질문이 삽입됩니다.}

#Context:
{이곳에 검색된 정보가 삽입됩니다.}
```

#### chain 구성

```
Question -> retriever -> db -> chunks -
    |                                  |
     --------------------------------- + -> prompt (chunks + question) -> llm
```

코드

```python
chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)
```

## 3. PDF 문서 기반 QA RAG

## 4~8 프로젝트 진행
