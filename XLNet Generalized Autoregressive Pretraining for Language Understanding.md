# XLNet: Generalized Autoregressive Pretraining for Language Understanding
- Author : Zhilin Yang, Zihang Dai, Yiming Yang, Jaime Carbonell, Ruslan Salakhutdinov, Quoc V. Le
- NeurIPS 2019
- 
**Summary**
- permutation LM을 도입하고, 그것을 도입할 때 생기는 문제점을 해결하기 위해 two-stream attention을 제시했다.
- permutation LM은 autoregressive기반으로 학습하되, token이 나열된 순서(factorization order)를 뒤섞은 뒤 학습시키는 전략이다. 이것은 AR, AE의 장점을 둘 다 갖는다.
- permutation LM을 도입하면 어느 시점에서 다음에 올 단어를 특정할 수 없다는 문제점이 있는데, 이것을 two-stream attention으로 해결했다. 
- two-stream attention은 query, key/value의 representation을 다른 방식으로 구하는 것으로, 맞출 단어를 예상해야할 query representation이 이미 query에 대한 정보를 포함하고 있는 경우를 방지하기 위한 목적으로 도입되었다.
- BERT보다 더 큰 데이터셋, 더 긴 시간동안 학습을 시켰고, SQuAD, GLUE, Classification 등에서 좋은 성능을 냈다.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
  - related works가 딱 2개만 소개되었음.
  -  NADE
  - autoregressive denoising을 text generation에 적용하려고 한 시도.

**Main Idea**
- Permutation Language Modeling
  - input의 factorization order를 permutation시킨 뒤, 그 중 하나의 order를 sampling해서 input으로 사용하는 방법.
  - idea: AR과 AE의 장점을 둘다 갖는 LM objective가 없을까?
    - AR의 장점: pretrain-finetune discrepency가 없음.(MLM에는 있음.), 예측할 단어들간의 dependency를 활용 가능.
    - AE의 장점: bidirectional information을 사용가능.
- Two-stream attention
  - query, key&value의 representation을 다른 방법으로 구하는 것.
  - idea: Permut. LM이 갖는 문제점을 해결하자.(다음에 맞춰야 할 단어가 unique하게 정해지지 않음.)
  - query-stream: 어떤 query의 representation을 뽑을 때, 다른 단어의 token정보 및 위치정보 + 해당 query단어의 위치정보만 이용하는 방법
  - content-strema: query를 제외한 다른 단어의 representation을 뽑을 때, 다른 단어의 token정보 및 위치정보 + 해당 query단어의 token정보 및 위치정보만 이용하는 방법
- Architecture
  - transformer-XL의 recurrence mechanism을 차용함.
  - idea: long-term dependency를 leverage하기 위함.
  - recurrence mechanism에서는 absolute positional encoding을 쓸 수가 없어서 relative positional encoding을 도입함.
    - (absolute positional encoding을 쓰게 되면 이전 memory segment의 첫번째 토큰과 현재 segment의 첫번째 토큰이 같은 poitional encoding을 갖는 문제점이 있음.)
- 기타 적용된 기법
  - partial prediction: permutation된 factorization order를 처음부터 예측하는 것이 아니라, 마지막의 일정 portion만 예측하는 방법
  - bidirectional data: pretraining 때 이전 segment만을 memory cell로 쓰는 것 뿐만 아니라, 다음 segment를 memory cell로 쓰는것.
  - multiple segment: NSP를 XLNet-base에는 적용, XLNet-large에는 적용안함.
    

**Contributions**
- permutation language modeling이라는 novel objective를 제시함. (기존 연구에 소개된 permutation LM보다 얼마나 더 바뀌었는지는 잘 모르겠지만..)
- permutation LM을 실현하기 위해 two-stream attention을 제시함.

**Experiments**
- Fair Comparison with BERT: 
  - BERT와의 성능 비교를 위해, 딱 architecture만 바꿔본 것.
  - RACE, SQuAD1.1, 2.0, GLUE 등
- Comparison with RoBERTa:
  - roberta를 겨냥한 실험.
  - RACE로 실험.
- Classification task
- SQuAD1.1, SQuAD2.0
- GLUE
- Ablation Study
  - partial prediction hyperparameter인 K를 조정해봄.
  - transformer-XL를 backbone으로 쓴 것에 대한 실험.
  - memroy cell, span-based prediction, bidirectional data, NSP objective에 대한 실험.

**총평**
- permutation language modeling이 신박하고, two-stream attention은 더 신박하다. 근데 실험 결과가 좀 자기들에 유리한 셋팅으로 놓고 한거같아서 아쉽다.

## Study

**읽는데 걸린 시간**
- 읽는데 2:35 정리하는데 1:00분
- pages: p9 (without references&appendix) 


**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- Q1. Recurrence Mechanism은 사실 LSTM과 비슷한 idea를 사용하겠다는 것인데, transformer의 computation적 장점 중 하나인 병렬 처리가 안되는 것이 아닌가? 즉, 이전 메모리 셀을 사용하기 위해서는 이전 segment가 완료될 때까지 기다려야 현재 segment를 계산할 수 있는 것이 아닌가?
- Q2. Table 6.의 ablation study에 보면 K=7일 때가 K=6일 때보다 더 잘된다. 상식적으로 K=6일때가 더 많은 case에 대해서 학습을 하는 것이므로 더 잘되어야 할 것 같은데, 그것에 대한 설명이 부족하다. 
- Q2-1. 만약 permutation AR말고 forward AR을 적용할 때도 마찬가지인가? 만약 forward AR의 경우에는 처음부터 맞추게 하는 것이 잘된다면, permutation AR이 원론적인 방법론적 문제를 갖고 있다는 소리 같다. 이미 그것에 대한 연구가 있을 수도 있을 것 같다.(T5나 이런데에 실험이 있을 것 같은데..) 찾아보면 좋을듯.
- Q3. 몇 epoch을 돌았는지가 안나와있고 step으로 나와있던데, 그게 나와있으면 permutation의 expectation이 몇개의 표본으로 구해진지를 알 수 있는 거니, 그 정보가 안나온 것이 아쉽다. 

**이해못한 점**
- pretraining때 next sentence prediction(NSP)를 진행할 때 이전 memory cell에 어떤 정보가 들어가있게 되는지 잘 모르겠다. 
같은 document내에서 추출된 정보만이 들어가있게 되면, previous segment: [cls, a, sep, b, sep]에서 a가 같은 document의 것이라고 치면, a만 memory cell에 남긴다는 뜻인가? 이게 말이 되는지, 구현적으로 이게 어떻게 실현되는지 궁금하네.
