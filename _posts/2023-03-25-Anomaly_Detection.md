---
layout: single
title: "[Paper Review] Deep Learning for Anomaly Detection in Time-Series Data: Review, Analysis and Guidelines"
use_math: true
comments: ture
toc: true
author_profile: false
---
본 논문은 2021 IEEE 저널에 투고된 논문으로 딥러닝을 이용한 시계열 이상감지에 대한 풍부한 리뷰를 하고 있습니다.  
이번 포스팅에서는 본 [논문](https://ieeexplore.ieee.org/abstract/document/9523565)의 간략한 요약을 해보겠습니다.

## 1. Anomaly in Time-Series Data
### 1.1 Properties of Time-Series Data
시계열 데이터는 크게 네 가지 특성이 있습니다.
* **Temporality** : time series data는 시간적 상관관계 또는 의존성이 존재합니다.
* **Dimensionality** : 특정 시점에 수집된 데이터는 하나 또는 여러 개의 attribute를 가집니다. (ex. Univariate, Multivariate) Univariate에서는 temporal dependency만 고려하면 되지만, Mutivariate에서는 관찰값들 사이의 상관관계도 고려해야 합니다.
* **Nonstationary** : 데이터의 통계적 특징이 시간의 흐름에 따라 달라집니다. 즉, $F_x(x^{1+\tau},...,x^{t+\tau}) \ne F_x(x^1,...,x^t)$ 입니다. 이러한 특징을 갖는 데이터는 다음과 같은 종류가 있습니다.
    * Seasonality : 주기적으로 반복되는 패턴이 발생하는 특성
    * Concept Drift : 시간이 지남에 따라 데이터의 통계 분포의 변화
    * Change Points : 특정 이벤트가 발생해 데이터의 성질이 변화 (ex. 공정에서의 다른 설정으로 재시작)
* **Noise** : 데이터의 수집, 저장, 전달, 처리등의 모든 과정에서 발생하는 어쩔 수 없는 데이터의 변화를 의미합니다. noise로 인해 잘못된 이상치를 검출할 수 있으므로 항상 noise를 의식해야 합니다.

### 1.2 Anomalies in Time_Series Data
이상치는 이전 시간과 크게 다른 예기치 않은 동작을 보여주는 데이터로 정의할 수 있습니다. 이번 챕터에서는 이 이상치를 좀 더 세분화해서 어떤 종류가 있는지 소개합니다.
* **Point Anomaly** : 갑작스럽게 정상의 상태를 벗어나는 data point 또는 sequence를 의미합니다. 이러한 현상은 일시적인 노이즈로 보일 수도 있습니다.
* **Contextual Anomaly** : 정상 범위를 넘지 않으나 비정상 피턴이 관찰되는 data point 또는 sequence를 의미합니다. 이는 정상 범위를 넘지 않는 만큼 매우 탐지하기 어렵습니다.
* **Collective Anomaly** : 시간이 지남에 따라 정상 데이터와 점진적으로 다른 패턴을 보이는 data point 또는 sequence를 의미합니다. 한 번에 검출하기 어려워 장기적인 관찰이 특히 필요한 이상치 형태입니다.
  
![fig1]({{site.url}}/images/2023-03-25-paper2/fig1.png){: width="700" height="700"}

다음은 좀 더 다양한 이상치들의 종류들입니다.

![fig2]({{site.url}}/images/2023-03-25-paper2/fig2.png){: width="700" height="700"}

## 2. Classical Approaches
위의 챕터에서는 시계열 데이터의 간략한 특징과 이상치의 종류들에 대해 알아보았습니다. 이번 챕터에서는 시계열 데이터의 고전적인 이상치 탐지 방법들에 대해 알아봅니다.
* **Statistical Model** : 데이터의 통계량을 계산하여 모델링하고 이 모델을 통해 새로운 데이터가 그럴 듯 한지 평가하는 방법입니다.
* **Distance-Based Model** : 두 개의 temporal sequence에 대한 xplicit-distance(ex. Euclidean, DTW 등)을 계산하여 둘 사이의 유사도를 결정하고 결정한 유사도로 만든 유사도 행렬을 통해 정상 sequence와 새로운 sequence 사이의 거리를 통해 이상치를 판별하는 방법입니다.
* **Predictive Model** : 과거부터 현재까지의 데이터를 사용하여 미래의 상태를 예측하는 방법으로 예측된 값과 실제로 얻은 데이터 사이의 불일치 정도를 활용하여 이상치 탐지를 수행하는 방법입니다.
* **Clustering Model** : Unsupervied Setting에서 데이터를 군집화하고, 그 군집의 중심으로부터의 거리를 기준으로 새로운 데이터에 대한 이상치를 판별하는 방법이다.

하지만 위의 고전적인 방법들은 장애 데이터의 희소성으로 인해 label을 충분히 수집하는 데 시간과 리소스가 많이 소요된다고 합니다. 또한 label을 얻더라도 class imbalance 문제가 있을 수 있습니다.  
또, 산업현장의 자동화와 시스템의 복잡성이 증가함에 따라 Multivariate Time-Series Data에 대한 모니터링의 필요성이 대두되고 있습니다. 아는 다수의 Variable 사이의 Correlation을 분석하는 것이 중요해졌다는 것을 뜻하고, 차원의 수가 많기 때문에 전통적인 접근 방법은 일반적으로 차원의 저주로 인해 성능이 감소하기 쉽습니다. 즉, Anomaly Detection을 의한 Deep Learning 방식의 관심이 커지고 있습니다.  

## 3. Deep Learning for Anomaly Detection
### 3.1 가정
딥러닝을 통한 이상치 탐지는 준지도 또 비지도학습을 가정으로 합니다. 이는 부족한 labeled data 때문입니다. 준지도학습을 가정할 때는 학습 시 모든 data가 normal class라고 가정하고, 비지도학습일 경우 normal과 abnormal class가 구분되지 않습니다. 또 딥러닝을 통한 이상감지는 Multivariate data를 기준으로 합니다. Univariate data에 대해 딥러닝을 사용하면 부하도 많이 들고 추론 시간도 오래걸리기 때문이라고 생각합니다.

### 3.2 Inter-Correlation between Variables
Multivariate time series data를 적절히 다루려면 매 순간 변수 사이의 correlation을 고려해야 합니다. 다음은 correlation을 고려하는 방법입니다.
* **Dimensional Reduction** : 중요한 변수만을 남기고 불필요한 변수를 제거하는 방법입니다. 하지만 이 방법은 일부 연관성이 높은 변수를 제거하기 때문에 anomaly의 원인을 파악하기 어렵다는 단점이 있습니다.
* **2D Matrix** : n개의 변수가 있을 때, 직접적으로 각각의 변수 사이의 유사성과 상대적인 scale을 파악하는 행렬입니다.
* **Graph** : Attention mechanism을 GNN에 적용하여 각 변수들의 성질과 변수 간의 관계를 표현할 수 있는 Representation을 얻습니다. 서로 영향을 주고받는 변수들을 이웃하게 노드를 만들고 각 노드가 영향을 주는 정도를 score로 계산하여 edge를 형성합니다.

다음은 각 방법들과 그와 관련된 모델들을 정리한 표입니다.

![fig3]({{site.url}}/images/2023-03-25-paper2/fig3.png){: width="700" height="700"}

### 3.3 Modeling Temporal Context
Time series data에 대해 anomaly detection을 수행하기 위해서는 temporal context(시간적 정보)을 모델링해야 합니다. 이 시간적 정보를 모델링하는 적절한 방법으로는 크게 RNN, CNN, RNN+CNN, Attention등이 있다고 하는데 최근에는 Transformer 계열이나 Linear모델들이 주목받고 있습니다. 본 논문에서는 크게 언급을 하고있지는 않습니다. 다음은 모델들을 정리한 표입니다.

![fig4]({{site.url}}/images/2023-03-25-paper2/fig4.png){: width="700" height="700"}

### 3.4 Anomaly Criteria
본 챕터에서는 이상치에 대한 평가를 하기 위한 전략들을 소개하고 있습니다.
* **Reconstruction Error** : Reconstruction Model(AE, GAN 등)에서 정상 데이터를 재구축하는 방법을 학솝하고 실제 데이터에 대해서 알마나 잘 재구축이 가능한지를 평가하는 방법
* **Prediction Error** : Prediction model에서 예측 값과 실제 값 가이의 차이를 평가하는 방법
* **Dissimilarity** : 축적된 data를 통해 구축한 군집 혹인 분포에 대해 새로운 데이터가 얼마나 멀리 떨어져 있는지 평가하는 방법

![fig5]({{site.url}}/images/2023-03-25-paper2/fig5.png){: width="700" height="700"}

다음은 2021년 기준 최근의 연구 방향들을 표현한 표입니다.

![fig6]({{site.url}}/images/2023-03-25-paper2/fig6.png){: width="700" height="700"}

## 4. Guidelines for Practitioners
이번 챕터에서는 실제 이상치탐지를 할 때 어떠한 전략을 세워야 하는지에 대한 가이드라인을 소개하고 있습니다.

### 4.1 Real-Time vs. Early warning
* **Real-Time** : 실제 불량이 발생 시에만 알람을 발생 시키는 전략입니다. 이는 빠른 반응이 중요한 산업들(Online business, Finance, Monitoring Manufacturing equipment 등)에서 선택할 수 있는 전략입니다. 보통 Reconstruction error를 사용하는 모델이 사용되는데 실시간으로 판단되어야 하므로 연산에 대한 부담이 적어야 합니다.
* **Early warning** : 잠재적인 위험을 미리 제거하는 전략입니다. 이는 Anomaly 발생시 피해가 매우 큰 산업들(철강, 석유 산업 등 연속된 공정)에서 선택할 수 있는 전략입니다. 보통 Autoregressive model이 사용되는데 지나친 False alarm이 발생하지 않도록 적절한 임계치 설정이 필요합니다.

### 4.2 Batch Learning vs. Online Update
시계열 데이터의 일반적인 과제는 위에서 논의한 바와 같이 데이터의 비정상적인 특성입니다. 데이터 분포의 변화에 따라 논문은 모델을 업데이트하기 위한 두 가지 유형의 접근법을 제안합니다.
* **Batch Learning** : 일반적인 학습 방식으로 test data와 동일한 분포를 가지는 데이터를 sampling하여 batch단위로 학습하는 방법입니다. 이 방법은 시스템 관리자가 데이터를 업데이트할 때마다 데이터를 회수할 수 없을 때 문제가 될 수 있다고 합니다.
* **Online Learning** : 이전 단계의 데이터 없이 새로 추가된 데이터에 대해 지속적으로 model의 학습을 이어나가는 방식입니다. 하지만 online learning은 대부분의 딥러닝 모델에서 찾아볼 수 없는데 continual learning이 그 대안이 될 수 있습니다. 실제로 저도 continual learning의 고려를 많이 고민했었습니다. 하지만 분포가 바뀔때를 포착한다음 재학습을 하는 방향으로 현재는 진행하고 있습니다. continual learning에 대한 포스팅도 시간이 되면 하도록 하겠습니다.

## 정리
본 논문은 딥러닝을 이용한 시계열 이상감지에 대해 정리를 하고 있습니다. 사실 현업에서 streaming data의 이상감지를 할 때는 딥러닝을 사용하는 데 큰 제약조건이 있습니다. 학습속도는 그렇다쳐도 추론속도가 생각보다 오래걸리기 때문입니다. 추론이 새로 수집되는 속도보다 느리게 된다면 의미가 없습니다. 때문에 streaming data에 대한 설명이 적은 점이 아쉬운 논문이였습니다.