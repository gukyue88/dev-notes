# Chapter 1. Tools/Toolkits

## 1. 랭체인에서 제공하는 다양한 도구 (Tools) 툴킷 (Toolkits)

#### Tool

- `Tool`은 에이전트, 체인 또는 LLM이 외부 세계와 상호작용하기 위한 인터페이스
- LangChain에서 기본 제공하는 도구를 사용하여 쉽게 도국를 활용할 수 있으며, Custom Tool을 쉽게 구축하는 것도 가능함
- LangChain에 의해 통합된 도구 리스트는 doc 참조

#### Python REPL 도구

- llm이 작성한 코드를 실행까지 시켜줌
  - 오류가 난 경우 llm에게 다시 코드를 짜도록 할 수 있음
  - 정상적으로 끝난 경우 결과값도 받아볼 수 있음

```python
from langchain_experimental.tools import PythonREPLTool

python_tool = PythonREPLTool()

print(python_tool.invoke("print(100 + 200)"))
```

#### TavilySearchResults와 ravilyAnswer

- 웹 검색을 해주는 도구
- API 키가 별도로 필요함
- 여러 웹 검색 도구 중 지금까지는 성능이 가장 괜찮은 도구라고 함

```python
from langchain_community.tools.tavily_search import TavilySearchResults

tool = TavilySearchResults(
    max_result=6,
    include_answer=True,
    include_raw_content=True,
    # include_images=True,
    # search_depth="advanced", # 문서 내 link 검색
    include_domains=["github.io", "wikidocs.net"],
    # exclude_domains=[]
)

tool.invoke({"query": "LangChain Tools에 대해서 알려주세요"})
```

#### Image 생성 도구 (DALL-E)

- OpenAI의 DALL-E 이미지 생성기를 위한 Wrapper
- query를 넣으면 image_url를 return 함

## 2. 사용자 정의 도구 (Cutom Tool) 만드는 법

- langchain에 인테그레이션 되어있는 도구 외 커스텀 도구 만들기

```python
from langchain.tools import tool

# 데코레이터를 사용하여 함수를 도구로 변환
@tool
def add_numbers(a: int, b: int) -> int:
    """Add two numbers"""
    return a + b

add_numbers.invoke({"a": 3, "b": 4})
```

- docstring을 써주는게 중요함 -> llm이 언제 써야하는지 알 수 있음
  - 영어로 써주는 것이 좋음
