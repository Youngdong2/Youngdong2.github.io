---
layout: single
title: "[Paper Review] FluxEV: A Fast and Effective Unsupervised Framework Time-series Anomaly Detection(WSDM, 2021)"
categories: etc
use_math: true
---

# [Paper Review] FluxEV: A Fast and Effective Unsupervised Framework Time-series Anomaly Detection(WSDM, 2021)

본 논문은 실시간 이상탐지를 보다 효과적이고 빠르게 수행하는 방법론을 소개하고 있습니다. 본 포스팅은 본 논문의 요약 및 해석에 중점을 두고 있기 때문에 번역체가 빈번하게 나올 수 있으므로 양해바랍니다. 목차는 다음과 같습니다.
## 목차
1. Introduction
2. Related Work
3. Preliminaries
4. Methodology

## Introduction
* Online platform에서 이상 탐지 서비스를 제공하는 것은 대용량 데이터와 적시 대응이라는 두 가지 어려운 요소로 인해 쉽지 않은 작업이다. 이는 모니터링 솔루션을 개발하는 우리 회사에서도 현재 이상감지 모델을 개발하는 데 있어 어려움을 겪는 문제 중 하나이다.  
    *  수십만 또는 수백만 개의 비즈니스 관련 시계열이 매분 생성된다. 이렇게 많은 양의 데이터에서 feature를 추출하는 것은 매우 어려우며 이러한 양의 데이터는 레이블이 지정되어 있지 않으며 지정하기도 어렵다.
    * 온라인 시계열 이상 감지는 매초 지연이 심각한 결과를 초래할 수 있으므로 신속하게 대응하는 팀지 알고리즘이 필요하다. 따라서 시간 복잡도가 높은 모델은 뛰어난 정확도를 제공하지만 현업에 적용하기는 쉽지않다.
* 지난 몇년간, 다양한 연구가 제안되었지만 label의 부족, 성능 저하, 파라미터 튜닝에 의존, 시간 부하, 재학습의 필요, cold-start issue 중 하나의 문제는 항상 있었다.
* 선행 연구 중 SPOT이라는 모델은 extreme value를 탐지하는 데 라벨링 없이 좋은 성능을 보여줬다. 이는 대규모 시계열 데이터에서 이상 탐지를 해결할 잠재력이 있다고 판한하지만 extreme value만 탐지한다는 점에서 한계도 존재한다. 왜냐하면 실제 사례에서의 이상탐지는 전체 데이터 분포의 extreme value를 발견하는 것 뿐만 아니라 다음의 그림처럼 정상적인 패턴이 아닌 변동도 발견해야 한다.

![fig1]({{site.url}}/images/2023-02-18-anomaly_paper/fig1.png)

* 본 논문에서는 실시간으로 보다 효율적이고 효과적으로 이상탐지를 하는 것을 목적으로 한다. 이를 위한 핵심 아이디어는 다음과 같다.
    * **이상의 정도를 나타내기 위해 적절한 특징을 추출하고 이상치의 특징을 최대한 극단적으로 만들 수 있다면 이상치 탐지 문제는 잘 풀릴 수 있다.**

## Related Work
* 기본적으로 시계열 이상 탐지 알고리즘은 지도, 비지도 및 통계적 접근 등의 세가지 범주로 나뉜다. 이 중 지도학습은 일반적으로 레이블이 필요한데 실제로는 이상 레이블을 구하기 굉장히 힘들다. 나도 이때문에 학습은 차치하더라도 모델의 성능을 평가하는 데 있어 굉장히 어려웠다.
* 최근에는 딥러닝 기반 생성모델이 비지도 방법으로 주목받고 있지만 복잡한 모델일수록 시간이 많이 소모되고 파라미터 튜닝에 의존하는 등의 cost가 발생한다. 실제로 이 때문에 통계모형에 대한 논문을 찾아보았고 이 논문을 리뷰하는 것이다.
* 또한, 계절성이나 제품 업그레이드, 정책 변경 등으로 인해 시간이 지남에 따라 데이터 패턴이 달라질 수 있으며, 이로 인해 기존 모델을 더 이상 적용할 수 없는 경우도 생긴다. 따라서 정기적으로 재학습을 해야하는데 이는 시간이 많이 걸리고 경제적이지 않다. 본 논문의 이러한 고민은 지난 1년간 현업에서도 했던 같은 고민이라 더 크게 와닿았다.
* 효율성 문제와 컴퓨팅 리소스를 고려해 일부 전통적 통계모형이 여전히 실제로 응용되고 있다. 통계모형은 구현이 간단하고 레이블이 필요하지 않지만 도메인 지식이 필요할 수 있다.

## Perliminaries
### Problem Statement
먼저 논문에서 사용하는 표기에 대해 정의를 하고 들어간다.
* time series data $X=[X_1, ..., X_n], X_i \in \mathbb{R}$
* window slice from time $i$ to $j$: $X_{i,j}=[X_i,X_{i+1},...,X_{j-1},X_j]$
## Methodology
![fig2]({{site.url}}/images/2023-02-18-anomaly_paper/fig2.png)
### Fluctuation Extraction
* 선행 연구에서는 단순 통계량을 뽑아 분석했다면 본 논문은 데이터의 변동성에 집중하여 이상치를 확인하는데 좀 더 유용하게 하고자 한다.
* LSTM, GRU같은 딥러닝 모델들을 대신해 예측모델로 EWMA를 사용한다.
error는 현재값과 예측값을 통해 계산한다.
$$E=[E_1,..., E_n]$$
$$E_i=X_i-EWMA(X_{i-s,i-1})$$
* 이 예측값은 기대값으로도 볼 수 있기 때문에 $E_i$는 $i$시점의 local 변동성이라고 할 수 있다.
### Two-step Smoothing Processing
이 부분이 본 논문의 핵심이라고 할 수 있다.
* 본 논문에서는 두단계의 feature processing 방법을 소개한다. (sequential, periodic)

![fig3]({{site.url}}/images/2023-02-18-anomaly_paper/fig3.png)

* 