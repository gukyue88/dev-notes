# Panas 커닝페이퍼

## 출력옵션 변경

```python
pd.set_option('display.max_row', 500)
pd.set_option('display.max_columns', 500)
```

## 기본 기능

```python
# Series 객체는 sr, DataFrame 객체는 df로 표현

# Series index, values 확인
sr.index
sr.values

# Series 데이터 접근
sr[정수형인덱스]
sr["이름인덱스"]

# DataFrame index, columns 확인
df.index
df.columns

# DataFrame index, columns 변경
df.index = 행인덱스_ARRAY
df.columns = 열이름_ARRAY

# DataFrame index, columns 일부 변경 : rename
df = df.rename(index={기존인덱스:새인덱스, ...})
df = df.rename(columns={기존열이름:새열이름, ...})

# 삭제
df = df.drop(행인덱스 또는 배열, axis=0) # 행 삭제
df = df.drop(열이름 또는 배열, axis=1) # 열 삭제

# 행으로 접근하기
df.iloc[정수형인덱스]
df.loc["이름인덱스"]
df.iloc[정수형인덱스:정수형인덱스] # 마지막 행 미포함, :: 사용하여 사용하여 범위 슬라이싱도 가능
df.loc["이름인덱스":"이름인덱스"] # 마지막 행 포함, :: 사용하여 사용하여 범위 슬라이싱도 가능

# 열로 접근하기
df["열이름"] # 시리즈 객체 반환
df.열이름 # 시리즈 객체 반환
df[["열이름1", "열이름2", ...]] # 데이터 프레임 반환 (열이름이 하나라도)

# 원소 선택
df.loc[행이름인덱스, "열이름"]
df.iloc[행정수형인덱스, 열번호]

# 행 추가
df.loc["새로운 행 이름"] = 데이터값 또는 배열

# 행, 열 위치 바꾸기
df = df.T
df = df.transpose()

# 특정 열을 index로 활용
df = df.set_index("열이름" 또는 ["열이름1", "열이름2", ...])

# 행 인덱스 재배열
df = df.reindex(새로운 인덱스 배열) # 새로운 행 인덱스에는 NaN값이 채워짐
df = df.reindex(새로운 인덱스 배열, fill_value=값) # 특정 값과 함께 생성

# 행 인덱스 초기화
df = df.reset_index() # 정수형인덱스로 초기화

# 행 인덱스 기준 정렬
df = df.sort_index()
df = df.sort_index(ascending=True) # 오름차순 정렬
df = df.sort_index(ascending=False) # 내림차순 정렬

# 특정열의 데이터 값을 기준으로 정렬
df = df.sort_values(by="열이름", ascending=True or False)

# 시리즈 연산
# 시리즈 + 숫자 연산 : 모든 원소에 숫자 연산
sr3 = sr1 + 5
sr3 = sr1 - 5
sr3 = sr1 * 5
se3 = sr1 / 5
# 시리즈 + 시리즈 연산 : 순서 상관없이 index 기준으로 연산, 연산이 안되는 경우 NaN
sr3 = sr1 + sr2
sr3 = sr1 - sr2
sr3 = sr1 * sr2
se3 = sr1 / sr2
# 시리즈 + 시리즈 연산2 : NaN값 대신 다른 vlaue로 채움
sr3 = sr1.add(sr2, fill_value=숫자)
sr3 = sr1.sub(sr2, fill_value=숫자)
sr3 = sr1.mul(sr2, fill_value=숫자)
sr3 = sr1.div(sr2, fill_value=숫자)

# 데이터프레임 연산
# 데이터프레임 + 숫자 연산 : 모든 원소에 적용
df3 = df1 + 5
df3 = df1 - 5
df3 = df1 * 5
df3 = df1 / 5
# 데이터프레임 + 데이터 프레임 연산 : 같은 행열끼리 연산, 불가능이면 NaN
df3 = df1 + df2
df3 = df1 - df2
df3 = df1 * df2
df3 = df1 / df2
```

## 파일 읽고 쓰기

```python
# csv 파일 읽기
# header=0 : 열이름의 행을 지정, 디폴트 : 0, None으로도 가능
# index_col=False : 행의인덱스가 되는 column을 지정, 디폴트 : False, False인 경우 정수형인덱스 사용
# sep=',' : 구분자 지정, 디폴트 : ','
pd.read_csv("파일")
# 기타 읽기
pd.read_excel() # 엑셀파일
pd.read_json() # json
pd.read_html() # html 내 table들을 dataframe 리스트 형태로 가져옴 
pd.read_sql() # sql

# csv 파일 쓰기
# index=True : csv파일 생성시 인덱스 포함 여부 설정, 디폴트 : True
pd.to_csv("파일")
# 기타 쓰기
pd.to_json()
pd.to_excel() # ExcelWrite 사용시 여러 dataframe을 sheet단위로 저장 가능
```

