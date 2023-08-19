---
layout: single
title: "Session 2 인과추론을 위한 연구 디자인"
categories: Causal_Inference
use_math: true
comments: ture
toc: true
author_profile: false
---

본 포스팅은 인과추론 두번째 포스팅으로 인과추론을 위한 연구 디자인 방법에 대해 다뤄보도록 하겠습니다.

## Session 2.1 인과추론을 위한 연구 디자인

### Causal Hierarchy of Research for Causal inference

![fig1]({{site.url}}/images/causal_inference/session2-1.png "출처 : 인과추론의 데이터과학"){: width="700" height="700"}

위 피라미드는 인과추론을 위한 다양한 방법들을 보여줍니다.  
피라미드 위로 올라갈수록 인과추론의 수준이 높다를 의미합니다. 즉, 아래로 갈수록 인과관계의 신뢰도가 낮고 인과관계를 입증하기 어렵다는 뜻입니다. 한층씩 간단하게 설명해보면 다음과 같습니다.  

* Meta-Analysis : 기존의 여러 인과추론의 결과들을 종합적으로 분석하는 방법론.
* Randomized Experiment : 단일방법론으로 가장 수준이 높은 방법론. 하지만 현실에서는 실행하기 어렵다.
* Quasi-Experiment(준실험) : 실제 실험과 매우 유사한 상황을 찾아서 분석하는 준 실험은 randomized experiment에 가깝다고 기대할 수 있다.
* Instrumental Variable : 준 실험상황이 어려울 때 인위적 도구를 사용. 이는 인과추론을 방해하는 요인(내생성)을 인위적으로 제거하기 위함이다.
* 'Designed' Regression : 적절한 도구변수도 없을 때 적절한 톤제변수의 디자인을 통한 회귀분석도 적절한 인과관계를 추론할 수 있다. 이런 상황에서 Causal Diagram이 매우 유용할 수 있다.

## Session 2.2 인과추론의 정석: 무작위 통제실험

### Random Assignment

![fig2]({{site.url}}/images/causal_inference/session2-2.png "출처 : 인과추론의 데이터과학"){: width="700" height="700"}

ramdom assignment란 treatment group(실험군)과 control group(대조군)을 나눌때 실험참가자들의 특성이 평균적으로 두 그룹에 균일하게 분포되는 것을 의미합니다.  
여기서 중요한 점은 random assignment를 할 때, sample size가 충분히 커야하고, 두 그룹이 균등하게 나뉘었는지 확인해야 합니다.

### Example of Randomized Experments

다음으로는 Randomized Experments의 예시를 알아보겠습니다.  

![fig3]({{site.url}}/images/causal_inference/session2-3.png "출처 : 인과추론의 데이터과학"){: width="700" height="700"}
![fig4]({{site.url}}/images/causal_inference/session2-4.png "출처 : 인과추론의 데이터과학"){: width="700" height="700"}

실험 내용 : 교실 내에서 태블릿을 사용하는 것과 성적과의 인과관계  
treatment : 교실 내에서의 태블릿 사용 여부  
실험 조건 : 태블릿을 제외한 나머지 요인들(성별, 인종, 이전 성적 등)은 같아야 한다.
실험 결과 : 교실 내에서 태블릿을 사용하는 것이 성적을 유의미하게 낮춘다는 것을 확인할 수 있다.  

## Session 2.3 실험 아닌, 실험 같은 준실험

### Randomized Experiment vs. Quasi-Experiment

![fig5]({{site.url}}/images/causal_inference/session2-5.png "출처 : 인과추론의 데이터과학"){: width="700" height="700"}

Randomized Experiment와 Quasi-Experiment의 유일한 차이점은 treatment에 대한 assignment 방법입니다. 위 그래프는 random assignment가 아닐 때의 teatment assignment 방법 즉, Quasi-experiment일때의 방법을 보여줍니다. 이를 좀 더 알아보겠습니다.  

* self-selection : 연구 대상들이 스스로 treatment를 받을지 결정하는 방법.
* Exogenous Shock : 외부요인에 의해서 treatment와 control 그룹이 나뉘는 방법. 보통 자연실험이라 부른다.
* Discontinuity : 임의의 경계값을 기준으로 treatment와 control 그룹을 나누는 방법.

하지만 위 방법들은 treatment와 control 그룹을 명확히 나누기 쉽지 않다라는 한계점이 있습니다. 또한 전후의 데이터를 관찰할 수 없다면 Quasi-Experiment를 활용하기 힘듭니다.  
이럴 경우, treatment를 예측할 수 있고 동시에 오차항(결과에 영향을 줄 수 있지만 관찰할 수 없는 모든 변수)과 연관성이 없는 변수를 찾을 수 있느지의 여부를 판단해야 합니다. 만약 찾을 수 있다면 Instrumental Variable을 사용할 수 있을 것입니다.  
하지만, Instrumental Variable를 활용하기 어려운 경우, 관찰가능한 변수들만 가지고 선택편향을 통제하는 접근이 가능합니다. 이러한 방법으로는 Matching과 Regression이 있습니다.  

