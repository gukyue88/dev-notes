# Chapter 3. agent 활용

이전까지는 agent를 만드는 문법에만 초점이 맞춰져있었다면, 여기서부터는 agent를 어떻게 활용할지에 대해 다룸

- agent를 활용해서 RAG를 하는 Agentic RAG
- 파일시스템을 건드려서 보정을 하는 것들
- csv 데이터를 agent에 데이터 분석을 수행하는 agent
- 주어진 도구를 활용해서 이멜이에 자동으로 답변, 발송을 해주는 에이전트 등

## 1. Agentic RAG

- Agentic RAG : Agent를 활용한 RAG
- RAG를 더 잘 수행할 수 있는 도구들을 agent 한테 던져줌
- retriever 기능에 웹 검색 도구를 추가하여, 문서 내 정보가 없을 경우 웹검색을 하게 한다거나
- python 코드 실행 도구를 쥐어줘서 문서 검색 내용 기반으로 python 코드를 작성하고 실행한다거나 등등
- 문서 안에 있는 내용만 검색해야 하는 경우는 RAG를 쓰는게 유리하고, 어떨 때는 문서 내 검색, 어떨 때는 웹검색을 해야하는 경우에는 Agentic RAG를 쓰는 것이 유리

```python
from langchain_community.tools.travily_search import TavilySearchResults
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
from langchain.dodument_loaders import PyPDFLoader
from langchain.tools.retriever import create_retriever_tool

# 웹 검색 도구
search = TavilySearchResults(k=6)

# 문서 검색 도구
loader = PyPDFLoader("data/xxx.pdf")
text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=100)
split_docs = loader.load_and_split(text_splitter)
vector = FAISS.from_documents(split_docs, OpenAIEmbeddings())
retriever = vector.as_retriever()
retriever_tool = create_retriever_tool(
    retriever,
    nam="pdf_search",
    # 도구에 대한 description을 자세히 기입해야 llm이 잘 사용함! 가끔적이면 영어로 번역!
    description="use this tool to search information from the PDF document",
)

tools = [search, retriever_tool]

llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

prompt = ChatPromptTemplate.from_messages(
    [
        # system prompt로 페르소나와 tool 의 구체적인 사용법을 명시
        ("system", "You are a helpful assistant. Make sure to sue the 'pdf_search' tool for searching information from the PDF Document. If you can't find the information from the PDF document, use the 'search' tool for searching information from the web"),
        ("placeholder", "{chat_history}"),
        ("human", "{input}"),
        ("placeholder", "{agent_scratchpad}")
    ]
)

agnet = create_tool_calling_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=False)

# session_id별 chat_history를 위한 RunnableWithMessageHistory 추가

store = {} # session_id 별 history 저장

def get_session_history(session_id):
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]

agent_with_chat_history = RunnableWithMessageHistory(
    agent_executor,
    get_session_history,
    input_messages_key="input",
    hitory_messages_key="chat_history",
)

response = agent_with_chat_history.stream(
    {"input": "2024년 프로야구 플레이오프 진출한 5개 팀을 검색하여 알려주세요."},
    config={"configuralble": {"session_id": "abc123"}},
    )

for step in response:
    print(step)
```

## 2. CSV, EXCEL 파일을 분석하는 데이터 분석

- agent를 활용하여 pandas DataFrame의 데이터를 분석해보자
- 데이터 분석에는 agent를 쓰는게 좋음. 데이터 분석시 오류가 있으면 수정하는 기능까지 포함하기 때문에

