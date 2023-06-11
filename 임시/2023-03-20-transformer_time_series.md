---
layout: single
title: "Trnasformer를 이용한 시계열 예측 실습"
categories: Deep Learning
tag: Python
author_profile: false
---

최근 시계열 예측 모델은 트랜스포머 계열이 많이 보입니다. 이런 최신 모델을 쉽게 활용하기 위해 기초가 되는 트랜스포머를 이용한 시계열 예측에 대한 실습을 포스팅하겠습니다.

## 1. 데이터

시계열 예측 모델의 input으로는 크게 세가지 모양의 데이터가 들어갑니다.  
첫번째는 Univariate입니다. Univariate는 하나의 시계열 변수만 사용하게 되는데 이때는 input이 되는 변수의 이후 시간 값을 예측하게 됩니다.  
두번째는 Multivariate입니다. Multivariate는 여러개의 시계열 변수를 사용하여 하나의 변수의 이후 시간 값을 예측하게 됩니다.  
세번째는 input이 여러 변수가 되고 output도 그 여러 변수가 되는 형태입니다.  
이번 실습에서는 두번째 형태인 Multivariate 데이터로 ETTh1 데이터를 사용하겠습니다. 이 데이터는
