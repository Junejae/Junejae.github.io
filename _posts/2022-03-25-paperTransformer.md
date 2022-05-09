---
layout: post
title: "[논문 리뷰] Attention Is All You Need"
categories:
  - 논문 리뷰
tags:
  - 논문
  - Attention
  - Transformer
  - Modeling
  - NLP
---

# 1. Introduction

- 지금까지 RNN 구조 모델, 특히 LSTM과 GRU는 인코더-디코더 구조, Attention 등의 개선을 거쳐가며 언어모델과 기계번역 Task에서 SOTA 성능을 내고 있었음.
- 하지만, RNN 모델은 이전 시점의 hidden state를 참조하여 현재의 hidden state를 계산하는 구조적 특성 상 컴퓨팅이 시퀀스를 따라 순차적으로 진행되고, 이는 런타임 부분에서 상당한 손해를 본다는 것을 의미함. 여러 개선책이 제안 되어 왔으나 모델의 구조로 인한 근본적인 문제를 해결하진 못함.
- Attention은 RNN의 성능 개선책으로 소개되어 지금까지 RNN 모델의 addon 정도의 역할만 수행하였음.
- 본 논문에서 소개하는 Transformer 구조는 RNN 형태의 구조를 완전히 배제하고, Attention을 전적으로 활용하여 input과 output 사이의 전역적인 관계성을 추출하는데 활용함.
- 이 Transformer 구조는 컴퓨팅 부분에서 기존의 RNN 대비 막대하게 향상된 병렬성을 확보하였고, 기계번역 task에서 8개의 P100 GPU로 12시간 학습한 것 만으로도 SOTA를 달성함.

# 2. Background

- RNN의 순차적 계산 문제를 해결하기 위해 CNN 기반의 언어 모델도 제안 되었으나, 단어 간의 거리가 길 수록 관계성을 계산하기 어려워 진다는 문제가 있었음.
- Self-attention: attention 연산의 대상이 input sequence, 자기 자신이라는 것. 따라서 Q,K,V 벡터는 모두 input vector에서 유래하게 됨. (기존 attention은 Q,K,V 벡터가 인코더&디코더의 hidden state에서 유래한다.)
- End-to-end memory networks: RNN 대신 Recurrent Attention Mechanism을 사용하여 질의응답 task 등에서 좋은 성능을 보임.
- 상기한 개선법들에 영향을 받아 오로지 self-attention만으로 이루어진 Transformer 구조를 고안하게 됨.

# 3. Model Architecture

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/80b0756a-9ecc-40d2-91be-05cc769fe9db/Untitled.png)

- Transformer는 기존 Encoder-Decoder 구조의 큰 그림을 그대로 따라간다.
- 내부 구조는 Self-Attention과 말단에 연결된 Fully Connected Layer
- Encoder (6개의 레이어)
    - 레이어 = Multi-Head Attention + FC
    - 각 서브 레이어마다 Residual Connection과 Layer Normalization 배치
    - Input Embedding까지 포함한 모든 서브 레이어들은 512 차원의 output을 반환
- Decoder (6개의 레이어)
    - 레이어 = Multi-Head Attention + **Multi-Head Attention(Encoder의 output을 참조)** + FC
    - Encoder처럼 각 서브 레이어마다 Residual Connection과 Layer Normalization 배치
    - Self-Attention 부분을 일부 수정하여 디코더가 이후의 포지션 정보를 엿보지 못하도록 조치(상세설명은 후술)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/89be5fc0-c974-4ff4-9a0f-7898840a4164/Untitled.png)

