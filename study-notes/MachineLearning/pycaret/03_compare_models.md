이 글은 PyCaret홈페이지의 내용을 다루고 있습니다.

원문 : [Compare Models](https://pycaret.org/compare-models/)

# 여러 모델 비교하기

지도학습을 할때 **여러 모델 비교하기**를 가장 처음으로 해볼 것을 추천합니다. 이 기능은 모델 라이브러리 내에 있는 모든 모델을 default 하이퍼파라미터와 함께 학습시킵니다. 또한 교차검증을 사용하여 performance 지표를 평가합니다.

-   Classification : Accuracy, AUC, Recall, Precision, F1, Kappa, MCC
-   Regression : MAE, MSE, RMSE, R2, RMSLE, MAPE

여러 모델 비교하기를 사용하면 모든 모델의 folds의 평균값을 보여주는 표를 출력해줍니다. fold의 갯수는 **compare\_models** 함수의 **fold** 파라미터를 통해 결정할 수 있습니다. 기본 값으로 fold는 10 값을 가집니다. **sort** 파라미터를 사용하여 어떤 지표(metric)으로 정렬(내림차순)해서 보여줄 것임을 선택할 수 있습니다. Classification의 경우 **Accuracy**로, Regression의 경우 **R2**로 기본 값을 가집니다. 특정 모델의 경우 run-time이 길기 떄문에 비교기능이 막혀있습니다. 이 막혀있는 기능을 해지하기 위해서는 **turbo** 파라미터를 False로 설정합니다.

이 함수는 오로지 **pycaret.classification**과 **pycaret.regression**모듈에서만 쓸 수 있습니다.

## Classification 예제

```python
# Importing dataset
from pycaret.datasets import get_data
diabetes = get_data('diabetes')

# Importing module and initializing setup
from pycaret.classification import *
clf1 = setup(data = diabetes, target = 'Class variable')

# return best model
best = compare_models()

# return top 3 models based on 'Accuracy'
top3 = compare_models(n_select = 3)

# return best model based on AUC
best = compare_models(sort = 'AUC') #default is 'Accuracy'

# compare specific models
best_specific = compare_models(whitelist = ['dt','rf','xgboost'])

# blacklist certain models
best_specific = compare_models(blacklist = ['catboost', 'svm'])
```

## Regression 예제

```python
# Importing dataset
from pycaret.datasets import get_data
boston = get_data('boston')

# Importing module and initializing setup
from pycaret.regression import *
reg1 = setup(data = boston, target = 'medv')

# return best model
best = compare_models()

# return top 3 models based on 'R2'
top3 = compare_models(n_select = 3)

# return best model based on MAPE
best = compare_models(sort = 'MAPE') #default is 'R2'

# compare specific models
best_specific = compare_models(whitelist = ['dt','rf','xgboost'])

# blacklist certain models
best_specific = compare_models(blacklist = ['catboost', 'svm'])
```
