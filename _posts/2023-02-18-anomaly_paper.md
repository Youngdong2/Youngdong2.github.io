---
layout: single
title: "[Paper Review] FluxEV: A Fast and Effective Unsupervised Framework Time-series Anomaly Detection(WSDM, 2021)"
categories: Paper_Review
use_math: true
comments: ture
toc: true
author_profile: false
---

본 논문은 실시간 이상탐지를 보다 효과적이고 빠르게 수행하는 방법론을 소개하고 있습니다. 본 포스팅은 본 논문의 요약 및 해석에 중점을 두고 있기 때문에 번역체가 빈번하게 나올 수 있으므로 양해바랍니다. 


## 1. Introduction
* Online platform에서 이상 탐지 서비스를 제공하는 것은 대용량 데이터와 적시 대응이라는 두 가지 어려운 요소로 인해 쉽지 않은 작업이다. 이는 모니터링 솔루션을 개발하는 우리 회사에서도 현재 이상감지 모델을 개발하는 데 있어 어려움을 겪는 문제 중 하나이다.  
    *  수십만 또는 수백만 개의 비즈니스 관련 시계열이 매분 생성된다. 이렇게 많은 양의 데이터에서 feature를 추출하는 것은 매우 어려우며 이러한 양의 데이터는 레이블이 지정되어 있지 않으며 지정하기도 어렵다.
    * 온라인 시계열 이상 감지는 매초 지연이 심각한 결과를 초래할 수 있으므로 신속하게 대응하는 팀지 알고리즘이 필요하다. 따라서 시간 복잡도가 높은 모델은 뛰어난 정확도를 제공하지만 현업에 적용하기는 쉽지않다.
* 지난 몇년간, 다양한 연구가 제안되었지만 label의 부족, 성능 저하, 파라미터 튜닝에 의존, 시간 부하, 재학습의 필요, cold-start issue 중 하나의 문제는 항상 있었다.
* 선행 연구 중 SPOT이라는 모델은 extreme value를 탐지하는 데 라벨링 없이 좋은 성능을 보여줬다. 이는 대규모 시계열 데이터에서 이상 탐지를 해결할 잠재력이 있다고 판한하지만 extreme value만 탐지한다는 점에서 한계도 존재한다. 왜냐하면 실제 사례에서의 이상탐지는 전체 데이터 분포의 extreme value를 발견하는 것 뿐만 아니라 다음의 그림처럼 정상적인 패턴이 아닌 변동도 발견해야 한다.

![fig1]({{site.url}}/images/2023-02-18-anomaly_paper/fig1.png){: width="500" height="500"}

* 본 논문에서는 실시간으로 보다 효율적이고 효과적으로 이상탐지를 하는 것을 목적으로 한다. 이를 위한 핵심 아이디어는 다음과 같다.
    * **이상의 정도를 나타내기 위해 적절한 특징을 추출하고 이상치의 특징을 최대한 극단적으로 만들 수 있다면 이상치 탐지 문제는 잘 풀릴 수 있다.**

## 2. Related Work
* 기본적으로 시계열 이상 탐지 알고리즘은 지도, 비지도 및 통계적 접근 등의 세가지 범주로 나뉜다. 이 중 지도학습은 일반적으로 레이블이 필요한데 실제로는 이상 레이블을 구하기 굉장히 힘들다. 나도 이때문에 학습은 차치하더라도 모델의 성능을 평가하는 데 있어 굉장히 어려웠다.
* 최근에는 딥러닝 기반 생성모델이 비지도 방법으로 주목받고 있지만 복잡한 모델일수록 시간이 많이 소모되고 파라미터 튜닝에 의존하는 등의 cost가 발생한다. 실제로 이 때문에 통계모형에 대한 논문을 찾아보았고 이 논문을 리뷰하는 것이다.
* 또한, 계절성이나 제품 업그레이드, 정책 변경 등으로 인해 시간이 지남에 따라 데이터 패턴이 달라질 수 있으며, 이로 인해 기존 모델을 더 이상 적용할 수 없는 경우도 생긴다. 따라서 정기적으로 재학습을 해야하는데 이는 시간이 많이 걸리고 경제적이지 않다. 본 논문의 이러한 고민은 지난 1년간 현업에서도 했던 같은 고민이라 더 크게 와닿았다.
* 효율성 문제와 컴퓨팅 리소스를 고려해 일부 전통적 통계모형이 여전히 실제로 응용되고 있다. 통계모형은 구현이 간단하고 레이블이 필요하지 않지만 도메인 지식이 필요할 수 있다.

