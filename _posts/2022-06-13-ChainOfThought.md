---
layout: post
title: "[논문 리뷰] Chain of Thought Prompting Elicits Reasoning in Large Language Models Media Users"
categories:
  - 논문 리뷰
tags:
  - 논문
  - Prompting
  - Large Language Model
  - NLP
---

---

# 1. Introduction

![Untitled](/assets/img/2022-06-13-ChainOfThought/Untitled.png)

- GPT-3같은 Large Model은 종전의 모델들보다 확실히 뛰어난 성능을 보였다.
- 하지만, Large Model은 arithmetic, commonsense, symbolic reasoning 같은 NLP의 오랜 난제를 여전히 제대로 풀 수 없었다.
- 본 논문의 연구진은 단 2가지 key idea의 조합을 사용하면 해당 task의 성능을 비약적으로 상승 시킬 수 있는 것을 확인하였다
    - 여러 단계에 걸친 계산 스텝 → [인풋, 중간결과, 아웃풋] 포멧 = **Chain of Thought**
    - Task에 맞는 [input, output]의 예시를 보여주는 few-shot learning = **Prompting**
- 이 2가지의 조합을 **Chain of Thought Prompting**이라 명명하였고, 본 테크닉만을 활용하여 기존 pretrained large model을 사용하여 벤치마크 테스트를 한 결과, arithmetic의 경우 SOTA를 달성함.

# 2. Chain of Thought

![Untitled](/assets/img/2022-06-13-ChainOfThought/Untitled%201.png)

- 답을 찾기 위해 인간이 순차적으로 생각하는 방식에서 아이디어를 얻음.
- 여러 단계에 걸친 계산 스텝 → [인풋, Chain of Thought, 아웃풋] 포멧의 예시를 모델에게 보여줌으로써, **large model에 이미 내재된 단계적 추론 능력**을 끌어내는 것.
- model이 내놓은 output이 틀렸더라도, Chain of Thought을 분석하여 어느정도의 원인을 알 수 있음.
- 본 method의 효용성을 증명하기 위해, arithmetic, commonsense, symbolic reasoning 부문에서 성능 평가를 진행.

# 3. Arithmetic Reasoning

![Untitled](/assets/img/2022-06-13-ChainOfThought/Untitled%202.png)

- Arithmetic Reasoning - 초등학생 수준의 말로 풀어 쓴 수학 계산 문제를 푸는 task
- PALM 540B 모델은 CoTP를 활용하여 task-specific하게 finetuned 된 모델의 성능을 압도하고, GSM8K 벤치마크에서 SOTA 달성.
- Prompting에 쓰일 Chain of Thought는 벤치마크의 train 데이터셋 기반으로 연구진이 직접 제작.
- 실험에 쓰인 모델: GPT-3, LaMDA, PaLM
- 결과: CoTP 가 일반 Prompting 대비 확실한 성능 향상을 끌어냄, 일부 벤치마크의 경우 SOTA를 갱신

![Untitled](/assets/img/2022-06-13-ChainOfThought/Untitled%203.png)

- 모델이 반환한 Chain of Thought을 연구진이 직접 분석한 결과, 대부분이 논리적으로 구성되어 있었음.

![Untitled](/assets/img/2022-06-13-ChainOfThought/Untitled%204.png)

- Chain of Thought가 아닌 다른 방법론을 적용하여 실험 한 결과 Chain of Thought의 성능이 가장 좋은 것으로 나타남.

![Untitled](/assets/img/2022-06-13-ChainOfThought/Untitled%205.png)

- Chain of Thought의 대략적인 포멧을 유지하면서 다른 annotator, 문제 형식으로 구성해도 여전히 기존 prompting보다 월등한 성능을 보임.

# 4. Commonsense Reasoning

- 언어 모델의 일반 상식에 대한 이해를 측정하는 Task
- Prompting에 쓰일 Chain of Thought는 벤치마크의 train 데이터셋 기반으로 연구진이 직접 제작.
- 결과: CoTP 가 일반 Prompting 대비 확실한 성능 향상을 끌어냄, 일부는 기존 SOTA 갱신, 더 나아가 Human score를 넘어섬.

![Untitled](/assets/img/2022-06-13-ChainOfThought/Untitled%206.png)

# 5. Symbolic Reasoning

- Last Letter Concatenation: 이름의 마지막 철자들만 붙여서 반환 (ex: “Amy Brown” → “yn”)
- Coin Flip: 인물들이 동전을 뒤집거나/안 뒤집는 행동을 서술한 문장을 주고, 최종적으로 동전의 어느 면이 위를 향해 있는지 추론
- 결과: 두 task 모두 CoTP 가 일반 Prompting 대비 우세한 성능을 보임. 특히 Out of Domain(난이도를 높임)에서의 성능향상이 두드러짐

![Untitled](/assets/img/2022-06-13-ChainOfThought/Untitled%207.png)

# 6. Conclusions

- Chain of Thought Prompting이란 모델 입력으로 단계적 사고를 예시로 보여줌으로써 pretrained 모델에 내재된 Chain of Thought 성능을 끌어올리는 방법론이다.
- 이를 통해 task specific finetuning 없이 오직 Prompting 만으로 효율적인 성능을 낼 수 있다.
- 모델 아웃풋에 보여지는 Chain of Thought이 인공신경망의 정확한 생각흐름을 보증하진 않는다.
- 후속 연구로 ‘lets think step by step’을 인풋 최후미에 붙여줌으로써 large language model이 생각의 흐름을 아웃풋으로 반환하는 zero-shot reasoning이 가능함이 밝혀졌다. ([https://arxiv.org/pdf/2205.11916.pdf](https://arxiv.org/pdf/2205.11916.pdf))