# Neural Geometric Parser for Single Image Camera Calibration
- Author : Richard Socher, Alex Perelygin, Jean Y. Wu, Jason Chuang, Christopher D. Manning, Andrew Y. Ng and Christopher Potts
- 2013 EMNLP

**Summary**
- 논문에서 두 가지 contribution을 했다. 하나는 RottenTomatoes의 영화평론 데이터셋에 phrase-level로 감성라벨링을 한 것, 또 하나는 RNTN model을 설계한 것.
- 원래 RottenTomatoes 영화평론 데이터셋은 sentence-level로 binary sentimental classification(pos, neg)을 하는 데이터셋이다. 여기서 각각의 phrase마다 
5가지 감성label중 하나를 부여했다. (--, -, 0, +, ++)
- RNTN모델은 sentence-level label뿐만 아니라 각 phrase-level의 태깅 정보를 이용해서 최종 clasification을 한다.
- 기존 모델과의 차별점은, 기존 모델들은 weight matrix를 여러개 써서 word vector학습, aggregating function을 학습했는데, RNTN은 하나의 weight tensor만 썼다는 것 같다.
- 기존 모델들보다 좋은 성능이 나왔다.

**Related Works**
- RNN (standard Recursive Neural Network)
  - input vector를 d-dim random vector로 initialize한다.
  - dxd word embedding matrix로 학습시킨다.
  - embedding matrix를 곱한 결과를 softmax에 넣어서 각 phrase를 학습시킨다.
  - 각 phrase를 나타내는 vector는 bottom-up 방식으로 하나씩 aggregate된다.
  - aggregate하는 함수는 tanh(W(dxd) x phrase-vector(dx1)) 이다. 
- MV-RNN (Matrix, Vector RNN)
  - RNN과 비슷한 방식을 쓰는데, 하나의 input word를 (W, v)로 표현한다. (하나의 word representation matrix, 하나의 word representation vector)
  - aggreagate방법: (b, B), (c, C)가 있으면 p_1 = aggregate(Cb, Bc), P_1 = aggreagate(B, C)

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
- 읽는데 2:40 + 정리하는데 20분
- pages: 10 (without references, model설명)
