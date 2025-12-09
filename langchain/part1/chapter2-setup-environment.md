# Chapter 2. 환경 설정

## 1. 환경 설치 (Windows)

#### git 설치

- Add a Git Bash Profile to Windows Terminal 체크 후 설치

#### pyenv 설치 (파이썬을 버전별로 관리해주는 툴)

powershell 관리자 권한으로 실행 후 아래 내용 실행

```
# pyenv-win 클론
git clone https://github.com/pyenv-win/pyenv-win.git "$env:USERPROFILE\.pyenv"

# 환경변수 추가
[System.Environment]::SetEnvironmentVariable('PYENV', $env:USERPROFILE + "\.pyenv\pyenv-win\", "User")
[System.Environment]::SetEnvironmentVariable('PYENV_ROOT', $env:USERPROFILE + "\.pyenv\pyenv-win\", "User")
[System.Environment]::SetEnvironmentVariable('PYENV_HOME', $env:USERPROFILE + "\.pyenv\pyenv-win\", "User")
[System.Environment]::SetEnvironmentVariable('PATH', $env:USERPROFILE + "\.pyenv\pyenv-win\bin;" + $env:USERPROFILE + "\.pyenv\pyenv-win\shims;" + [System.Environment]::GetEnvironmentVariable('PATH', "User"), "User")

# powershell 종료 후 다시 열고, pyenv 동작 확인
pyenv
```

#### python 설치

```bash
# 파이썬 3.11 버전 설치
pyenv install 3.11

# 3.11 버전을 전역으로 설정
pyenv global 3.11

# 파이썬 버전 확인
python --version
```

#### Poetry 설치 (파이썬 패키지 관리 도구)

LangChain을 사용시 여러 패키지를 많이 사용하는데, 설치시 패키지별 의존성을 잘 지키기 위해 poetry로 만들어서 배포함

```bash
# Poetry 설치
pip3 inatll poetry
```

#### 실습코드 다운로드

```bash
# Documents로 경로를 맞춤
cd Documents
git clone https://github.com/teddylee777/langchain-kr

cd langchain-kr
poetry shell # poetry 로 가상환경을 실행함
petry update # 해당 가상환경에 패키지들을 설치
```

- langchain-kr repository 안에 패키지 설치 내용이 담겨있는 poetry 파일이 있음

#### Visual Studio Code 설치

- 다운로드 받은 실습코드를 열기
- Extensions
  - python (by Microsoft)
  - jupyter (by Microsoft)
- 우측 상단에 python 버전을 선택 > Select Another Kernel > Python Environment > langchain-kr-xxxx-py3.11을 선택
  - 안뜨는 경우, Visual Studio Code 껏다 켜기
  - 이 부분은 ipynb 파일을 바꿀때마다 지정해줘야하는 번거로움이 있긴함

#### 01-Basic/01-OpenAI-APIKey.ipynb 실행

- `.env_sample` 파일을 복사하여 `.env` 을 생성
  - `OPENAI_API_KEY`만 남겨두고 나머지 삭제
- `01-Basic/01-OpenAI-APIKey.ipynb` 실행

## 2. OpenAI API 키 발급 및 설정

- 사이트 접속 : https://platform.openai.com/docs/overview
- Sign up 눌러 회원가입 진행
- 세팅(톱니바퀴 아이콘) > Billing > Payment methods > 신용카드 등록
- 세팅 > Billing > Overview > Add to credit balance > 최소 달러가 5$ 임
  - 실습할 때마다 위의 balance가 차감됨
- 세팅 > Limits > Usage limit > Set a montyly budget > 과도한 요금이 나가지 않게 설정
  - Set an email notification threshold : 어느 정도 이상 되면 이메일로 noti를 주는 기능
- 프로필 사진 > Your profile > API keys > Create new secret key
  - Name : 아무 이름이나 상관없음, 알아보기 쉽게만 지정
  - Project : Default project
  - Permissions : All
  - Create secret key 버튼 클릭
- 나온 키를 복사하여 `.env`의 `OPENAI_API_KEY={복사한내용}` 채워주기
- 다시 `01-Basic/01-OpenAI-APIKey.ipynb`를 실행해서 API KEY가 잘 설정 되었는지 확인하기
- `01-Basic/02-OpenAI-LLM.ipynb`를 실행해서 API Key를 사용해서 질문에 대한 답변이 잘 오는지 확인하기

## 3. LangSmith 키 발급 및 설정

#### LangSmith

- 코드 실행을 추적해주는 시스템
- 질의를 하고 답변을 받을 때의 과정들이 데이터베이스에 전부 다 남아서 추적을 할 수 있도록 만들어 줌
- 나중에 RAG 시스템을 만들다보면 파이프라인이 점점 복잡해지게 되는데, 이 과정을 트래킹하기 위해 LangSmith를 사용함

#### 키 발급

- https://smith.langchain.com 접속 후 회원가입, 로그인 진행
- Settings(왼쪽 톱니바퀴이아이콘) > API Keys > Create API Key
  - Description : 아무거나 입력
  - Key Type : Personal Access Token
  - Create API Key 버튼 클릭
- 해당 API Key 복사 후, `.env` 파일에 아래 내용 추가

```
LANGCHAIN_TRACING_V2=true
LANGCHAIN_ENDPOINT=https://api.smith.langchain.com
LANGCHAIN_API_KEY={위에서 복사한 키}
LANGCHAIN_PROJECT=FASTCAMPUS
```

- `LANGCHAIN_TRACING_V2` : 프로젝트를 추적할지 여부 설정
- `LANGCHAIN_PROJECT` : LANG CHAIN 공용 프로젝트 이름?

#### 01-Basic/02-OpenAI-LLM.ipynb 실행

- .env 파일을 바꿨기 떄문에, Restart로 커널 재시작을 한 다음 실행
- 이제 langsmith 웹페이지로 들어가서 Project(앱 모양 아이콘) > `FASTCAMPUS` (위에서 설정한 프로젝트명) 을 클릭
- 그럼, 우리가 llm 사용시 몇 개의 토큰을 사용했는지, 몇초가 걸렸는지, input은 뭐였고, output은 뭐였는지, 어떤 모델을 썻는지 등이 나오게 됨

> langsmith는 최근에 유료로 전환 되긴 했지만, 무료량이 넉넉해서 실습에는 문제 없을 것임

## 4. Visual Studio Code - User Settings

- ctrl + shift + p > user settings (JSON) 선택
- JSON 파일이 열리면 안에 있는 내용을 지우고 아래 내용으로 대체하고 저장
  - JSON 내용 : https://gist.github.com/teddylee777/e9d9845fabfd3379dfcd7ffbc37d1286

#### extension 설치

- Black Formatter (by Microsoft) : python formatter임