## 3. Perliminaries
### 3.1 Problem Statement
먼저 논문에서 사용하는 표기에 대해 정의를 하고 들어간다.
* time series data $X=[X_1, ..., X_n], X_i \in \mathbb{R}$
* window slice from time $i$ to $j$: $X_{i,j}=[X_i,X_{i+1},...,X_{j-1},X_j]$

## 4. Methodology
![fig2]({{site.url}}/images/2023-02-18-anomaly_paper/fig2.png){: width="500" height="500"}
### 4.1 Fluctuation Extraction
* 선행 연구에서는 단순 통계량을 뽑아 분석했다면 본 논문은 데이터의 변동성에 집중하여 이상치를 확인하는데 좀 더 유용하게 하고자 한다.
* LSTM, GRU같은 딥러닝 모델들을 대신해 예측모델로 EWMA를 사용한다.
error는 현재값과 예측값을 통해 계산한다.

$\begin{align} E=[E_1,...,E_n] \end{align}$

* 이 예측값은 기대값으로도 볼 수 있기 때문에 $E_i$는 $i$시점의 local 변동성이라고 할 수 있다.

### 4.2 Two-step Smoothing Processing
이 부분이 본 논문의 핵심이라고 할 수 있다.
* 본 논문에서는 두단계의 feature processing 방법을 소개한다. (sequential, periodic)

![fig3]({{site.url}}/images/2023-02-18-anomaly_paper/fig3.png){: width="500" height="500"}

* 일반적으로 $E_i$는 대부분 정삼 범위에 있다. (논문에서는 local noises라고 지칭)
* 한 주기의 다른 시간대에서는 local 변동이 다르며 다른 주기의 같은 시간대에서는 일반적으로 유사한 변동을 가진다. (논문에서는 이를 periodic pattern이라고 본다)
* 때문에 smoothing처리를 하는 것은 주로 이런 local 노이즈와 주기적 패턴의 영향을 제거하고 정상 변동을 0에 가깝게 만들고 잠재적 이상치 변동만 유지하는 것이고 이는 본 논문의 핵심이다.

#### 4.2.1 First-step Smoothing
* first-step smoothing은 추출된 변동값 $E_i$를 순차적으로 처리해서 local 노이즈를 제거하는 데 사용된다. 다음과 같은 수식으로 이루어져 있다.
$\begin{align} \vartriangle \sigma= \sigma(E_{i-s, i})-\sigma(E_{i-s, i-1})
\end{align}$
$\begin{align} F_i=max(\vartriangle \sigma, 0)
\end{align}$
수식에 대해 잠깐 알아보자.

* $\vartriangle \sigma$는 현재 window인 $E_{i-s,i-1}$에 $E_i$가 추가되었을 때 std의 변화량이다. 즉, 현재 시점에 편차를 크게 하는 값이 들어왔다면 $\vartriangle \sigma$는 양수값이 될 것이다.
* $F_i$는 first-step smoothing 처리 후 time $i$에서의 변동값이다. 즉, local에서 변동성이 작은 값들은 0으로 처리해주는 과정이다.

코드로는 다음과 같이 짤 수 있을 것 같다.

```python
def first_smoothing(data: Series, s: int):
    F = [None] * len(data)
    ma = data.rolling(s).mean()
    remainder = data - ma
    f = []
    for i in range(s, len(remainder)):
        delta_sigma = np.std(remainder[i-s:i]) - np.std(remainder[i-s:i-1])
        f_i = max(delta_sigma, 0)
        f.append(f_i)
        F[s:] = f

    return F
```

#### 4.2.2 Second-step Smoothing
* second-step smoothing의 목적은 주기적 패턴의 영향을 제거하는 것이다.
* 주기적 데이터에 대한 작업을 수행하기 전에 고려해야 할 한가지 이슈는 data drift이다. 즉, 다른 기간의 동일한 시간대에서는 일반적으로 유사한 변동을 나타내지만 엄격하게 보면 일대일 대응은 아니다. 이를 해결하기 위해 data drift를 처리하는 데 간단한 방법을 제안한다.
* 현재 값: $X_t$, 주기의 길이: $l$, 현재 시간의 $p$번째 주기 전에 대해서, $max(X_{t-pl-d},..., X_{t-pl-1}, X_{t-pl},X_{t-pl+1},...,X_{t-pl+d})$를 사용하여 주기적 처리를 한다.
* second-step smoothing의 정의는 다음과 같다.
$\begin{align}  M_{i-d}=max(F_{i-2d,i}) \end{align}$
$\begin{align} \vartriangle F_i=F_i-max(M_{i-l(p-1)},...,M_{i-dl},M_{i-l}) \end{align}$
$\begin{align} S_i=max(\vartriangle F_i, 0) \end{align}$