```python
import pandas as pd
from langchain_experimental.tools import PythonAstREPLTool

df = pd.read_csv("./data/titanic.csv")

# 파이썬 코드를 실행하는 도구 생성
python_tool = PythonAstREPLTool()
python_tool.locals["df"] = df

# 도구 호출시 실행되는 콜수함수
def tool_callback(tool) -> None:
    print(f"<<<<<<< Code >>>>>>>")
    if tool_name := tool.get("tool"): # 도구에 입력된 값이 있다면
        if tool_name == "python_repl_ast":
            tool_input = tool.get("tool_input")
            for k, v in tool_input.items():
                if k == "query":
                    priunt(v)
                    result = python_tool.invoke({"query": v})
                    print(result)
    print(f"<<<<<<< Code >>>>>>>")

# 관찰 결과를 출력하는 콜백함수
for observation_callback(observation) -> None:
    print(f"<<<<<<< Message >>>>>>>")
    if "observation" in observation:
        print(observation["observation"])
    print(f"<<<<<<< Message >>>>>>>")

# 최종 결과를 출력하는 콜백함수
def result_callback(result: str):
    print(f"<<<<<<< 최종 답변 >>>>>>>")
    print(result)
    print(f"<<<<<<< 최종 답변 >>>>>>>")

# prefix 에 세부적인 지시사항을 넣으면 더 좋은 결과물이 나옴
agent = create_pandas_dataframe_agent(
    ChatOpenAI(model="gpt-4o", temperature=0),
    df,
    verbose=False,
    agent_type="tool-calling",
    allow_dangerous_code=True,
    prefix="""
    You are a professional data analyst and expert in Pandas.
    You must use Pandas DataFrame('df') to answer user's request.

    [IMPORTANT] DO NOT create or overwrite the 'df' variable in your code.
    If you are willing to generate visualization code, please use 'plt.show()' at the end of your code.
    I perfer seaborn code for visualization, but you can use matplotlib as well.

    <Visualization Preference>
    - 'muted' cmap, white background, and no grid for your visualization.
    Recomment to set palette parameter for seaborn plot.
    """
)

parser_callback = AgentCallbacks(tool_callback, observation_callback, result_callback)
stream_parser = AgentStreamParser(parser_callback)

def ask(query):
    response = agent.stream({"input": query})

    for step in response:
        stream_parser.process_agent_steps(step)

ask({"input": "corr()을 구해서 히트맵 시각화"})
ask({"input": "몇 개의 행이 있어?"})
ask({"input": "남자와 여자의 생존률은 얼마야?"})
```

- llm에게 dataframe과 dataframe을 다룰 수 있는 python tool을 쥐어주고, 질문하면, 질문에 대한 df 관련 코드를 만든후 실행해보기도 하고, 실행중 에러가 발생하면 수정도 함. 시각화도 가능
- 데이터가 많은 경우, chatgpt한테 데이터를 다 넣을 수 없는데, 이 방법을 사용하면 코드를 만들어 필요한 데이터 내용만 query해서 사용하기 때문에 prompt가 절약이 되고 대용량 데이터도 처리할 수 있음

## 3. (업무자동화) FileManagementToolkits를 활용한...

- LangChain 프레임워크를 사용하는 가장 큰 이점은 3rd-party integration 되어 있는 다양한 기능들임
- 그 중 Toolkits는 다양한 도구를 통합하여 제공함
- Agent Toolkits 참고!
- Agent Toolkits 중 파일을 제어할 수 있는 FileManagementToolkits를 사용해보자
  - CopyFileTool : 파일 복사 툴
  - DeleteFileTool : 파일 삭제 툴
  - FileSearchTool : 파일 검색 툴
  - MoveFileTool : 파일 이동 툴
  - ReadFileTool : 파일 읽기 툴
  - WriteFileTool : 파일 쓰기 툴
  - ListDirectoryTool : 디렉토리 목록 조회 툴

