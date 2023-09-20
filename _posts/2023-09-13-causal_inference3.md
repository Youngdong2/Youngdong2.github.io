---
layout: single
title: "Session 7 인과 그래프"
categories: Causal_Inference
use_math: true
comments: ture
toc: true
author_profile: false
---

이번 포스팅에서는 지난 session2에 이어 session7 인과 그래프에 대한 강의를 정리해보도록 하겠습니다.  

## Session 7.1 인과 그래프

데이터 분석에 기반한 인과추론은 크게 두 가지 접근법으로 구분해볼 수 있습니다.

* Design-based approach : potential outcome framework를 기반으로 디자인할 수 있고, 적절한 디자인을 통해 선택편향을 제거하는 접근방법.
* Structire-based approach : 인과관계의 구조를 명시적으로 나타내고, 구조를 고려해 인과관계 이외의 path를 차단함으로써 인과관계를 추려내는 방법.

### Causal Diagram: Directed Acyclic Graph (DAG)

먼저 그래프에 대한 간단한 정리를 하고 시작하겠습니다.  
Directed Acyclic Graph(DAG)란 변수들간의 영향을 화살표로 표현하는 그래프를 formal하게 나타낸 그래프 입니다. 여기서 Directed는 방향이 있다는 것을 의미하고, Acyclic이란 순환 고리가 없다는 뜻입니다.

![fig1]({{site.url}}/images/causal_inference/session7-1.png "출처 : 인과추론의 데이터과학"){: width="700" height="700"}

다음으로 DAG에서 relationship 종류에 대해 알아보겠습니다.

* 직접적인 원인 (Direct Causal Effect)
![fig1]({{site.url}}/images/causal_inference/session7-2.png "출처 : 인과추론의 데이터과학"){: width="100" height="100"}

* 간접적인 원인 (Indirect Causal Effect)
![fig1]({{site.url}}/images/causal_inference/session7-3.png "출처 : 인과추론의 데이터과학"){: width="100" height="100"}

* 교란 요인 (Fork)
![fig1]({{site.url}}/images/causal_inference/session7-4.png "출처 : 인과추론의 데이터과학"){: width="100" height="100"}

* 충돌체 (Collider)
![fig1]({{site.url}}/images/causal_inference/session7-5.png "출처 : 인과추론의 데이터과학"){: width="100" height="100"}

여기서 직접적인 원인과 간접적인 요인은 직관적으로 이해할 수 있지만, 교란 요인과 충돌체는 그러지 않습니다. 뒤에서 좀 더 구체적으로 설명해보도록 하고 일단 넘어가보겠습니다.

### Association in Causal Diagram

DAG를 통해 그래프를 그리면 상관관계(또는 Association)을 쉽게 파악할 수 있습니다. 또, 아래 그림과 같이 다양한 Association에 대해 분석할 수 있다는 것이 장점입니다.  

![fig1]({{site.url}}/images/causal_inference/session7-6.png "출처 : 인과추론의 데이터과학"){: width="500" height="500"}

위 그림에서 X가 원인, Y가 결과라고 했을 때 direct causal effect와 indirect causal effect를 제외한 모든 path를 **backdoor path**라고 합니다. 즉, X $\to$ W $\to$ Y를 제외한 모든 path들은 backdoor path입니다.  

이러한 의미로 봤을 때 이러한 causal diagram에서 인과관계를 추론한다의 의미는 모든 backdoor path를 차단한다와 같습니다.  

다음으로는 강의에서 용어에 대한 설명을 했습니다.  
causal diagram에서 원인변수 X와 변수 Y사이에 정보 흐름이 없다면 **d-separated**라고 합니다. 그 반대는 **d-connected**라고 합니다. 아래 예시에서 X와 Y는 d-connected입니다. 정보의 흐름을 차단하는 것을 controlling for 또는 conditioning on이라고 표현하는데, A가 conditioning되었다면 X와 Y는 d-separated됨을 확인할 수 있습니다. A,B도 마찬가지겠죠. 하지만 C를 차단했을 때는 위의 경로가 열려있기 때문에 d-connected입니다. C,E를 차단했을 때도 마찬가지 입니다. 주의할 점은 A, D를 차단했을 때 d-connected라는 것입니다. 이는 collider의 개념때문입니다.  

![fig1]({{site.url}}/images/causal_inference/session7-7.png "출처 : 인과추론의 데이터과학"){: width="500" height="500"}

 아래 그림은 정보를 차단했을 때 정보의 흐름을 나타냅니다. 마지막 Collider를 보면 Z가 차단되었을 때 정보가 튕겨나와 d-connected가 됩니다.

 ![fig1]({{site.url}}/images/causal_inference/session7-8.png "출처 : 인과추론의 데이터과학"){: width="700" height="700"}
