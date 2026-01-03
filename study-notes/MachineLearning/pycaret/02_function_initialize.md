이 글은 PyCaret홈페이지의 내용을 다루고 있습니다.

원문 : [Setting up Environment](https://pycaret.org/setup/)

# 설정 작업

pycaret의 머신러닝을 시작하기 전에, 환경을 설정하는 것이 반드시 필요합니다. 이 작업은 단지 두 가지 단계로 구성되어 있습니다.

## 1단계 : 관련 모듈 임포트

원하는 종류의 머신러닝에 따라, 아래 6개 모듈중 하나를 임포트 합니다. 모듈을 임포트하면 작업을 위한 환경이 준비됩니다. 예를들어, 만약 **Classificaton** 모듈을 가져오면, 환경은 분류를 수행하기 위한 환경으로 설정됩니다.

```python
# 아래 모듈 중 하나만 import 합니다.
from pycaret.classification import * # Classification
from pycaret.regression import * # Regression
from pycaret.clustering import * # Clustering
from pycaret.anomaly import * # Anomaly Detection
from pycaret.nlp import * # Natural Language processing
from pycaret.arules import * # Association Rule Mining
```

## 2단계 : 설정 초기화

어떤 머신러닝을 시작하던지, PyCaret의 모든 모듈에 공통적으로 **설정(setup)**은 가장 처음 할 일이자 유일한 작업입니다. 몇 가지 기본 작업을 내부적으로 수행하는 것 외에도, 머신러닝 성능을 평균적으로 향상시키는 고급 솔루션 전처리 기능을 제공합니다. 이 글에서는 설정 기능의 필수 부분만 다룹니다. 전처리 기능 관련 내용은 다른 글에서 다룹니다. 아래는 PyCaret에서 수행하는 필수 기본 작업입니다.

**데이터 타입 예상** : PyCaret은 올바른 데이터 유형을 결정하는 것부터 시작합니다. **설정(setup)**은 데이터 타입 예상한 뒤, 이를 기반으로 내부 알고리즘을 이용해 'ID 및 날짜 열 무시, 범주형 인코딩, 결측값 채우기'등의 작업을 수행합니다. **설정(setup)**이 실행되면, 모든 feature와 예상 데이터 타입을 대화상자형태로 보여줍니다. 데이터 타입 예상은 일반적으로 정확하긴하지만 사용자의 검토가 필요합니다. 모든 데이터 타입이 올바르게 예상된 경우 Enter키를 눌러 계속하거나 '종료'를 입력하여 중지시킬 수 있습니다.

![setup dialog](https://i0.wp.com/pycaret.org/wp-content/uploads/2020/02/setup1-1.png?resize=599%2C346&ssl=1)

만약 데이터 타입 예상이 틀렸을 경우 **quit**를 입력 후 Enter키를 입력합니다. 데이터 타입이 맞지 않은 경우, **setup** 명령어에 파라미터를 사용해 강제로 지정할 수 있습니다. (범주형 : **categorical\_feature**, 숫자형 : **numeric\_feature**) 또한, 무시하고 싶은 feature 들은 **ignore\_features** 파라미터로 설정할 수 있습니다.

-   만약 PyCaret 예상한 데이터 타입을 확인작업 없이 수행하려면, **silent** 값을 **True**로 넘겨주면 됩니다. PyCaret의 데이터 타입 예상이 맞다고 확실하거나 **numeric\_feature**나 **categorical\_feature** 파라미터를 사용한 것이 아니라면 추천하지 않습니다.

**Data Cleaning과 준비** : **setup** 함수는 자동으로 결측값 채우기와 범주형 인코딩을 수행합니다. 숫자형 feature의 결측값에 대해서는 default로 평균값을 사용합니다. 범주형 feature의 결측값에 대해서는 default로 가장 많이 출현한 값 또는 최빈값을 사용합니다. default값을 변경하기 위해서는 **numeric\_imputation**과 **categorical\_imputation** 파라미터를 사용할 수 있습니다. 분류 문제에 대해서 target이 숫자형이 아니라면, setup은 타겟인코딩을 수행합니다.

**Data Sampling** : 만약 표본의 크기가 25,000보다 크다면, PyCaret은 자동적으로 예비선형모델(a preliminary linear model)을 구축하고 표본 크기에 따라 모델의 성능을 시각화해줍니다. 해당 도표를 사용하여 표본 크기가 증가함에 따라 모델의 성능이 증가하는지 평가할 수 있습니다. 그렇지 않은 경우 실험의 효율성과 성능을 위해 더 작은 표본 크기를 선택할 수 있습니다. 아래는 PyCaret의 데이터셋 저장소의 'bank' 데이터의 예입니다. 해당 데이터셋의 크기는 45,211개 입니다.

![bank datasets](https://i2.wp.com/pycaret.org/wp-content/uploads/2020/02/setup2.png?resize=558%2C333&ssl=1)

**Train/Test 분리** : setup 함수는 train/test 분리 작업 또한 수행합니다. (분류 문제를 위해 계층화됨) train/test 분리의 Default 비율은 70:30 입니다. 하지만 **train\_size** 파라미터를 통해 변경할 수 있습니다. **Train** 셋에 대해서만 k-fold 교차검증을 사용해 머신러닝 평가와 하이퍼파라미터 최적화 평가가 수행됩니다.

**Seed로 Session ID 할당** : **session\_id** 파라미터를 지정하지 않으면, session id는 pseudo 랜덤 수가 됩니다. PyCaret은 이 id를 모든 함수에 seed로 제공하여 각자 머신러닝이 다른 random seed를 사용해 생길 수 있는 문제를 방지합니다. 이는 또한 나중에 동일하거나 다른 환경에서의 재현이 가능하도록 합니다.

## Classification 예제

```python
# Importing dataset
from pycaret.datasets import get_data
diabetes = get_data('diabetes')

# Importing module and initializing setup
from pycaret.classification import *
clf1 = setup(data = diabetes, target = 'Class variable')
```

## Regression 예제

```python
# Importing dataset
from pycaret.datasets import get_data
boston = get_data('boston')

# Importing module and initializing setup
from pycaret.regression import *
reg1 = setup(data = boston, target = 'medv')
```

## Anomaly 예제

```python
# Importing dataset
from pycaret.datasets import get_data
anomalies = get_data('anomaly')

# Importing module and initializing setup
from pycaret.anomaly import *
ano1 = setup(data = anomalies)
```

## Natural Language Processing 예제

```python
# Importing dataset
from pycaret.datasets import get_data
kiva = get_data('kiva')

# Importing module and initializing setup
from pycaret.nlp import *
nlp1 = setup(data = kiva, target = 'en')
```

## Association Rule Mining 예제

```python
# Importing dataset
from pycaret.datasets import get_data
france = get_data('france')

# Importing module and initializing setup
from pycaret.arules import *
arules1 = setup(data = france, transaction_id = 'InvoiceNo', item_id = 'Description')
```
