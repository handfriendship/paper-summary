# AN IMAGE IS WORTH 16X16 WORDS: TRANSFORMERS FOR IMAGE RECOGNITION AT SCALE
- Author : Alexey Dosovitskiy∗,†, Lucas Beyer∗, Alexander Kolesnikov∗, Dirk Weissenborn∗, Xiaohua Zhai∗, Thomas Unterthiner, Mostafa Dehghani, Matthias Minderer, Georg Heigold, Sylvain Gelly, Jakob Uszkoreit, Neil Houlsby∗,†
- 2021 ICLR

**Summary**
- Transformer를 이용해 image classification 문제를 푸는 ViT를 제안했다.
- image를 patch단위로 잘라서 NLP의 sequence와 같은 형태로 처리하도록 했다.
- 기존의 approach인 CNN + self-attention을 섞어서 쓰는 것이 아니라, Vanilla Transformer를 그대로 가져와서 쓰는 것이 더 효과적이라고 주장했다.
- 그 이유는, (1)큰 데이터셋에서는 inductive bias를 주입하는 것 보다는 데이터셋의 패턴 그대로를 배우도록 하는 것만으로도 충분하다고 한다.
- (2) Transformer를 사용하는 것이 computational cost 측면에서 더 효과적이다.
- 적절한 position embedding, position embedding을 incorporate하는 방법, 적절한 patch size, 적젏한 dataset size 등을 조사하는 실험을 했다.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- CNN + self-attention: CNN을 이용해서 preprocessing을 하고, 그 결과물에 self-attention을 적용하려는 시도들. 하지만 GPU를 쓰기위해 고난도의 engineering이 필요하다.
- : 
- transformer + image patch: ViT와 접근법은 똑같음. 하지만 patch size가 너무 작았다.

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