- Attention - 기존처럼 Q, K, V 벡터를 사용하되, 세 벡터 모두 같은 input에서 유래
    - Scaled Dot-Product Attention
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/578b0dfc-5396-4917-b337-b50654a3fc11/Untitled.png)
        
        - Attention 계산법은 크게 두 가지가 있음(Additive Attention & Dot-product Attention)
        - 이 중 행렬곱을 사용하는 Dot-product Attention을 사용하여 컴퓨팅 효율성을 추구
        - Dot-product Attention은 계산 크기가 커질수록 softmax시 과하게 적은 값을 가지는 스팟이 생길 수 있음(gradient에 악영향)
        - 따라서 softmax 연산 전 q,k의 길이의 제곱근으로 전체 값을 나눠줌으로써 사전 scaling 수행
        - Scaled Dot-Product Attention = scaling + dot-product
    - Multi-Head Attention
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d93f57d1-ecf9-435d-9d4f-3d30c58d16ba/Untitled.png)
        
        - 단일 Attention 모델 보다 다중 Attention을 사용하여 문장의 각 단어 요소들을 여러 측면으로 바라볼 수 있게 조치.
        - 이 때, 인풋 벡터를  Attention 갯수인 h개로 등분하여 각각을 Q, K, V 벡터로 활용해 연산
        - 이렇게 multi-head attention은 인풋의 길이는 동일하게 가져가면서 연산은 병렬화
        - 논문에서는 h = 8로 잡고, 원본 인풋의 길이는 512 이므로 각각의 Attention 모듈에 들어갈 Q, K, V 벡터의 차원을 일괄 64로 세팅
    - Multi-Head Attention의 3가지 적용 예
        - Encoder 모듈의 Self-Attention
            - 임베딩 된 input에서 Q, K, V를 사용함, encoder 내에서 일어나는 모든 연산과 그 결과물이 decoder까지 보존되어 전달
        - Decoder 모듈의 2번째 Attention(Encoder-Decoder Attention)
            - Q(Decoder), K(Encoder), V(Encoder) 구조로 디코더가 인코더의 모든 정보를 연산에 쓸 수 있게 만듬(기존 Encoder-Decoder RNN의 Attention 쓰임새를 모방)
        - Decoder 모듈의 1번째 Attention(Masked Self-Attention)
            - Encoder의 것과 구조적으로 동일하나, Decoder에 들어간 output의 다음 시계열 정보를 엿보는 것을 방지하기 위해 Masking 처리를 함(음의 무한대)
- Position-wise Feed-Forward Networks
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c7977019-057a-4f67-b81d-24708a264752/Untitled.png)
    
    - 각 모듈의 말단부에 FC with ReLU 부착
    - fc (with bias)  → ReLU → fc (with bias) 로 이해하면 편함
- Embeddings & Softmax
    - input, output을 Embedding 하는 레이어
    - Softmax로 확률분포 계산 전에 행하는 linear transformation 레이어
    - 상기 세 레이어는 weight를 공유
    - 다만, Embedding 레이어는 weight를 embedded 된 벡터 차원 수의 제곱근으로 일괄적으로 나눠서 축소화
- Positional Encoding
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d9be83f2-62aa-4bbd-9f21-c911001ec7b9/Untitled.png)
    
    - RNN, CNN 등이 없는 모델이므로, sequence 데이터의 포지션 정보를 따로 모델에게 제공할 필요가 있음
    - Positional encoding을 계산하고(상기 sin, cos 함수) embedded 된 결과물에 부착하여 포지션 정보를 녹여냄

# 4. Why Self-Attention

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e22fb461-4f2c-44f9-8f4f-cf09c30cce89/Untitled.png)

- Complexity per Layer: 레이어당 공간복잡도
- Sequential Operations: 순차적 계산에 드는 시간복잡도
- Maximum Path Length: 서로 거리가 긴 요소간의 관계성 계산에 필요한 네트워크 탐색 길이
- 기존의 RNN, CNN보다 Self-Attention의 컴퓨팅 효율성이 뛰어남을 알 수 있음
- Self-Attention(restricted): 효율성 증대를 위해 attention 계산을 위한 주변탐색의 범위를 r로 제한한 형태

# 5. Training

- 데이터 & 배치
    - 영-독 번역
        - WMT 2014 English-German dataset 사용
        - 450만개의 문장쌍
        - BPE로 토크나이징 (Max: 37000)
    - 영-프 번역
        - WMT 2014 English-French dataset 사용
        - 3600만개의 문장쌍
        - WordPiece 토크나이징 (Max: 32000)
    - 문장쌍들은 Bucketing 방식으로 비슷한 길이끼리 묶어서 배치화
    - 각 배치마다 토큰의 갯수는 source=25000여개, target=25000여개였음
- 하드웨어 & 스케줄
    - 8개의 P100 GPU를 사용하여 학습 (TMI: V100이 P100보다 3배 빠르고, 메모리도 2배 많음)
    - 연구진이 세팅한 베이스 모델 기준 step 당 0.4초 소요, big 모델은 1초 소요
    - 베이스 모델은 12시간동안 10만스텝을 거치며 학습, big 모델은 3.5일동안 30만스텝을 거치며 학습
