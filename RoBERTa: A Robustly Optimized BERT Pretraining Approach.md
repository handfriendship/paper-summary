# RoBERTa: A Robustly Optimized BERT Pretraining Approach
- Author : Yinhan Liu, Myle Ott, Naman Goyal, Jingfei Du, Mandar Joshi, Danqi Chen, Omer Levy, Mike Lewis, Luke Zettlemoyer, Veselin Stoyanov
- 2019 arxiv

**Summary**
- 기존의 BERT를 더 많은 데이터셋으로 더 오래, 그리고 여러 hyperparameter들 및 input 요소들을 바꾸어 실험했더니 더 좋은 성능을 낼 수 있었다는 논문
- 이 논문의 교훈은, 많은 연구들이 architecture나 objective를 개선하는데 치안이 되어있는데 반해, dataset, training strategy와 같은 mundane한 요소들을 신경써서 모델을 제대로 학습시키는 것도 중요하다는 것임.
- 더 많은 데이터, 더 오래 학습, masking 기법, 더 큰 batch size, NSP loss를 없앰, pretraining시 input format을 바꿈, byte-level BPE, 기타 여러 hyperparameter 수정.
- GLUE, SQuADv1.1/v2.0, RACE에서 좋은 성능을 보임.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- 

**Main Idea**
- 데이터셋 설계: 
  - RottenTomatoes의 영화평론 데이터셋에 phrase-level로 감성라벨을 붙였다.
    - 원래는 7-level(very positive, positive, somewhat positive, neutral, ...) 로 구분했는데, very positive과 positive를 합쳤다.
    - Amazon Mechanical Turk 3명을 고용해서 labeling했다.
    - 총 문장 개수는 11,855개 (RottenTomatoes의 개수와 같음), 총 태깅된 phrase개수는 215,154개.
  - 데이터셋 distribution
    - 5개의 label은 uniformly distributed되어있다.
    - n-gram이 낮을수록 neutral이 많고, n-gram이 높을수록 uniformly distributed되어있다.
- RNTN: 
    - 하나의 tensor를 이용한다는 것 같다.
    - 읽지 않아서 모르겠다.

**Contributions**
- summary에서 설명함.

**Experiments**
- 용어정리
  - negation: but, although 등으로 문장의 sentiment가 반전되는 것
- Fine-grained Sentiment For All Phrases
  - 낮은 n-gram을 맞추는 성능은 높고, 높은 n-gram을 맞추는 성능은 낮다.
  - 기존 bag-of-words방법론은 낮은 n-gram에선 약하고, 높은 n-gram(negation이 많아지는)에선 강하다. (아마 bag-of-words가 문장의 전체적인 통계 정보를 보는 방법이라 그런듯?)
- Negating Positive Sentences
  - positive sentence를 골라서 일부로 negate시킨 것. (positive -> negative로 바뀜)
  - RNTN이 젤 잘되더라.
- Negating Negative Sentences
  - negative sentence를 골라서 negate시킨 것. 
  - 'The movie was not terrible'
  - negative -> 덜 negative
  - RNTN이 젤 잘되더라.
  - Figure 8.: negate를 시켰을 때 어떤 문장의 감성 분류 결과의 positive속성 또는 negative속성이 바뀌는 정도

**총평**


## Study

**읽는데 걸린 시간**
- 읽는데 8:00 + 정리하는데 30분
- pages: 9 (without references&appendix)

**비판적 사고**
- Vanilla Transformer를 그대로 가져왔다는 것이 이 논문의 contribution이다.
  - Figure 1에서 소개된 Transformer Encoder는 Layer Norm의 위치가 변경되었는데, 왜 그렇게 했는지에 대한 설명이 있으면 좋겠다.
- Table 2에서 ViT-H/14, ViT-L/16 두가지 모델의 성능이 비교되었다.
  - Large->Huge, 16x16->14x14로 동시에 바꾸어서 무엇이 더 많은 영향을 미쳤는지 알기가 애매하다.(it is not clear ~ )
- 4.5에서 현재의 trainable embedding layer가 학습을 통해 sinusoidal structure를 배우기 때문에 따로 2D embedding을 주입해주는 것이 도움이 되지 않았다고 나와있다.
  - 현재 모델(transformer)은 inductive bias가 부족하기 때문에 transfer를 할 때 크기가 작은 dataset을 사용하면 따로 2D embedding을 사용하는 것이 도움이 된다고 유추할 수 있는데, 이것에 대한 실험이 있으면 더 좋을 듯 하다.

**이해못한 점**
- attention distance가 정확히 무엇을 의미하는지 잘 모르겠다. Figure 6을 기반으로 유추하면 attention weight가 큰 것은 attention distance가 가깝다는 말인데, 어짜피 attention weight의 sum은 1이기 때문에 Figure 7의 y축인 Mean attention distance를 구하면 layer에 상관없이 모두 같은 값을 가져야하는 것 아닌가?
