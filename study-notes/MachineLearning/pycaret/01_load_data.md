이 글은 PyCaret홈페이지의 내용을 다루고 있습니다.

원문 : [Getting Data](https://pycaret.org/get-data/)

# 데이터 불러오기

PyCaret의 모든 모듈은 panda Dataframe과 작동할 수 있다. 아래는 판다스의 기본기능인 **read\_csv** 를 통해 csv 파일을 불러오는 아래 예제입니다.

## panda를 사용하여 Data 불러오기

```python
import pandas as pd

data = pd.read_csv('file.csv')
```

## PyCaret 내부 Data 불러오기

PyCaret은 테스트를 위한 오픈소스 데이터셋 저장소를 온라인상으로 제공합니다. 이것들은 PyCaret의 github에서 온라인 제공되며, **pycaret.datasets** 모듈을 통해 바로 불러올 수 있습니다.

```python
from pycaret.datasets import get_data
data = get_data('juice')
```

## PyCaret Data 저장소

| Dataset | Data Types | Default Task | Target Variable | \# Instances | \# Attributes |
| :-- | :-- | :-- | :-- | :-- | :-- |
| anomaly | Multivariate | Anomaly Detection | None | 1000 | 10 |
| france | Multivariate | Association Rule Mining | InvoiceNo, Description | 8557 | 8 |
| germany | Multivariate | Association Rule Mining | InvoiceNo, Description | 9495 | 8 |
| bank | Multivariate | Classification (Binary) | deposit | 45211 | 17 |
| blood | Multivariate | Classification (Binary) | Class | 748 | 5 |
| cancer | Multivariate | Classification (Binary) | Class | 683 | 10 |
| credit | Multivariate | Classification (Binary) | default | 24000 | 24 |
| diabetes | Multivariate | Classification (Binary) | Class variable | 768 | 9 |
| electrical\_grid | Multivariate | Classification (Binary) | stabf | 10000 | 14 |
| employee | Multivariate | Classification (Binary) | left | 14999 | 10 |
| heart | Multivariate | Classification (Binary) | DEATH | 200 | 16 |
| heart\_disease | Multivariate | Classification (Binary) | Disease | 270 | 14 |
| hepatitis | Multivariate | Classification (Binary) | Class | 154 | 32 |
| income | Multivariate | Classification (Binary) | income >50K | 32561 | 14 |
| juice | Multivariate | Classification (Binary) | Purchase | 1070 | 15 |
| nba | Multivariate | Classification (Binary) | TARGET\_5Yrs | 1340 | 21 |
| wine | Multivariate | Classification (Binary) | type | 6498 | 13 |
| telescope | Multivariate | Classification (Binary) | Class | 19020 | 11 |
| glass | Multivariate | Classification (Multiclass) | Type | 214 | 10 |
| iris | Multivariate | Classification (Multiclass) | species | 150 | 5 |
| poker | Multivariate | Classification (Multiclass) | CLASS | 100000 | 11 |
| questions | Multivariate | Classification (Multiclass) | Next\_Question | 499 | 4 |
| satellite | Multivariate | Classification (Multiclass) | Class | 6435 | 37 |
| asia\_gdp | Multivariate | Clustering | None | 40 | 11 |
| elections | Multivariate | Clustering | None | 3195 | 54 |
| facebook | Multivariate | Clustering | None | 7050 | 12 |
| ipl | Multivariate | Clustering | None | 153 | 25 |
| jewellery | Multivariate | Clustering | None | 505 | 4 |
| mice | Multivariate | Clustering | None | 1080 | 82 |
| migration | Multivariate | Clustering | None | 233 | 12 |
| perfume | Multivariate | Clustering | None | 20 | 29 |
| pokemon | Multivariate | Clustering | None | 800 | 13 |
| population | Multivariate | Clustering | None | 255 | 56 |
| public\_health | Multivariate | Clustering | None | 224 | 21 |
| seeds | Multivariate | Clustering | None | 210 | 7 |
| wholesale | Multivariate | Clustering | None | 440 | 8 |
| tweets | Text | NLP | tweet | 8594 | 2 |
| amazon | Text | NLP / Classification | reviewText | 20000 | 2 |
| kiva | Text | NLP / Classification | en | 6818 | 7 |
| spx | Text | NLP / Regression | text | 874 | 4 |
| wikipedia | Text | NLP / Classification | Text | 500 | 3 |
| automobile | Multivariate | Regression | price | 202 | 26 |
| bike | Multivariate | Regression | cnt | 17379 | 15 |
| boston | Multivariate | Regression | medv | 506 | 14 |
| concrete | Multivariate | Regression | strength | 1030 | 9 |
| diamond | Multivariate | Regression | Price | 6000 | 8 |
| energy | Multivariate | Regression | Heating Load / Cooling Load | 768 | 10 |
| forest | Multivariate | Regression | area | 517 | 13 |
| gold | Multivariate | Regression | Gold\_T+22 | 2558 | 121 |
| house | Multivariate | Regression | SalePrice | 1461 | 81 |
| insurance | Multivariate | Regression | charges | 1338 | 7 |
| parkinsons | Multivariate | Regression | PPE | 5875 | 22 |
| traffic | Multivariate | Regression | traffic\_volume | 48204 | 8 |
