# Chapter 4. LangGraph Usecases

## 1. 에이전트간 대화 시뮬레이션

```mermaid
graph TD
    start(Simulated User Message)
    assistant((AI Assistant))
    user((Simulated User))
    end(END)

    start --> assistant
    assistant --> user
    user --> assistant
    assistant --> end
```

- 고객 응대 챗봇을 만들었을 때, 고객응대를 제대로 하는지 검증이 필요함
- 자가테스트 용도로 시나리오 테스트할 때 좋음
- 가상의 유저를 만들고 상황을 주어줘서 AI Assistant가 제대로 응대 하는지를 테스트함
