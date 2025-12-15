# Chapter 2. 미니 프로젝트

## Streamlit으로 나만의 ChatGPT 웹앱 제작

- streamlit 으로 제작

#### 실행 방법

```bash
streamlit run main.py
```

#### 예제 코드

```python
import streamlit as st
from langchain_core.messages.chat import ChatMessage

st.title("나의 챗GPT")

# 사용자의 입력
user_input = st.caht_input("궁금한 내용을 물어보세요")
```

> 상세 코드는 github를 참조하자