위의 수식을 해석해보자.

* 먼저 $M$이라는 array에 $F$의 local maximum을 저장한다. 즉, first smoothing 이후의 변동성을 저장한다고 이해하면 될 것 같다. 여기서 $d$는 window size의 절반을 의미하는데 직접적으로 사용하진 않고 drift를 위한 notation을 위해 표현했다고 이해했다. 즉 $M_{i-d}$는 $[F_{i-2d}, F_{i-2d+1},..., F_{i-d},..., F_{i-1}, F_i]$중 최대값을 의미한다.
* 두번째로 이전 $p$개의 주기들을 보며 그 주기의 local maximum들의 최댓값을 현재 timestamp에서의 정상 변동의 최댓값으로 간주하기 위해 (5), (6)번 수식을 사용했다고 이해했다.

first-smoothing과 second-smoothing과정을 거쳐 우리는 다음과 같은 결과를 기대할 수 있다.  
![fig4]({{site.url}}/images/2023-02-18-anomaly_paper/fig4.png){: width="500" height="500"}

코드로는 다음과 같이 짜볼 수 있을 것 같은데 자신은 없다.. (혹시 이글을 보시는 분이 계시다면 피드백주시면 너무나 감사하겠습니다..ㅎ)
```python
def second_smoothing(F, l, p):
    S = [None] * len(F)
    for i in range(2*l*(p-1), len(F)):
        M = []
        if i > 2*l*(p-1):
            for j in range(2*l, 2*l*(p-1), l):
                m = max(F[i-j:i])
                M.append(m)

            delta_F = F[i] - max(M)
            S[i] = max(delta_F, 0)

    return S
```

위의 과정들을 논문에서 sudo code로는 다음과 같이 나타냈다.

![fig5]({{site.url}}/images/2023-02-18-anomaly_paper/fig5.png){: width="500" height="500"}

### 4.3 MOM-based Automatic Thresholding

* 데이터를 smoothing처리를 한 후 우리는 실시간으로 이상감지를 해야한다.
* 그러기 위해서는 데이터에 대한 threshold를 구할 수 있어야 하는데 본 논문에서는 SPOT이라는 알고리즘을 통해 threshold를 추출한다. 이 때 효율성을 더욱 향상시키기 위해 매개변수 추정 방법으로 MLE가 아닌 MOM(Method of Moments)을 활용한다.

#### 4.3.1 SPOT Algorithm
* SPOT은 streaming 버전의 Peaks Over Threshold(POT)라고 한다. POT은 모니터링 하는 데이터의 분포를 가정하는 것이 아닌 극단값의 분포를 가정한다. 
* SPOT의 핵심은 Generalized Pareto Distribution (GPD)의 꼬리 분포를 fit하는 것이라고 할 수 있다.
$\begin{align} \bar{F}_t(x)=P(X-t|X>t) \sim (1+{\gamma x\over{\sigma}})^{-{1 \over \gamma}}
\end{align}$
이제 위의 수식을 봐보자.
* $t$ : peaks를 탐색하기 위한 초기 임계값
* $\gamma, \sigma$ : shape, scale parameter
* $X$는 데이터 포인트를 의미하고, $X-t$는 임계값 $t$를 넘는 부분을 나타낸다. 
* 즉 (7) 수식은 임계값을 넘는 값 - 임계값이 GDP 분포를 따른 다는 것.
* 오리지널 SPOT과 다르게 본 논문에서는 $\hat{\gamma}, \hat{\sigma}$를 MLE가 아닌 MOM으로 추정한다.  
* 최종 threhold $th_F$는 다음과 같이 계산된다.
$\begin{align} th_F=t+{\hat{\sigma} \over \hat{\gamma}}(({qn \over N_t})^{-\hat{\gamma}}-1)
\end{align}$
* $q$: 이상을 결정하는 risk 계수, $n$: 현재 관측치의 수, $N_t$: $X_i$의 개수
* streaming pipeline동안 이러한 작업은 각 타임스탬프에서 한 번씩 업데이트 된다.  

전체적인 FluxEV의 구조는 다음과 같다.  

![fig6]({{site.url}}/images/2023-02-18-anomaly_paper/fig6.png){: width="500" height="500"}


