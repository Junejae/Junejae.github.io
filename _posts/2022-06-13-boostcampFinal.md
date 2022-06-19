---
layout: post
title: "[부스트캠프 AI Tech 3기] 최종 프로젝트 정리 & 회고"
categories:
  - BoostCamp
tags:
  - 부스트캠프
  - AI
  - Deep Learning
  - NLP
  - GPT
  - BART
  - Product
  - Dataset
  - Project
---
![Untitled](/assets/img/AITech로고.png)

## 🧭개인 학습 목표

- 생성 모델의 훈련부터 generation까지 혼자 코딩해보기
- Selenium을 통한 웹페이지 크롤링 시도해보기
- Streamlit을 이용한 프로토타입 시연 웹페이지 제작해보기
- Pytorch Lightning과 Huggingface가 혼재된 스타일의 코드를 Huggingface만 사용하는 스타일로 리팩토링 하기
- 텍스트 감정분석 모델을 활용하여 의미있는 작업에 활용하기

## 📖사용한 지식 & 기술

- Pre-trained LM: KoBART, KoGPT2, KcELECTRA
- Metric: Rouge, 공감수치, sBERT 유사도, glove 유사도
- Hyperparameter Tuning: batch size, epoch, learning rate, weight decay
- Web Service: Streamlit, FastAPI
- Dataset: KOTE, Pandas
- ETC: Pytorch, Huggingface, Selenium

## 🧑🏻‍🎓결과 ⇒ 깨달음

- Selenium을 활용하여 서울 지역 내 카페/디저트 업종에 종사하는 요기요 가맹점의 손님 리뷰-사장 답글 조합의 데이터를 약 25만개 획득할 수 있었다.
    - 섬세한 데이터 수집 가이드라인과 코드 로직이 있다면 웹상에서 양질의 NLP 데이터를 손쉽게 획득할 수 있음을 깨달았다.
- 서울대 KOTE 데이터셋으로 finetuning된 KcELECTRA 모델을 활용하여 리뷰 텍스트의 감정을 분석한 결과, 손님 리뷰-사장 답글 간의 감정적 공감대를 발견하여 수치화 할 수 있었다.
    - NLP 모델을 활용하여 새로운 metric을 고안하여 개발 과정에 적용할 수 있음을 깨달았다.
- 생성모델로 많이 사용되는 KoGPT-2 말고도 KoBART를 finetuning 시켜서 이용 해 본 결과, 트렌디한 스타일의 답글을 잘 생성하는 것을 확인 할 수 있었다.
    - 다양한 언어모델들을 활용하여 각 성격별로 어울리는 세부 task를 배정할 수도 있음을 깨달았다.

## 🛠️새롭게 시도한 변화 ⇒ 효과

- 수집한 데이터셋을 python arrow 데이터셋화 하여 학습과 분석에 이용하였다.
    - 학습과 분석 코드의 실행속도가 빨라졌고, 데이터 조작 등에도 pandas보다 더 손쉽게 사용할 수 있어서 일의 능률이 올라갔다.
- 기존 KoBART-summarize 코드의 Pytorch Lightning 활용 부분을 모두 Huggingface 스타일로 치환하여 전체 코드가 Huggingface 기반으로 돌아가도록 리펙토링 하였다.
    - 더 직관성 있는 코드가 만들어진 덕에, 다른 팀원들에게도 더 직관적인 설명을 할 수 있게 되어 협업 효율성이 상승하였다.
- 서울대 KOTE 데이터셋으로 finetuning된 KcELECTRA 모델을 활용하여 리뷰 텍스트의 감정을 분석하고, 이 결과를 기반으로 공감수치라는 새로운 metric을 고안하여 모델 테스트에 이용하였다.
    - 원본 대비 모델이 생성한 사장 답글의 퀄리티를 정량적으로 평가할 수 있었다.

## 🏗️실수 & 한계 ⇒ 개선 방안

- 개발 시간 제한에 쫓겨 언어 모델로 다양한 실험을 하지 못하여 최적의 결과물을 낼 수 없었다.
    - 다음 프로젝트부터 모델 실험을 더 체계적으로 구성하여 실험 효율을 더욱 끌어올릴 것이다.
- KoBART 코드 리펙토링에 성공한 뒤, 같은 방식으로 KoGPT-2 코드도 Huggingface 스타일로 리펙토링을 하려 시도했으나, 만족스런 결과가 나오지 않았다.
    - 인풋과 타겟 구조를 바꿔 절반의 성공을 거뒀으나, 팀원들이 사용하기엔 부적합한 상황이었다. 모델에 대한 이해도를 더 높였어야 했다.

## 🆕다음 프로젝트에서 새롭게 시도해볼 것들

- 즉각적인 개인 실험 기록 관리
- git의 rebase 기능을 활용한 업데이트 로그 가시성 확보
- 논문에 근거한 추론
- 클라우드 베이스 배포