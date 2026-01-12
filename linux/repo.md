# repo

## repo 란?

구글이 안드로이드(AOSP) 개발을 위해 Git 위에 덧씌워 만든(Wrapper) 파이썬 스크립트 도구

## 주요 개녕
- repo (도구) : 여러 개의 git 저장소를 마치 하나의 거대한 저장소처럼 다루게 해주는 관리자
- manifest (설계도) : 전체 프로젝트의 구성 정보를 담은 xml (`repo init` 할 때 지정함)
- project (하위 저장소) : 실제 소스 코드가 들어있는 개별 git 저장소들

## 주요 명령어

#### `repo init` : 초기화

manifest를 모아놓은 저장소에 있는 xml 중 특정 xml 기준으로 repo를 구성함 (`.repo` 디렉토리 생성)

- 해당 xml 안에서는 또다른 xml을 include 할 수 있음

```bash
# -u : manifest URL 지정 (git 저장소의 url)
# -b : 특정 브랜치 지정 (예: main)
# -m : 특정 xml 파일 지정 (기본값 : default.xml)
repo init -u https://github.com/my-project/manifest.git -b main
```

#### `repo sync` : 동기화

실제 소스 코드를 **다운로드** 하거나, **최신 상태로 업데이트**

```bash
# -j4 : 4개의 병렬 프로세스로 다운로드 (속도 향상)
# -c : 현재 브랜치만 다운로드 (용량 절약)
repo sync -j4 -c
```

#### `repo start` : 브랜치 생성

여러 git 저장소에 동시에 브랜치를 만듦

```bash
# "new-feature"라는 브랜치를 모든 프로젝트(--all)에서 생성
repo start new-feature --all
```

#### `repo status` : 상태 확인

내가 수정한 파일이 있는 곳만 콕 집어서 보여줌

```bash
repo status
```

#### `repo forall` : 일괄 명령

모든 하위 저장소에서 특정 쉘 명령어를 실행

```bash
# 모든 저장소에서 "git reset --hard"를 실행하여 변경사항 초기화
repo forall -c "git reset --hard"
```