## 데이터 살펴보기

```python
# 데이터 일부 살펴보기
# n=5 : 살펴볼 행의 갯수, 디폴트 : 5
df.head() # 첫부분 일부 행
df.tail() # 마지막부분 일부 행

df.shape # 데이터 프레임 크기
df.info() # 기본정보 : 클래스유형, 행인덱스 구성, 열이름과 종류 개수, 각 열의 자료형과 개수, 메모리 할당량

df.dtypes # 각 열의 자료형 확인
df.열이름.dtypes # 해당 열의 자료형 확인
df.describe() # 통계 정보 요약
df.count() # 각 열의 유효한 값을 계산! (유효한 값이라는 것이 중요!!!), info()와 달리 시리즈 객체 반환
df["열이름"].value_counts() # 해당 열의 고유값 갯수 확인

# 통계함수 적용
# mean() : 평균값
# median() : 중간값
# max() : 최대값
# min() : 최소값
# std() : 표준편차
# corr() : 두 열간의 상관 계수, 2개씩 짝지어서 상관계수를 모두 구함
df.통계함수() # 모든 열의 통계값을 시리즈 객체로 반환
df["열이름"].통계함수()

# 내장 그래프 도구 : 판다스는 Matplotlib를 일부 내장함
# plot 함수내 kind 옵션으로 종류 선택
df.plot() # 선그래프
df.plot(kind='line') # 선그래프
df.plot(kind='bar') # 수직 막대 그래프
df.plot(kind='barh') # 수평 막대 그래프
df.plot(kind='his') # 히스토그램
df.plot(kind='box') # 박스플롯
df.plot(kind='kde') # 커널 밀도 그래프
df.plot(kind='area') # 면적 그래프
df.plot(kind='pie') # 파이 그래프
df.plot(kind='scatter') # 산점도 그래프
df.plot(kind='hexbin') # 고밀도 산점도 그래프
```

판다스는 Numpy 기반으로 만들어서 **파이썬 자료형이 아닌 Numpy 자료형을 따른다.**

| 판다스 자료형 | 파이썬 자료형 |
| :-- | :-- |
| int64 | int |
| float64 | float |
| object | string |
| datetime64, timedelta64 | 없음 |

## 데이터프레임 응용

```python
# 대체하기
sr = sr.replace(원래값, 대체값)

# 함수 매핑
sr.apply(매핑함수) # 함수 인자가 series의 각 원소, 반환 : 시리즈
df.applymap(매핑함수) # 함수 인자가 dataframe의 각 원소, 반환 : 데이터 프레임
df.apply(매핑함수, axis=0) # 함수의 인자가 시리즈(열), 반환 : 매핑함수에 따라 다름
df.apply(매핑함수, axis=1) # 함수의 인자가 각 행, 반환 : 시리즈
df.pipe(매핑함수) # 함수인자가 데이터 프레임, 반환 : 매핑함수에 따라 다름

# 열 순서 변경
df = df[재구성한열리스트]

# 열 분리 예제
df['연월일'] = df['연월일'].astype('str')
dates = df['연월일'].str.split(-)
df['연'] = dates.str.get(0)
df['월'] = dates.str.get(1)
df['일'] = dates.str.get(2)

# 필터링
# 시리즈 객체에 조건식을 적용하면 참/거짓으로된 Boolean 시리즈를 반환한다.
# Boolean 시리즈를 데이터프레임에 적용하면 원하는 조건을 만족하는 행들만 선별할 수 있다.
mask = (titanic.age >= 10) & (titanic.age < 20)
df_teenage = titanic.loc[mask,:]
df_teenage = titanic.loc[mask,['age', 'sex']]

# isin(리스트)
df['열'].isin(리스트) # 리스트 안에 해당하는 값이 있는지 여부를 Boolean 시리즈로 반환
```

## 데이터 프레임 합치기

