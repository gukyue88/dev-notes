# Pandas 데이터 시각화

## matplotlib 사용법

### plt 바로 사용하기! (하나의 그림판에다가만 사용)

```python
import matplotlib.pyplot as plt

plt.stype.use('ggplot;) # ggplot 스타일 서식 적용
print(plt.style.available) # 사용 가능한 스타일 목록 조회

plt.title("차트제목") # 제목설정
plt.xlabel("레이블") # x축 레이블 설정, size=숫자 옵션으로 글씨크기 조절 가능
plt.ylabel("레이블") # x축 레이블 설정, size=숫자 옵션으로 글씨크기 조절 가능

plt.figure(figsize=(14, 5)) # figure 사이즈 가로 14 * 세로 5 지정
plt.xticks(rotation='vertical') # x축 눈금라벨 회전하기, rotation=숫자 가능 : 반시계반향 기울임 각도, size=숫자 글씨크기 조절

plt.legend(labels=[범례리스트], loc='best') # 범례 설정, fontsize=숫자 : 글씨크기 조정
plt.annotate() # 설명 덧붙이는 부분이지만 일단 생략.. 필요시 판다스책(p.118) 참고

plt.show() # 변경사항 저장하고 그래프 출력
```

### 화면 분할하여 그래프 여러 개 그리기

plt 바로 사용과 axe객체 얻어서 사용에서 함수 차이가 좀 있어보임

-   여러 개의 axe객체를 만들고 분할된 화면마다 axe객체를 하나씩 배정하는 개념

figure의 구성

![part of figure](https://matplotlib.org/3.3.2/_images/anatomy.png)

```python
fig = plg.figure(figsize=(10, 10)) # axe가 없는 그림틀을 만든다. (size 10 * 10)
ax1 = fig.add_subplot(2, 1, 1) # figure에 2행 1열 중 1번째에 axe를 할당한다.
ax2 = fig.add_subplot(2, 1, 2) # figure에 2행 1열 중 2번째에 axe를 할당한다.

# 이후 ax1,2는 생략하고 ax로 함수 설명...
ax.plot(sr 또는 df) # ax2에 df 데이터 지정
ax.set_ylim(최소값, 최대값) # y축의 범위 설정
ax.set_xticklabels(레이블, rotation=숫자) # x축의 눈금 라벨 지정 및 회전
as.set_xlabel(레이블) # x축 레이블 지정
as.set_xlabel(레이블) # y축 레이블 지정
ax.tick_params(axis="X", labelsize=10) # X축 눈금라벨크기 지정
ax.tick_params(axis="Y", labelsize=10) # Y축 눈금라벨크기 지정
ax.set_title("제목") # 제목 변경
ax.legend(레이블, loc='best') # 범례설정

plt.show() # 변경사항 저장하고 출력
```

### 동일한 그림(axe객체)에 여러개의 그래프를 추가하는 것도 가능

```python
fig = plt.figure(figsize=(20,5)) # axe가 없는 그림틀을 만든다. (size 10 * 10)
ax = fig.add_subplot(1,1,1) # figure에 1행 1열 중 1번째에 axe를 할당한다.

ax.plot(sr1)
ax.plot(sr2)
ax.plot(sr3)

plt.show()
```

## Seaborn

```python
# figure 사이즈 조정
f, axs = plt.subplots(figsize=(20, 4))
```

추가 참조 : [python-graph-gallery.com](https://python-graph-gallery.com/)
