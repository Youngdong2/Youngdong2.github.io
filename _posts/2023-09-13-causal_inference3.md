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

