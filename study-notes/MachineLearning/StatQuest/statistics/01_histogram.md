# 히스토그램이란?

동영상을 시청하시려면 아래 그림을 클릭하세요

[![Watch the video](https://img.youtube.com/vi/qBigTkBLU6g/hqdefault.jpg)](https://youtu.be/qBigTkBLU6g)

## 동영상 내용 정리

- 히스토그램이란 bins(구간)을 만들고, 해당 구간에 속하는 값을 쌓아올려 보여줍니다.

- 히스토그램을 통해 어떤 분포(정규분포, 지수분포 등)를 사용할지 결정할 수 있습니다.

- Default bin의 폭을 믿지 말고, 다양하게 테스트 해보면서 최적의 bin의 폭값을 찾는 것이 중요합니다!

## 궁금한 내용

### seaborn의 distplot에도 bin의 폭을 지정하는 부분이 있을까?

- bins 라는 argument가 존재합니다.
- Numpy Histogram : 값이 이산형일 경우 default 값은 10, 연속형일 경우 균일하지 않은 bin의 넓이를 사용합니다.
- string('auto', 'fd', 'doane', 'scott', 'stone', 'rice', 'sturges', 'sqrt')등도 사용 가능하며, 상황에 맞게 사용합니다.

##### [numpy.histogram\_bin\_edges](https://numpy.org/doc/stable/reference/generated/numpy.histogram_bin_edges.html#numpy.histogram_bin_edges)

```
bins : int형 또는 스칼라 시퀀스 또는 문자열 (선택사항임)

만약 bins 가 int 형일때, 주어진 범위의 동일한 폭의 bin이 정의된다. (기본값 : 10) 만약 bins가 시퀀스라면 동일하지 않은 bins의 폭을 허용하면 가장 오른쪽 엣지까지 포함하는 bin 의 폭을 정의한다.

만약 bins가 아래 나열된 string 이라면, hitogram_bin_edges는 선택된 방법을 사용하여 최적의 bins폭을 계산한다.  bins의 폭이 실제 데이터의 최적인 반면에, 전체 범위를 채우기위해 비어있는 부분을 포함하여 계산됩니다. 시각화를 위해서는 'auto' 옵션을 사용하는 것이 좋습니다. 가중치가 적용된 data는 자동 bin의 폭 선택이 지원되지 않습니다.

‘auto’
Maximum of the ‘sturges’ and ‘fd’ estimators. Provides good all around performance.

‘fd’ (Freedman Diaconis Estimator)
Robust (resilient to outliers) estimator that takes into account data variability and data size.

‘doane’
An improved version of Sturges’ estimator that works better with non-normal datasets.

‘scott’
Less robust estimator that that takes into account data variability and data size.

‘stone’
Estimator based on leave-one-out cross-validation estimate of the integrated squared error. Can be regarded as a generalization of Scott’s rule.

‘rice’
Estimator does not take variability into account, only data size. Commonly overestimates number of bins required.

‘sturges’
R’s default method, only accounts for data size. Only optimal for gaussian data and underestimates number of bins for large non-gaussian datasets.

‘sqrt’
Square root (of data size) estimator, used by Excel and other programs for its speed and simplicity.
```

### 관련 링크

[seaborn.distplot](https://seaborn.pydata.org/generated/seaborn.distplot.html)  
[matplotlib.pyplot.hist](https://matplotlib.org/3.1.1/api/_as_gen/matplotlib.pyplot.hist.html)  
[numpy.histogram](https://numpy.org/doc/stable/reference/generated/numpy.histogram.html#numpy.histogram)  
[bins string](https://numpy.org/doc/stable/reference/generated/numpy.histogram_bin_edges.html#numpy.histogram_bin_edges)