```python
from langchain_community.agent_toolkits import FileManagementToolkit

working_directory = "tmp"
toolkit = FileManagementToolkit(root_dir=str(working_directory))
tools = toolkit.get_tools()

print("[사용 가능한 파일 관리 도구들]")
for tool in available_tools:
    print(f"- {tool.name}: {tool.description}")

@tool
def latest_news(k: int = 5) -> List[DIct[str, str]]:
    """Look up latest news"""
    news_tool = GoogleNews()
    return news_tool.search_latest(k=k)

# FileManagementToolkit에서 제공하는 tools에 latest_news 툴 추가
tools.append(latest_news)

store = {}
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant. Make sure to use the 'latest_news' tool to find latest news. Make sure to use the 'file_management' tool to manage files."),
        ("placeholder", "{chat_history}"),
        ("human", "{input}"),
        ("placeholder", "{agent_scratchpad}"),
    ]
)

# ...

result = agent_with_chat_history.stream(
    {"input" : "최신 뉴스 5개를 검색하고, 각 뉴스의 제목을 파일명으로 가지는 파일을 생성하고(.txt), 파일의 내용은 뉴스의 내용과 url을 추가하세요."},
    config={"configurable": {"sesion_id": "abc123"}},
)

for step in result:
    print(step)
```

- latest_news 도구를 사용해서 뉴스를 검색한 후,
- filemanagement toolkit을 사용하여 하나씩 저장하는 것을 확인해볼 수 있음

## 4. (업무자동화) 보고서 작성 Agent (web-search, retriever, file, image-generation)

- RAG + Image Generator Agent (보고서 작성)
- 문서 기반 검색 도구 (Retriever) + 웹 검색 도구(Tavily) + DALL-E image generator를 통해 이미지 생성

```python
# retriever_tool 에게 어떤 내용을 포함할지 알려줌
documnet_prompt = PromptTemplate.from_template(
    "<document><content>{page_content}</content><page>{page}</page><filename>{source}</filename></document>"
)

retriever_tool = create_retriever_tool(
    retriever,
    name="pdf_search",
    description="use this tool to search for information in the PDF file",
    document_prompt=document_prompt,
)

...

dalle = DallEAPIWrappter(model="dalle-3", size="1024x1024", quality="standard", n=1)

@tool
def dalle_tool(query):
    """use this tool to generate image from text"""
    return dalle.run(query)

...


prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            """
            You are a helpful assistant.
            You are a professional researcher.
            You can use the pdf_search tool to search for information in the PDF file.
            You can find further information by using search tool.
            You can use image generation tool to generate image from text.
            Finally, you can use file management tool to save your research result into files.
            """
        ),
        ...
    ]
)

...

result = agent_with_chat_history.stream(
    {
        "input":
        """
        삼성전자가 개발한 생성형 AI 와 관련된 유용한 정보들을 PDF 문서에서 찾아서 bullet point로 정리해 주세요.
        한글로 작성해주세요.
        다음으로는 'report.md' 파일을 새롭게 생성하여 정리한 내용을 저장해주세요.

        #작성방법:
        1. markdown header 2 크기로 적절한 제목을 작성하세요.
        2. 발췌한 PDF 문서의 페이지 번호, 파일명을 기입하세요. (예시: page 10, filename.pdf)
        3. 정리된 bullet point를 작성하세요.
        4. 작성이 완료되면 파일을 'report.md'에 저장하세요.
        5. 마지막으로 저장한 'report.md' 파일을 읽어서 출력해주세요.
        6. 'report.md' 파일을 열어서 기존의 내용을 읽고, 웹 검색으로 찾은 정보를 이전에 작성한 형식에 맞춰 뒷 부분에 추가해 주세요.
        7. 파일 내용에 어울리는 이미지를 생성하고, 생성한 이미지 url을 markdown 형식으로 보고서의 가장 상단에 추가히세요.
        """
    },
    ...
)
```

> 위에서는 한 번에 여러 지시를 내렸지만, 지시를 나누면 좀 더 완성도 높은 결과물을 얻을 수 있음

## 5. [프로젝트] CSV 파일 기반 데이터 분석 Agent

- csv 파일을 업로드하고, 행의 갯수, dataframe 분석, 차트 그리기등을 요청하는 프로젝트
- message type을 text, image, code, dataframe으로 정의하고, 메시지 타입에 따라 웹에서의 요소출력을 다르게 가져감
