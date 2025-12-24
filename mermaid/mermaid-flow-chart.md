# Mermaid Flow chart (순서도)

## 시작

```
%% flowchart Left to Right
flowchart LR

%% flowchart Top to Down
flowchart TD
```

- `graph` vs `flowchart` : 둘 다 flowchart이나, flowchart가 최신 버전이고 좀 더 빠르고 복잡한걸 더 잘 표시한다고 함

## 도형 모양

```
%% 사각형 (default) : 프로세스 (일반적인 작업, 명령, 계산)
ID[text]

%% 라운드 사각형 : 시작/종료
ID(text)

%% 마름모 : 판단 (조건 분기)
ID{text}

%% 평행사변형 : 데이터 입출력
ID[/text/]
```

## 선 종류

```
%% 실선 화살표 : 표준 흐름
A --> B

%% 점선 화살표 : 분기 흐름
A -.-> B

%% 텍스트 포함 : 조건부 흐름에 조건 넣을 때
A -- 텍스트 --> B

%% 굵은 화살표 : 주요 경로
A ==> B
```