### Example (Exogenous Shock)

다음으로는 Exogenous Shock에 대한 예시를 확인해보겠습니다.  

![fig6]({{site.url}}/images/causal_inference/session2-6.png "출처 : 인과추론의 데이터과학"){: width="700" height="700"}

이 실험은 지방 재정 접근성이 기업 형성에 미치는 영향에 대한 실험입니다. 셰일 붐을 통해 treatment 그룹을 정의할 수 있습니다.  

* treatment group : 셰일유전이 발견되고 개발된 지역  
* control group : 셰일유전이 발견되지 않은 지역

이런 경우 Exogenous Shock은 아무도 예상하지 못하는 경우이기 때문에 어느정도 random하게 배정한 것과 유사하다고 인정할 수 있습니다.

### Example (self-selection)

![fig7]({{site.url}}/images/causal_inference/session2-8.png "출처 : 인과추론의 데이터과학"){: width="700" height="700"}

이 실험은 소셜미디어에서의 입소문의 효과에 대한 실험입니다. 이 연구의 treatment는 해당 기업의 팔로우이므로 쉽게 treatment와 control 그룹을 나눌 수 있습니다.

* treatment group : 팔로워 중 해당 기업을 팔로우한 그룹
* control group : 팔로워 중 해당 기업을 팔로우하지 않은 그룹

이 실험의 경우 두 그룹 모두 팔로워이기 때문에 어느정도 특성이 유사할 것이라고 주장하기 용이합니다.

## Session 2.4 준실험 분석도구: 이중차분법 & 회귀불연속

### Difference in Difference (DID)

![fig8]({{site.url}}/images/causal_inference/session2-9.png "출처 : 인과추론의 데이터과학"){: width="700" height="700"}

먼저 이중 차분의 의미에 대해 확인해보겠습니다.

* 첫 번째 차분 $(T_A-T_B)$ : treatment 그룹의 전후 변화
* 두 번째 차분 $(T_A-T_B)-(C_A-C_B)$: (treatment 그룹의 전후 변화) - (control 그룹의 전후 변화)

Conterfactual은 treatment가 없었다면 있었을 잠재적 결과입니다. 그렇다면 Conterfactual $T_A'$은 $T_B+(T_A'-T_B)$로 표현가능하고 이는 시간에 따라 변하지 않는 값과 시간에 따라 변하는 값으로 구분할 수 있습니다.  
하지만 $(T_A'-T_B)$는 관찰할 수 없습니다. Conterfactual은 관찰할 수 없기 때문이죠. 따라서 Conterfactual과 가까운 control 그룹의 시간에 따른 변화량과 비슷하다고 가정할 수 있다면 대체하여 Conterfactual을 추정하는 것이 DID의 핵심 아이디어입니다.  
즉, $T_B+(T_A'-T_B) \Rightarrow T_B+(C_A-C_B)$  
따라서 control그룹이 Conterfactual과 얼마나 유사한지 여부가 DID 분석의 신빙성을 결정하게 될 것입니다.

### Matching Techniques

Matching Technique은 비교가능한 control 그룹을 찾을 수 없을 때 사용할 수 있는 방법입니다.  
이는 우리가 가질 수 있는 변수들 중 평균적으로 유사한 샘플들만 서로 matching하는 것인데 PSM, CEM이라는 두 가지 방법이 있습니다.

#### PSM(Propensity Score Matching)

![fig9]({{site.url}}/images/causal_inference/session2-10.png "출처 : 인과추론의 데이터과학"){: width="700" height="700"}

이 방법은 treatment 그룹에 속할 확률(propensity score)를 계산하고 이 score가 유사한 샘플들끼리 서로 matching하는 방법입니다.  
하지만 모든 샘플을 하나로만 matching하기 때문에 변수가 굉장히 많다면 경우에 따라 어떤 변수들은 차이가 많이 날 수 있습니다. 이를 해결하기 위한 방법이 CEM입니다.

#### CEM(Coarsened Exact Matching)

![fig10]({{site.url}}/images/causal_inference/session2-11.png "출처 : 인과추론의 데이터과학"){: width="700" height="700"}

이 방법은 각 변수별로 구간을 나누고 구간에 따라 일치하는 샘플들끼리 매칭하자는 방법입니다.

### 회귀불연속

![fig11]({{site.url}}/images/causal_inference/session2-12.png "출처 : 인과추론의 데이터과학"){: width="700" height="700"}

Regression Discontinuity (RD)는 running variable의 모델링이 핵심입니다. 즉, 모델링에 민감한 방법이죠.  
RD는 discontinuous jump를 활용합니다. discontinuous jump가 없을 때의 상황을 모델링해서 counterfactual을 계산합니다. 즉, counterfactual과 실제 값과의 차이를 바탕으로 treatment effect를 구하는 것입니다.

## Reference

[인과추론의 데이터과학](https://youtube.com/playlist?list=PLKKkeayRo4PWyV8Gr-RcbWcis26ltIyMN)