- Optimizer
    - Adam 사용
    - betas=(0.9, 0.98), eps=1e-09 (Pytorch default: betas=(0.9, 0.999), eps=1e-08)
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/07b6cc2d-339c-4eed-a267-a2ca31400c96/Untitled.png)
    
    - LR은 warmup_steps=4000 동안 선형적으로 증가하게 세팅, 이후 상기한 공식대로 저하시킴
- Regularization
    - Residual Dropout
        - embedding과 모듈 내 sub-layer의 output에 일괄적으로 dropout=0.1을 적용
    - Label Smoothing
        - LabelSmoothing=0.1을 적용하여 모델의 학습을 어렵게 만들었음 → accuracy와 BLEU 상승

# 6. Results

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/33d08b35-ec87-4b18-8de6-5bfaebbc024f/Untitled.png)

- TMI: English <=>German 이 EN-DE로 표기된 이유는 German = Deutsch(도이치)라서
- 각 번역 task의 test 데이터셋은 newstest2014를 사용
- Big 모델이 BLEU score에서 SOTA 달성.
- Big, base 모델 둘 다 Training Cost 대비 월등히 효율적임을 보여줌
- Base 모델은 10분마다 설정한 checkpoint 중에서 마지막 5개의 checkpoint의 평균을 취함
- Big 모델은 마지막 20개의 평균을 취함
- 번역 task에 Beam search를 활용 (4-gram, alpha=0.6)  ← dev set으로 실험하며 결정

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f69b3775-1cb4-4383-85f3-6dc4aad0315b/Untitled.png)

- Transformer 구조에서 중요한 포인트들을 알아보기 위해 하이퍼 파라미터를 다변화 하며 실험
    - A) Attention 갯수는 일단 single보단 많아야 함을 확인
    - B) Attention에 쓰이는 Key의 size를 줄이면 성능이 하락하는 것을 확인
    - C) & D) 모델 사이즈가 커질 수록 성능이 향상됨을 확인, dropout을 주어 overfitting을 방지하는 것도 중요
    - Positional embedding을 고전적 방식으로 바꿔 보았으니, 성능 향상은 없었음(= sin, cos 계산법이 유효)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c7eab6e3-a859-4d99-888b-81f6f6af262c/Untitled.png)

- Transformer가 번역 말고도 다른 task에도 유효함을 보이기 위하여 영어 구문 분석 task도 수행
    - 구문 분석 특성 상 output 길이가 input 대비 아주 긴 경우가 일반적이라, RNN은 이 task에서 SOTA를 달성한 적이 없음
    - h=4인 transformer로 input=1024를 설정, Wall Street Journal 의 4천만개 단어를 사용해 학습, semi-supervised learning 기법도 동원
    - 본 실험을 위한 적절한 하이퍼 파라미터 세팅은 충분히 이뤄지지 않았음.(번역 Task용 세팅에서 미세조정만 함)
    - 그랬음에도 SOTA에 근접한 성능을 달성.

# 7. Conclusion

- Transformer는 sequence transduction 모델 중 self-attention만으로 이루어진 최초의 모델이다.
- 번역 task에서 Transformer는 기존 모델들보다 월등히 빠른 학습속도로 (기존 모델들의 앙상블도 넘을 수 없는)새로운 SOTA를 달성했다.
- 이 결과에 연구진들은 매우 고무되었고, 번역과 구문분석 task 뿐만 아니라 이미지, 오디오, 비디오 분야에도 transformer를 사용한 모델을 연구할 계획이다.

# 8. 정리

- Transformer 구조는 GPU의 빠른 행렬계산을 이용하여 컴퓨팅 병렬화를 추구함으로써 비약적으로 빠른 학습 속도를 달성 할 수 있었다.
- 이 구조에 대한 아이디어는 하루 아침에 나온 것이 아닌, 이전에 존재해왔던 다른 방법론들을 잘 취사선택하여 얻어낸 것이다.
- Transformer의 성능대비 연산자원 이점과, 구조 내 유의미한 하이퍼 파라미터 탐색 등에 초점을 맞춰 일관적으로 설계된 실험들이 인상적이다.