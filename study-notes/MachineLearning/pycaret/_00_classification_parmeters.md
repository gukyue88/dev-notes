# pycaret classification setup 파라미터 정리

영어가 부족한 부분이나, ML의 이해가 부족한 부분은 ?? 처리되었거나 간단히 서술하였습니다.

```python
# data (데이터)
data = df

# target (타겟 컬럼명)
target = '타겟컬럼'

# train_size (train/test 분리 비율) : 0.7 (default)
train_size = 0.7

# sampling (train 사이즈별 예상 지표 출력 여부 결정)
# silent = True 일 경우 작동 안함
sampling = True

# sample_estimator (samling 지표 예상시 사용되는 모듈)
# None 일 경우 'Logistic Regression' 가 사용
sample_estimator = None

# categorical_imputation (범주형에서 값이 없을 때 대체값) : 'constant' or 'mode'
# 'constant' (default) : 'not_available' 값으로 대체
# 'mode' : 가장 빈번한 값으로 대체
categorical_imputation = 'constant'

# categorical_features (범주형피처 리스트 )
# 무조건 원-핫 인코딩 됨
categorical_features = []

# ordinal_features (서수형피처)
# 'low', 'medium', 'high' 처럼 값의 크기의 순서가 정해진 것들
# dict에 key에 column명, value의 작은것에서 큰 순서로 나열
# {col명:['low', 'medium', 'high']}
ordinal_features = {
    'age_group':['10s', '20s', '30s', '40s', '50s', '60s', '+70s'],
    'education':['1', '2', '0', '3', '4']}

# high_cardinality_features (범주형피처 중 범주의 종류가 많은 피처)
# 한 범주형피처내 범주의 종류가 많을 때, one-hot encoding은 feature를 늘려 학습을 느리게하고, 몇몇 ML에게는 노이즈가 될 수 있음
# 특정 method(high_cardinality_method)를 통해 범주의 개수를 줄여줌
high_cardinality_features = []

# high_cardinality_method (high_cardinality_features의 처리방식 결정) : 'frequency' or 'clustering'
# 'frequency' (default) : 범주의 갯수로 인코딩
# 'clustering' : k-means같은 알고리즘을로 클러스터링하여 인코딩
high_cardinality_method = 'frequency'

# numeric_features (숫자형피처 리스트)
numeric_features = []

# numeric_imputation (숫자형피처 결측값 처리 방법) : 'mean' or 'median' or 'zero'
# 'mean' (default) : 평균 값으로 대체
# 'median' : 중앙값으로 대체
# 'zero' : 0으로 대체
numeric_imputation = 'mean'

# date_features (날짜형피처)
# setup시 자동인식 안됨
# 이 피처는 모델링에 직접 사용되지 않고, feature 추출시 사용됨
date_features = None

# ignore_features (사용하지 않을 피처)
ignore_features = []

# normalize (스케일링 여부)
# 피처 공간을 normalize_method에 따라 정규화
# 선형알고리즘에서 성능이 좋다고 알려져 있으나, 효과가 있는지 테스트가 필요함
normalize = False

# normalize_method (스케일링 방식) : 'zscore' or 'minmax' or 'maxabs' or 'robust'
# 'zscore' (default) : z = (x -u) / s
# 'minmax' : 각 피처별 0~1 사이 값으로 변경
# 'maxabs' : 각 피처별 절대값의 최대 값이 1이 되도록 변경, 데이터이동 및 중심화가 되지 않아 희소성을 파괴하지 않음
# 'robust' : 각 피처별 사분위수 범위에 따라 변환, 데이터에 이상치가 포함된 경우 좋은 효과를 보임
normalize_method = 'zscore'

# transformation (정규분포화 여부 설정)
# normalize가 분산의 크기의 영향을 줄이기 위한 것이라면, transformation은 근본적인/급진적인 방법이다.(?)
# 데이터의 분포가 정규분포의 모양을 띄도록 변경해줌
# 데이터가 정규분포를 따른다는 것을 가정하는 ML (LogisticRegression, LDA, Gaussian Naive Bayes)에서 효과가 좋음
# 정규분포화시 최적파라미터는 skewness와 분산을 최소화하는 방식으로 정해짐
transformation = False

# transformation_method (정규분포화 방식) : 'yeo-johnson' or 'quantile'
# 'yeo-johnson' (default) : ??
# 'quantile' : 분위수 변환, 비선형이며 동일한 척도에서 측정된 변수 간의 선형상관관계를 왜곡시킬 수 있음
transformation_method = 'yeo-johnson'

# handle_unknown_categorical (새로운 category의 level을 만났을 때 처리 여부 설정)
# unknown_categorical_method에 따라 처리 방식 변경
handle_unknown_categorical = True

# unknown_categorical_method (handle_unknown_categorical 처리 방식) : 'least_frequent' or 'most_frequent'
# 'least_frequent' (default) : 가장 적은 갯수의 범주로 대체
# 'most_frequent' : 가장 많은 갯수의 범주로 대체
unknown_categorical_method = 'least_frequent'

# pca (Pricipal Component Analysis 적용 여부)
# pca_method 방식으로 데이터 차원 축소 여부 결정
pca = False

# pca_method (pca 방식) : 'linear' or 'kernel' or 'incremental'
# 'linear' (default) : ??
# 'kernel' : ??
# 'incremental' : ??
pca_method = 'linear'

# pca_components (pca 계수(?)) : int/float
# None (default) : 0.99 적용
pca_components = None

# ignore_low_variance (중요하지 않은 분산을 가진 피처 제거 여부 결정)
ignore_low_variance = False

# combine_rare_levels (희소피처(?) 결합 여부 결정)
# 범주형피처 중 범주가 많은 경우 피처들을 결합하여 좀 더 의미 있는 피처를 만들어 냄 (희소행렬읠 피함)
combine_rare_levels = False

# rare_level_threshold (combine_rare_levels의 임계값)
# 이 값 아래의 분산을 가진 범주들을 결합함
rare_level_threshold = 0.10

# bin_numeric_features (숫자형피처를 범주형피처로 변경시킬지 여부 결정)
# 숫자형피처에 연속값이 많거나, 예상범위를 벗어난 극단값이 적을 때 효과적임
# Kmeans cluster를 사용하며, 클러스터 갯수는 'sturges' 방식에 의해 결정
bin_numeric_features = None 

# remove_outliers (이상치제거 여부 결정)
# pca 선형 차원 축소 방법으로 이상치 제거
remove_outliers = False

# outliers_threshold (이상치 제거시 임계값)
# 0.05 (default) : 학습데이터에서 분포 양 끝 0.025 제거
outliers_threshold = 0.05

# remove_multicollinearity (다중공선성 문제 피처 제거 여부 결정)
# 상관관계가 높은 피처중 target과 덜 관련있는 피처 제거
remove_multicollinearity = False

# multicollinearity_threshold (다중공성선 문제 피처 제거 임계값)
# 0.9 (default) : 상관관계 값이 0.9 이상이면 제거
multicollinearity_threshold = 0.9

# remove_perfect_collinearity (완전동일 피처 제거)
# 상관관계 1.0 인 피처 하나만 남김, 제거되는 피처는 랜덤
remove_perfect_collinearity = False

# create_clusters (클러스터 피처 생성 여부 결정)
# 각 인스턴스에 할당된 클러스터에서 추가 피처가 생성됨(?)
# 각 피처마다 클러스터링 된 피처를 추가하는 것 같음...
create_clusters = False

# cluster_iter (클러스터 피처 생성시 클러스터 크기)
cluster_iter = 20

# polynomial_features (다항 피처 생성 여부 결정)
# 숫자형 피처를 다항식 방식으로 결합하여 새로운 피처 생성
polynomial_features = False

# polynomial_degree (다항 피처 생성시 계수)
# 2 (default) : 1, a, b, a^2, ab, b^2 피처 생성
polynomial_degree = 2

# trigonometry_features (삼각함수 피처(?) 생성 여부 결정)
# sin, tan, cos 적용하여 새로운 피처 생성
trigonometry_features = False

# polynomial_threshold (다항 피처 생성시 필요없는 피처 제거시 사용하는 임계값)
# Random Forest, AdaBoost, Linear correlation 조합으로 피처 중요도가 나오고 임계값 밑의 피처는 다시 제거
polynomial_threshold = 0.1

# group_features (통계 피처 생성할 그룹 리스트)
# group으로 주어진 feature들을 가지고 평균, 중앙값, 분산, 표준편자 같은 통계 feature 생성
# None (default)
# list : 하나의 그룹 열리스트
# list의 list : 그룹이 여러개일때
group_features = None

# group_names (group_features의 그룹 이름 지정)
# None (default) : group_1, group_2 형식으로 그룹 이름 지정
# list : 사용자가 그룹 이름 지정
group_names = None

# feature_selection (피처 셀렉션 활성화 여부 결정)
# RF, AdaBoost, 선형상관관계를 통한 중요도로 피처 셀렉션
feature_selection = False

# feature_selection_threshold (피처 셀렉션시 임계값)
# polynomial_feature와 feature_interation이 사용되는 경우 여러 값들로 시도해보는 것이 좋음
feature_selection_threshold = 0.8

# feature_selection_method (피처 셀렉션 방식 지정) : 'classic' or 'boruta'
# 'classic' (default) : pycaret 내 기본 피처중요도 계산
# 'boruta' : boost tree model을 통한 피처중요도 계산
feature_selection_method = 'classic'

# feature_interaction (숫자형피처 * 피처 생성 여부 결정)
# 모든 숫자형 피처에 두 개씩 *하여 피처 생성
feature_interaction = False       

# feature_ratio (숫자형피처 / 피처 생성 여부 결정)
# 모든 숫자형 피처에 두 개씩 /하여 피처 생성
feature_ratio = False

# interaction_threshold (feature_interaction시 중요도 떨어지는 피처 제거시 사용되는 임계값)
interaction_threshold = 0.01

# fix_imbalance (데이터 불균형 알고리즘 사용 여부 결정)
fix_imbalance = False

# fix_imbalance_method (데이터 불균형시 샘플 생성 방식 결정)
# 'None' (default) : SMOTE 적용
# fit_resample을 지원하는 모듈이라면 인자로 가능!
fix_imbalance_method = None

# data_split_shuffle (데이터 분리(split)시 행을 섞을지 여부 결정)
# True (default)
data_split_shuffle = True

# folds_shuffle (교차검증시 행을 섞을지 여부 결정)
# False (default)
folds_shuffle = False

# n_jobs (병렬처리시 job의 수)
# -1 (default) : 모든 프로세서 사용
n_jobs = -1

# use_gpu (gpu 사용여부 결정)
# False (default)
use_gpu = True #added in pycaret==2.1

# html (실행시 출력 여부 결정)
# True (default)
html = True

# session_id (모든 함수에서 사용될 radom seed)
# None(default) : 내부적으로 랜덤 숫자 생성
session_id = 2020

# log_experiment (모든 지표와 파라미터가 MLFlow server에 기록될지 여부 결정)
log_experiment = False

# experiment_name (로깅시 사용되는 테스트 이름)
# None(default) : 'clf'로 지정됨
experiment_name = None

# log_plots (MLflow 중 특정 plot이 png 파일로 저장될지 여부 결정)
log_plots = False

# log_profile (MLFlow 중 data profile이 html 파일로 저장될지 여부 결정)
log_profile = False

# log_data (train/test data 기록 여부 결정)
log_data = False

# silent (data type 결정시 확인하는 과정 스킵 여부 결정)
silent=True

# verbose (정보표 출력 여부 결정)
verbose=True

# profile (EDA를 위한 interactive HTML 보고서 실행 여부 결정)
profile = False

```
