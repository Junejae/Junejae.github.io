---
layout: post
title: "[논문 리뷰] CHoRaL: Collecting Humor Reaction Labels from Millions of Social
Media Users"
categories:
  - 논문 리뷰
tags:
  - 논문
  - Labeling
  - Dataset
  - NLP
---

---
# 0. 사전 정보

- EMNLP 2021 Best Short Paper 선정.
- Written in Columbia University.
- Choral: chorus(코러스, 합창)의 형용사 형태.

# 1. Introduction

- 유머는 인간의 삶에 중요한 부분을 담당한다. → 유머 자동 탐지는 여러 문제에 적용할 수 있는 굉장히 중요한 task다.
- 하지만, 유머를 자각하는 것은 개인적인 요인이 상당히 많은 영향을 끼치는 탓에 논리적으로 annotate 하기 어렵다.
- 기존 연구들
    - ‘유머’와 ‘뉴스’를 구별하기 → 스타일이 서로 다른 도메인은 구별이 쉽지만, 같은 도메인 내에서 유머 여부를 탐지하는 task에 부적합.
    - SNS 글에 달린 해시태그 기준으로 글 전체의 유머를 탐지 → 방대한 양의 노이즈와 라벨링 정확성이 떨어지는 문제 발생.
    - 레딧의 upvote 수치를 기준으로 유머 라벨링을 결정 → 유머에 집중한 서브레딧 기반으로 이루어진 연구라 도메인 확장에 어려움이 발생.
- CHoRaL
    - A framework for Collecting Humor Reaction Labels.
    - 페이스북 포스트에 달리는 리엑션 기반으로 해당 포스트의 유머수치를 계산하는 프레임워크.
    - 장점
        - 추가적인 annotation 없이 페이스북의 모든 포스트의 유머여부를 라벨링 가능.
        - 유머/유머아님 라벨링 뿐만 아니라 유머 점수까지 제공.
        - SNS 상에서 유머와 관련된 text data를 대량으로 수집할 수 있는 기회를 제공.
- 본 연구진은 CHoRaL을 COVID-19와 관련된 약 78만 5천여개의 페이스북 포스팅 데이터셋을 라벨링 하는 데 사용함.
- 주제를 COVID-19으로 한정한 이유는 거의 모든 페이스북 유저에게 영향을 끼친 주제이기 때문.(도메인을 제한하면서도 편향성이 상대적으로 적은 편)
- CHoRaL은 COVID-19 뿐만 아니라 다른 주제의 페이스북 포스팅에도 쉽게 사용 가능.

# 2. CHoRaL Framework and Dataset

- 데이터 수집
    - 페이스북의 CrowdTangle 툴을 이용해 데이터를 수집.
    - COVID-19 관련 키워드를 이용하여 검색. (covid19, coronavirus, corona, covid 19, sars-cov-2, covid, sars cov 2)
    - 2020년 1월 20일 ~ 2021년 3월 18일 사이에 생성된 포스트를 수집
    - 언어는 영어로 한정
    - 포스트에 이미지나 비디오 등이 있는 경우를 제거
    - 총 2백만개의 포스트를 1차적으로 수집
- 데이터 정제
    - 중복, non-english 포스트 제거
    - 렌더링된 링크(미리보기)가 있는 포스트 제거
    - 렌더링 되어있지 않은 링크만 있는 경우, 링크를 스페셜 토큰으로 대체
    - 포스트의 총 문자 갯수가 500자를 넘지 않는 경우만 남김 (BERT 기반 모델에 맞추기 위함)
    - 최종적으로 약 78만 5천여개의 데이터 확보
- Humor Score(HS)
    
    ![Untitled](/assets/img/2022-05-03-paperChoral/0.png)
    
    - 페이스북에 기본 탑재된 이모지 리엑션 기능을 활용하여 유머 수준을 탐지
    - 가정: 전체 리엑션 중 ‘haha’ 리엑션의 비중이 높을 수록 포스트가 유머스러울 것이다.
        
        ![Untitled](/assets/img/2022-05-03-paperChoral/1.png)
        
    - h = (haha 리엑션의 수), t = (전체 리엑션의 수)
    - 전체 리엑션의 수를 50으로 나누고, 그 결과값을 tanh 함수에 넣어 나온 값을 곱한다 → 전체 리엑션 량이 적은 포스트의 유머 수치가 높게 나오는 경우를 방지
    - 전체 리엑션이 100개 이상인 포스트의 경우 tanh를 적용한 것과 하지 않은 것에 별 차이가 없어지게 됨
