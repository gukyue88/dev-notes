# Chapter 1. Document Loader

## 1. Document loader의 종류, 기본 구조, Document 구조

- RAG 단계 (Load -> Split -> Embed -> Store) 중 load에 대한 부분임
- 주로 쓰는 로더들을 다루고, 종류가 많으니 각자 필요에 맞게 찾아서 써야함
- langchain에서 제공하는 loader들은 BaseLoader를 상속받아 사용하고 있고, 인터페이스가 잘 정형화 되어있음
  - 또, 커스텀 로더를 만들 때도 BaseLoader를 상속받고 구현해야하는 부분을 구현하면 됨

## 2. Document, Document Loader의 구조 이해하기

#### Document

> langchaing에서 doc, ppt, pdf 파일등을 불러오면 전부 Document 타입으로 변환된다고 보면 됨

- Document : LangChain의 기본 문서 객체임
  - page_content(str) : 문서의 내용
  - metadata(dict) : 파일명, 페이지번호 등 메타데이터

```python
from langchain_core.documents import Document

document = Document(page_content="안녕하세요. 이건 랭체인의 도큐먼트 입니다.", metadata={"source": "TeddyNote"})

print(document.metadata["source"])
document.metadata["page"] = 1
document.metadata["author"] = "Teddy"
```

#### 자주 쓰이는 함수

- `load()` : 젠체 문서를 로드하여 반환함, return 타임은 List[Document] 형태임
- `load_and_split()` : 문서를 로드 하면서 chunking을 동시에함, return 타임은 List[Document] 형태임
- `lazy_load()` : 제너레이터 방식으로 로딩을 하는 방식, 문서양이 너무 커서 메모리 공간이 모자랄 때 사용
- `aload()` : 비동기 방식의 load()

#### Tip

- metadata에 페이지에 대한 summary를 만들어서 넣어놓는 기법 사용 가능
- metadata에 태거를 만들어서 태깅을 해주면(?) 태그들을 넣어놓는 기법 사용 가능
  - 위 정보로 나중에 검색 단계에서 바로 조회 가능

## 3. PDF 로더

- 여러가지 PDF Loader가 있지만, 벤치마크 상으로는 `PDFPlumberLoader`가 가장 좋긴함
- 각 로더 마다 지원하는 기능이 조금씩 달라서 사용용도에 따라 loader를 다르게 하는 방법도 있음
- OCR을 지원한다던지, 속도가 중요하다던지, html 형태로 가져온다던지, Diretory내 순환하면서 가져온다던지 등등

## 4. HWP 로더

- langchain에는 인테그레이션이 안되어 있음
- llama에는 인테그레이션 되어있어서 Teddy님에 따로 구현하여 공유함

## 5. CSV 로과과 효과적인 파일 처리 방법

- csv는 쉼표로 값을 구분하는 구분된 텍스트 파일임

#### 일반적인 CSV 파일 Load

```python
# 일반적인 방법 : page_content를 확인해보면 row를 페이지 처럼 식해 각 page_content에 하나의 row가 들어감
# 이렇게하면 llm이 잘 csv 파일 내용을 잘 인지하지 못하는 경우가 발생함
laoder = CSVLoader(file_path="titanic.csv")
docs = loader.load()
```

#### 효과적인 CSV 파일 Load

- 모든 정보를 xml 형태로 값을 감싸서 각 정보를 llm에 문서 검색에 값의 의미를 더 잘 이해할 수 있도록 하자

#### DataFrameLoader

- DataFrame 타입으로 가져옴

> DataFrameLoader도 좋지만, Teddy님이 가장 추천하는건 xml 형태로 만들어서 넣어주는 것임

## 6. WebBase 로더

- `WebBaseLoader` : 웹 기반 문서를 로드하는 로더
- `bs4` : 웹 페이지 파싱
  - `bs4.SoupStrainer`를 사용하여 파싱할 요소를 지정
  - `bs_kwargs` 매개변수를 사용하여 `bs4.SoupStrainer`의 추가적인 인수를 지정함

## 7. Directory 로더

- `DirecotyLoader` : 디스크에서 파일을 읽어 `Document` 객체로 변환하는 기능
- 와일드카드 패턴을 포함할 수 있음
- 멀티스레딩 옵션을 줘서 load를 할 수 있음
- 특정 로더 지정을 가능함

## 8. Upstage LayoutAnalysis 로더

- `UpstageLayoutAnalysisLoader` : Upstgage AI에서 제공하는 문서 분석 도구
- PDF, 이미지 등 다양한 형식의 문서에서 레이아웃 분석 수행
- OCR 기능 지원 (선택적)
- Upstage API 키 필요
- 제외하고 싶은 요소(header, footer등)을 지정 가능
- 출력 형식을 html로도 지정가능
  - llm이 이걸 보고 좀 더 분석하기가 좋음

## 9. LlamaParser 로더

- `LlamaParse`는 LlamaIndex에서 개발한 문서 파싱 서비스
- PDF, Word, PowerPoint, Excel 등 다양한 문서 형식 지원
- 자연어 지시를 통한 맞춤형 출력 형식 제공
- 복잡한 표와 이미지 추출 가능
- JSON 모드 지원
- 외국어 지원
- API 키 별도 필요
- 멀티모달 지원
