이 글은 PyCaret홈페이지의 내용을 다루고 있습니다.

원문 : [Create Models](https://pycaret.org/create-model/)

# 모델 생성하기

**create\_model**을 사용하면 간단하게 모델 생성을 할 수 있습니다. 이 함수는 string형의 Model ID를 파라미터로 갖습니다. 지도학습모듈(classification, regression)의 경우, 학습된 모델 객체와 함께 k-폴드 교차검증된 성능지표 테이블을 함께 return 합니다. 비지도학습모듈 중 clustering 의 경우, 학습된 모델 객체와 함께 성능지표를 return 합니다. 나머지 비지도학습모듈인 anomaly detection, natural language processing, association rule mining은 학습된 모델 객체만 return합니다. 사용되는 평가 지표는 아래와 같습니다.

-   Classification : Accuracy, AUC, Recall, Precision, F1, Kappa, MCC
-   Regression : MAE, MSE, RMSE, R2, RMSLE, MAPE

fold의 수는 **create\_model**함수의 **fold** 파라미터를 사용해 변경할 수 있습니다. 기본값으로 **fold**는 10입니다. 모든 지표(metric)은 기본적으로 4자리 소수로 반올림되며, **create\_model**의 **round** 파라미터를 사용해 변경할 수 있습니다. 비록 모델을 앙상블 하는 기능은 따로 떨어져있지만, **create\_model**함수 내 **methode** 파라미터와 함께 **ensemble** 파라미터를 사용하여 모델을 앙상블할 수 있는 빠른 방법이 있습니다.

## 모델 예제

#### Classification

| ID | Name |
| :-- | :-- |
| ‘lr’ | Logistic Regression |
| ‘knn’ | K Nearest Neighbour |
| ‘nb’ | Naives Bayes |
| ‘dt’ | Decision Tree Classifier |
| ‘svm’ | SVM – Linear Kernel |
| ‘rbfsvm’ | SVM – Radial Kernel |
| ‘gpc’ | Gaussian Process Classifier |
| ‘mlp’ | Multi Level Perceptron |
| ‘ridge’ | Ridge Classifier |
| ‘rf’ | Random Forest Classifier |
| ‘qda’ | Quadratic Discriminant Analysis |
| ‘ada’ | Ada Boost Classifier |
| ‘gbc’ | Gradient Boosting Classifier |
| ‘lda’ | Linear Discriminant Analysis |
| ‘et’ | Extra Trees Classifier |
| ‘xgboost’ | Extreme Gradient Boosting |
| ‘lightgbm’ | Light Gradient Boosting |
| ‘catboost’ | CatBoost Classifier |

#### Regression

| ID | Name |
| :-- | :-- |
| ‘lr’ | Linear Regression |
| ‘lasso’ | Lasso Regression |
| ‘ridge’ | Ridge Regression |
| ‘en’ | Elastic Net |
| ‘lar’ | Least Angle Regression |
| ‘llar’ | Lasso Least Angle Regression |
| ‘omp’ | Orthogonal Matching Pursuit |
| ‘br’ | Bayesian Ridge |
| ‘ard’ | Automatic Relevance Determination |
| ‘par’ | Passive Aggressive Regressor |
| ‘ransac’ | Random Sample Consensus |
| ‘tr’ | TheilSen Regressor |
| ‘huber’ | Huber Regressor |
| ‘kr’ | Kernel Ridge |
| ‘svm’ | Support Vector Machine |
| ‘knn’ | K Neighbors Regressor |
| ‘dt’ | Decision Tree |
| ‘rf’ | Random Forest |
| ‘et’ | Extra Trees Regressor |
| ‘ada’ | AdaBoost Regressor |
| ‘gbr’ | Gradient Boosting Regressor |
| ‘mlp’ | Multi Level Perceptron |
| ‘xgboost’ | Extreme Gradient Boosting |
| ‘lightgbm’ | Light Gradient Boosting |
| ‘catboost’ | CatBoost Regressor |

#### Clustering

| ID | Name |
| :-- | :-- |
| ‘kmeans’ | K-Means Clustering |
| ‘ap’ | Affinity Propagation |
| ‘meanshift’ | Mean shift Clustering |
| ‘sc’ | Spectral Clustering |
| ‘hclust’ | Agglomerative Clustering |
| ‘dbscan’ | Density-Based Spatial Clustering |
| ‘optics’ | OPTICS Clustering |
| ‘birch’ | Birch Clustering |
| ‘kmodes’ | K-Modes Clustering |

#### Anomaly Detection

| ID | Name |
| :-- | :-- |
| ‘abod’ | Angle-base Outlier Detection |
| ‘iforest’ | Isolation Forest |
| ‘cluster’ | Clustering-Based Local Outlier |
| ‘cof’ | Connectivity-Based Outlier Factor |
| ‘histogram’ | Histogram-based Outlier Detection |
| ‘knn’ | k-Nearest Neighbors Detector |
| ‘lof’ | Local Outlier Factor |
| ‘svm’ | One-class SVM detector |
| ‘pca’ | Principal Component Analysis |
| ‘mcd’ | Minimum Covariance Determinant |
| ‘sod’ | Subspace Outlier Detection |
| ‘sos | Stochastic Outlier Selection |

#### Natural Language Processing

| ID | Model |
| :-- | :-- |
| ‘lda’ | Latent Dirichlet Allocation |
| ‘lsi’ | Latent Semantic Indexing |
| ‘hdp’ | Hierarchical Dirichlet Process |
| ‘rp’ | Random Projections |
| ‘nmf’ | Non-Negative Matrix Factorization |

## 코드 예제

#### Classification

```python
# Importing dataset 
from pycaret.datasets import get_data 
diabetes = get_data('diabetes') 

# Importing module and initializing setup 
from pycaret.classification import * 
clf1 = setup(data = diabetes, target = 'Class variable')

# train logistic regression model
lr = create_model('lr') #lr is the id of the model

# check the model library to see all models
models()

# train rf model using 5 fold CV
rf = create_model('rf', fold = 5)

# train svm model without CV
svm = create_model('svm', cross_validation = False)

# train xgboost model with max_depth = 10
xgboost = create_model('xgboost', max_depth = 10)

# train xgboost model on gpu
xgboost_gpu = create_model('xgboost', tree_method = 'gpu_hist', gpu_id = 0) #0 is gpu-id

# train multiple lightgbm models with n learning_rate<br>import numpy as np
lgbms = [create_model('lightgbm', learning_rate = i) for i in np.arange(0.1,1,0.1)]

# train custom model
from gplearn.genetic import SymbolicClassifier
symclf = SymbolicClassifier(generation = 50)
sc = create_model(symclf)
```

#### Regression

```python
# Importing dataset 
from pycaret.datasets import get_data 
boston = get_data('boston') 

# Importing module and initializing setup 
from pycaret.regression import * 
reg1 = setup(data = boston, target = 'medv') 

# train linear regression model
lr = create_model('lr') #lr is the id of the model

# check the model library to see all models
models()

# train rf model using 5 fold CV
rf = create_model('rf', fold = 5)

# train svm model without CV
svm = create_model('svm', cross_validation = False)

# train xgboost model with max_depth = 10
xgboost = create_model('xgboost', max_depth = 10)

# train xgboost model on gpu
xgboost_gpu = create_model('xgboost', tree_method = 'gpu_hist', gpu_id = 0) #0 is gpu-id

# train multiple lightgbm models with n learning_rate
import numpy as np
lgbms = [create_model('lightgbm', learning_rate = i) for i in np.arange(0.1,1,0.1)]

# train custom model
from gplearn.genetic import SymbolicRegressor
symreg = SymbolicRegressor(generation = 50)
sc = create_model(symreg)
```

### Clustering

```python
# Importing dataset
from pycaret.datasets import get_data
jewellery = get_data('jewellery')

# Importing module and initializing setup
from pycaret.clustering import *
clu1 = setup(data = jewellery)

# check the model library to see all models
models()

# training kmeans model
kmeans = create_model('kmeans')

# training kmodes model
kmodes = create_model('kmodes')
```

#### Anomaly Detection

```python
# Importing dataset
from pycaret.datasets import get_data
anomalies = get_data('anomalies')

# Importing module and initializing setup
from pycaret.anomaly import *
ano1 = setup(data = anomalies)

# check the model library to see all models
models()

# training Isolation Forest
iforest = create_model('iforest')

# training KNN model
knn = create_model('knn')
```

#### Natural Language Processing

```python
# Importing dataset
from pycaret.datasets import get_data
kiva = get_data('kiva')

# Importing module and initializing setup
from pycaret.nlp import *
nlp1 = setup(data = kiva, target = 'en')

# check the model library to see all models
models()

# training LDA model
lda = create_model('lda')

# training NNMF model
nmf = create_model('nmf')
```

#### Association Rule

```python
# Importing dataset
from pycaret.datasets import get_data
france = get_data('france')

# Importing module and initializing setup
from pycaret.arules import *
arule1 = setup(data = france, transaction_id = 'InvoiceNo', item_id = 'Description')

# creating Association Rule model
mod1 = create_model(metric = 'confidence')
```
