이 글은 PyCaret홈페이지의 내용을 다루고 있습니다.

원문 : [Tune Models](https://pycaret.org/tune-model/)

# 모델 튜닝하기

**tune\_model**을 사용하면 간단하게 모델의 하이퍼파라미터를 튜닝 할 수 있습니다. 이 함수는 custom 가능한 random grid search를 사용하여 estimator로 전달된 모델의 하이퍼파라미터를 튜닝합니다. 하이퍼파라미터 튜닝은 지도학습(classification 또는 Regression에서 target 변수에 자동으로 연결되는 목적함수(objective-function)이 필요합니다. 그러나 비지도학습(Clustering, Anomaly Detection, Natural Language Processing)의 경우 PyCaret은 **tune\_model**내 **supervised\_target** 파라미터를 사용하여 supervised tartge 변수를 지정하여 custom 목적함수를 정의할 수 있도록 합니다. 지도학습의 경우 이 함수는 학습된 모델 + k-fold 교차 검증 평가지표 점수 표를 반환합니다. 비지도학습의 경우, 이 함수는 학습된 모델만 반환합니다. 지도학습에서 사용되는 평가 지표는 아래와 같습니다.

-   Classification : Accuracy, AUC, Recall, Precision, F1, Kappa, MCC
-   Regression : MAE, MSE, RMSE, R2, RMSLE, MAPE

fold의 수는 **tune\_model**함수의 **fold** 파라미터를 사용해 변경할 수 있습니다. 기본값으로 **fold**는 10입니다. 모든 지표(metric)은 기본적으로 4자리 소수로 반올림되며, **tune\_model**의 **round** 파라미터를 사용해 변경할 수 있습니다. PyCaret 내 모델 튜닝 함수는 사전 정의된 search space의 randomized grid search 입니다. 그러므로 search space의 iteration 숫자와 관련있습니다. 기본적으로 이 함수는 tune\_model내에서 n\_iter 파라미터를 사용하여 변경할 수 있는 search space에 대해 10번의 무작위 iteration을 수행합니다. **n\_iter** 파라미터를 증가시키면 학습시간은 늘어날 수 있지만 종종 고도로 최적화된 모델이 생성됩니다. 최적화 할 지표는 **optimize** 파라미터를 사용하여 정의할 수 있습니다. 기본적으로 Regression 작업은 **R2**를 최적화하고 Classification은 **Accuracy**를 최적화 합니다.

## 예제

#### Classification

```python
# Importing dataset 
from pycaret.datasets import get_data 
diabetes = get_data('diabetes') 

# Importing module and initializing setup 
from pycaret.classification import * 
clf1 = setup(data = diabetes, target = 'Class variable')

# train a decision tree model
dt = create_model('dt')

# tune hyperparameters of decision tree
tuned_dt = tune_model(dt)

# tune hyperparameters with increased n_iter
tuned_dt = tune_model(dt, n_iter = 50)

# tune hyperparameters to optimize AUC
tuned_dt = tune_model(dt, optimize = 'AUC') #default is 'Accuracy'

# tune hyperparameters with custom_grid
params = {"max_depth": np.random.randint(1, (len(data.columns)*.85),20),
          "max_features": np.random.randint(1, len(data.columns),20),
          "min_samples_leaf": [2,3,4,5,6],
          "criterion": ["gini", "entropy"]
          }
tuned_dt_custom = tune_model(dt, custom_grid = params)

# tune multiple models dynamically
top3 = compare_models(n_select = 3)
tuned_top3 = [tune_model(i) for i in top3]
```

#### Regression

```python
from pycaret.datasets import get_data 
boston = get_data('boston') 

# Importing module and initializing setup 
from pycaret.regression import * 
reg1 = setup(data = boston, target = 'medv')

# train a decision tree model
dt = create_model('dt')

# tune hyperparameters of decision tree
tuned_dt = tune_model(dt)

# tune hyperparameters with increased n_iter
tuned_dt = tune_model(dt, n_iter = 50)

# tune hyperparameters to optimize MAE
tuned_dt = tune_model(dt, optimize = 'MAE') #default is 'R2'

# tune hyperparameters with custom_grid
params = {"max_depth": np.random.randint(1, (len(data.columns)*.85),20),
          "max_features": np.random.randint(1, len(data.columns),20),
          "min_samples_leaf": [2,3,4,5,6],
          "criterion": ["gini", "entropy"]
          }
tuned_dt_custom = tune_model(dt, custom_grid = params)

# tune multiple models dynamically
top3 = compare_models(n_select = 3)
tuned_top3 = [tune_model(i) for i in top3]
```

#### Clustering

```python
# Importing dataset
from pycaret.datasets import get_data
diabetes = get_data('diabetes')

# Importing module and initializing setup
from pycaret.clustering import *
clu1 = setup(data = diabetes)

# Tuning K-Modes Model
tuned_kmodes = tune_model('kmodes', supervised_target = 'Class variable')
```

#### Anomaly Detection Example

```python
# Importing dataset
from pycaret.datasets import get_data
boston = get_data('boston')

# Importing module and initializing setup
from pycaret.anomaly import *
ano1 = setup(data = boston)

# Tuning Isolation Forest Model
tuned_iforest = tune_model('iforest', supervised_target = 'medv')
```

#### Natural Language Processing

```python
# Importing dataset
from pycaret.datasets import get_data
kiva = get_data('kiva')

# Importing module and initializing setup
from pycaret.nlp import *
nlp1 = setup(data = kiva, target = 'en')

# Tuning LDA Model
tuned_lda = tune_model('lda', supervised_target = 'status')
```