```python
# 데이터 프레임끼리 구성형태와 속성이 균일할 때, concat을 사용 - 형태를 유지하면서 붙인다는 개념
# 옵션
# axis=0 : 행 추가식으로 연결, 1:열 추가식으로 연결
# join='outer' : 열 이름의 합집합으로 새로운열 구성, 'ineer' : 열 이름의 교집합으로 구성
# ignore_index : True - 새로운 행 인덱스로 설정
# concat을 사용하면 데이터프레임과 시리즈, 시리즈와 시리즈 연결도 가능하다!
pd.concat(데이터프레임리스트)

# merge : concat과는 다르게 어떤 두 프레임을 병합하는 개념
pd.merge(df_left, df_right, how='inner', on=None)

# join : merge 함수 기반이라 비슷, 행 인덱스 기준으로 결합하는 것이 기본이나 on=keys를 설정하면 열 기준 결합도 가능
df1.join(df2, how='left')
```

## 그룹연산

특정 기준을 적용하여 몇 개의 그룹으로 분할하여 처리하는 것

3단계 분할, 적용, 결합으로 구성됨

그룹객체는 생각에.. 각 그룹별 key를 기준열이름으로 가지고 있고 (논리적으로) 안에 해당 행이 있는 collection 객체 정도로 보임

```python
grouped = df.groupby(기준열) # 기준열은 한 개 혹은 열리스트 가능

for key, group in grouped:
    print(key)
    print(len(group))
    print(group.head())

avg = grouped.mean() # 그룹별 각 열에 대한 평균값
group3 = grouped.get_group('키이름') # 해당 키 그룹 데이터프레임(?)

# 여러 열을 기준으로 그룹화
# titanic 예제 기준 열리스트가 ['class', 'sex']라면 grouped 객체 내 key값들은
# ('first', 'female')
# ('first', 'male')
# ('second', 'female')
# ('second', 'male')
# ('third', 'female')
# ('third', 'male')
grouped = df.groupby(기준이되는열의리스트)

# 그룹 연산 메소드 (적용-결합 단계)
# 그룹 객체에 다양한 연산 적용
# mean, max, min, sum, count, size, var, std, describe, info, first, last등
grouped.함수()

agg = grouped.agg(매핑함수) # 사용자 정의 함수 적용, 인자가 각 열이 되는 듯.. 확인필요..
agg = grouped.agg([함수1, 함수2, ...]) # 여러 함수 일괄 적용
agg = grouped.agg({'열1':['min', 'max'], '열2':'mean'}) # 각 열에 원하는 함수 적용

# agg()는 각 그룹별 함수 적용
# transform()은 그룹별 집계 대신 연산결과를 처리하여 데이터프레임형태로 반환함
df = grouped.transform(매핑함수)

# 필터링
grouped.filter(조건식함수) # 조건이 참인 그룹만 남김!

# 함수매핑 apply() - 판다스객체의 개별원소를 일대일 매핑
grouped.apply(매핑함수)
agg_grouped = grouped.apply(lambda x: x.describe())

# 여러 열 groupby 이후, 함수 적용되면 DataFrame이 반환되는데 얘네들은 multi-index를 가짐..
gdf = grouped.mean()
# gdf는 멀티인덱스를 가짐..

# pivot_table()
# aggfunc, values내 리스트도 가능
pdf = pd.pivot_table(df,            # 피벗할 데이터프레임
                    index='열1',        # 행위치에 들어갈 열
                    columns='열2',    # 열 위치에 들어갈 열
                    values='열3',    # 데이터로 사용할 열
                    aggfunc='함수')    # 데이터 집계 함수

```

## Feature Engineering

```python
# 결측값 채우기
df.fillna(특정값) # 특적값으로 결측치 대체
df.fillna(method='ffill') # 앞 행의 값으로 대체, limit=숫자 옵션으로 채우는 갯수 제한
df.fillna(method='bfill') # 뒤 행의 값으로 대체, limit=숫자 옵션으로 채우는 갯수 제한
df.fillna(df.mean()) # 평균 값으로 채우기 - 시리즈로 각 열마다 줄 수 있는 듯.. dict로 원하는 열마다도 될 듯

# sql like 구현
mask = df["열"].str.startswith("문자열") # 문자열로 시작하는 열만 boolean sereies
df = df[mask]

# target mean encoding : feature_mean 형태로 열을 붙여줌..
def target_mean_encoding(df, feature, target):
    grouped_target_mean = df.groupby(feature)[target].mean()
    df[f"{feature}_mean"] = df[feature].map(grouped_target_mean)
    return df

df = target_mean_encoding(df, "타겟민인코딩필요한피처", "타겟")
```
