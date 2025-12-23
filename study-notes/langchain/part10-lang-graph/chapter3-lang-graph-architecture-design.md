# Chapter 3. LangGraph 구조 설계

## 1. LangGraph로 자유롭개 그래프 로직 구성

- LangGraph의 그래프 정의 단계
  - State 정의
  - 노드 정의
  - 그래프 정의
  - 그래프 컴파일
  - 그래프 시각화

> 노드와 edge 구조를 가지면 유연하게 구조 설계가 가능함

## 2~5. RAG(전통적인 방식의 RAG)를 LangGraph로 구현

```mermaid
graph TD
    question --> query_rewrite
    query_rewrite --> retrieve
    retrieve --> relevance_check{relevance check}

    relevance_check -- no --> web_search
    web_search --> llm

    relevance_check -- yes --> llm
    llm --> END
```

## 6. [프로젝트] Agentic RAG

```mermaid
graph TD
    question --> Agent

    Agent -- tools --> retrieve
    Agent -- no_tools --> END

    retrieve -- need_rewrite --> Agent
    retrieve -- no_need_rewrite --> generate
    generate --> END
```

- Agent를 쓰게 되면 RAG를 쓸지 말지 스스로 결정하게 된다는게 이점임
- 여기서 Agent는 retrieve_tool을 가지고 있음

## 7. [프로젝트] Adaptive RAG
