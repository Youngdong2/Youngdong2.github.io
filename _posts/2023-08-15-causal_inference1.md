---
layout: single
title: "Session 1 인과추론의 다양한 접근법"
categories: Causal Inference
use_math: true
comments: ture
toc: true
author_profile: false
---

AIOps에서 중요한 부분 중 하나는 장애에 대한 원인이 무엇인지 찾는 것입니다. 이러한 연구분야를 Root Cause Analysis(RCA)라고 하는데 회사에서 최근에 이 연구에 대한 업무를 맡게 되었습니다.  

장애에 대한 근본원인을 찾기 위해서는 도메인 지식도 매우 중요하지만 인과관계, 특히 Causal Discovery분야(데이터를 통해 인과관계를 추론하는 분야)에 대한 이해가 중요하다고 생각되어 인과추론 스터디를 시작하게 되었습니다.  

이번 포스팅부터 시작하여 [인과추론의 데이터과학](https://youtube.com/playlist?list=PLKKkeayRo4PWyV8Gr-RcbWcis26ltIyMN)을 정리하는 글을 써보려고 합니다. 

# 인과추론의 다양한 접근법

## Frameworks for Causation
인과관계에 대해 모든 사람들이 동일하게 이해하고 동일한 방식으로 검증할 수 있는 수단이 필요합니다. 그런 역할을 하는 것이 인과관계에 대한 framework입니다.(공통의 이해에 대한 틀)

## Various Approaches to Causation
인과관계에 대한 다양한 접근 방법을 소개합니다.
1. 이론에 기반한 가설적 인과관계
* 수학적 법칙이나 논리적 추론에 따라서 인과관계가 형성될 수 밖에 없는 당위성(logical imperative)을 구축하여 인과관계를 밝히고자 하는 접근방법.

2. Statistics-Based Approach
* 통계적 비편향성으로 인과관계를 정의하는 방법.
* 모집단에서 여러번 샘플링을 하고 각 샘플에서 원인과 결과에 대한 관계를 추론했을 때 평균적으로 모집단에서 가지고 있는 실제 관계에 가깝게 추론할 수 있다는 방법.
* 인과추론을 방해하는 요인으로써 Endogeneity(내생성)를 강조하고 있음.
* 장점 : Endogeneity에 대해 수학적으로 계산할 수 있고, 이는 Endogeneity에 대해 얼마나 효과적으로 줄일 수 있는지에 대해 평가하거나 평가를 위한 통계지표를 개발하는 데 있어 유용하다.
* 단점 : 이론적으로는 Endogeneity를 측정할 수 있지만, 실제로 데이터분석 관점에서는 구체적 가이드를 주지 못한다.

3. Design-Based Approach
* 인과관계를 연구 디자인적 방법으로 인과관계를 정의.
* 장점 : 적절한 연구 디자인만 설정할 수 있다면, 인과추론에 대한 깊이있는 이해 없이도 충분히 인과적 효과를 추정할 수 있다.
* 단점 : 구체적으로 어떤 인과구조에 대해 직접적으로 알 수 없다.

4. Structure-Based Approach
* 원인과 결과가 서로 얽혀있는 인과적 구조를 추정하고자 하는 접근방법.
* 장점 : 인과구조에 대한 관계를 직접적으로 추정하여 원인이 결과에 어떤 영향을 주는지에 대한 path에 대해 보여줄 수 있다.
* 단점 : 추정 결과는 인과구조에 기반하고 있기 때문에 구조를 잘못산정하게 된다면 완전히 잘못된 결과를 도출할 수 있다.

![fig1]({{site.url}}/images/causal_inference/session1-1.png "출처 : 인과추론의 데이터과학"){: width="700" height="700"}

위 이미지에서 볼 수 있듯이 Statistics-Based Approach는 인과추론에 대한 이론적 기반을 제공할 수 있고, Design-Based Approach와 Structure-Based Approach는 데이터분석의 전력을 고안해줄 수 있습니다.  
Design-Based Approach는 어떤 요인들이 인과구조를 그리는 데 있어 포함이 되어야 하는지에 대한 인사이트를 줄 수 있고, 반대로 인과구조에 대한 이해는 연구 디자인을 고안하는 데 있어 중요한 인사이트를 줄 수 있습니다. 즉, 인과추론에 있어 완벽한 feramework는 없고, 각각이 서로 상오보완적으로 작동할 수 밖에 없죠. 때문에 서로 다른 접근 방법에 대해 충분히 이해해야 혹, 연구 목적에 맞는 적절한 framework와 상응하는 적절한 데이터를 수집하고 그에 맞는 적절한 분석방법론을 채택하는 것이 중요합니다.

# Potential Outcomes Framework
## Design-Based Approach to Causation