- Non-humor Score(NS)
    - 유머가 아닌(negative) 포스트의 구분도 잘 하게 만들고 싶었음.
    - 낮은 HS로 유머가 아닌 포스트를 판별 가능하지만, 이 경우 극단적인 케이스만 집중적으로 찾게 됨.(죽음 등의 슬픈 사건)
    - 전반적인 의미의 ‘유머가 아닌’ 포스트를 판별하려면 새로운 측정법을 도입할 필요가 있었음.
    - 가정: 대부분의 페이스북 포스트는 ‘유머가 아닌’ 포스트이다. → 페이스북 평균 리엑션 분포를 따르는 포스트는 ‘유머가 아닌’ 포스트일 가능성이 높다.
        - 증거: 대부분의 포스트는 낮은 HS 수치를 가지고 있었다.
        
        ![Untitled](/assets/img/2022-05-03-paperChoral/2.png)
        
    - t = (전체 리엑션 수), R = (리엑션 가짓수), r = (특정 리엑션), S = (표준 분포 내에서 r의 비율), O = (타겟 포스트 내에서 r의 비율)
    - 타겟 포스트 내의 리엑션의 MSE를 tanh로 normalize 한 뒤, 음의 로그를 취한 값을 NS로 정의.
    - NS 값이 높을 수록 페이스북 평균 수준의 포스트 → 유머가 아님.

# 3. Humor Analysis

![Untitled](/assets/img/2022-05-03-paperChoral/3.png)

- 정의한 HS, NS 등이 현실에 맞는지 증명하기 위해 다양한 공식과 툴을 사용.
- 안타깝게도 여백~~과 시간과 본인의 사전지식~~이 부족하여 자세한 설명은 생략.

# 4. Humor Detection Experiments

- 수집 된 78만 5천여개의 포스트 중 유머 포스트의 비율은 상당히 적은 편.(imbalanced)
- (높은 HS 수치를 가진 포스트 2만여개) + (높은 NS 수치를 가진 포스트 2만여개) = 총 4만여개 포스트를 실제 모델 훈련에 사용.
- train-test set 비율은 8대 2로 랜덤하게 분할.
- fine-tuning에 사용된 모델
    - RoBERTa-base: 위키, 뉴스, 그 외 웹 텍스트로 훈련된 BERT 계열 모델.
    - BERTweet: RoBERTa의 방법론과 영어 트윗 데이터로 학습된 BERT 계열 모델.
    - BERTweet-covid: BERTweet에 COVID 관련 트윗 데이터를 추가 학습시킨 모델.
- 라벨 종류
    - CHoRaL로 산출한 HS를 Ground Truth로 사용.(Continuous)
    - 유머글 여부를 나타내는 Binary Classification.(HS와 NS 값으로 라벨 결정)
- 하이퍼 파라미터
    - epoch = 3
    - lr = 2e-5
        
        ![Untitled](/assets/img/2022-05-03-paperChoral/4.png)
        
- Metric으로 F1-score와 AUC(Area Under Curve) 사용.
- 실험 결과 & 분석
    - 세 모델 모두 인간 수준의 유머 판별력을 확보.
    - SNS와 특정 주제에 맞게 추가 학습시킨 모델의 판별력이 더 높았음 → DAPT, TAPT가 중요.
    - Binary Label로 훈련시킨 결과가 더 좋았음 → HS 뿐만 아니라 NS도 유의미한 지표임을 확인.

# 5. Conclusions and Future Work

- 일련의 실험으로 CHoRaL 프레임워크는 모델에게 사람 수준의 유머 판별력을 학습시킬 수 있는 labeled dataset을 자동적으로 생성할 수 있음을 보여 줌.
- CHoRaL을 응용하여 공격적인 가짜정보 포스트와 단순 유머 포스트를 판별할 모델을 학습시킬 수 있을 것이라 기대.
- CHoRaL은 단순 유머뿐만 아니라 분노나 슬픔 등의 다른 인간적인 감정도 라벨링 할 수 있을 것.

# 6. 분석 & 감상

- 새로운 지표를 고안 할 때 통계학적 접근법이 기본적으로 필요함을 느낌
- 범용 인공지능의 길은 아직 멀고도 험하다
    - 본 논문에서도 사용된 데이터셋의 편향성에 대해 논하고 있음(SNS에 익숙한 영어 사용자에 편향 될 가능성)