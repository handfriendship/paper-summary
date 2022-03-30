# Domain-Adversarial Training of Neural Networks
- Author : Yaroslav Ganin et al.
- 2016 JMLR
- 
**Summary**
- source sample과 target sample의 discriminator를 둬서 둘을 구분하기 어렵도록 학습하여 domain adaptation을 잘 하는 DANN 모델을 제시하고, 수학적 근거를 제시함.
- feature를 뽑는 backbone을 두고, 2개의 head를 둠. 하나는 sample의 +/- clasifier, 다른 하나는 domain classifier.  
- domain classifier는 domain 구분을 잘하도록, backbone은 domain 구분을 못하도록 adversarial하게 학습함.
- domain classiifer를 gradient reversal layer를 둠. doamin classifier에서 domain classification을 잘하도록 하는 gradient의 부호를 바꾼 후에 backbone에 흘려보냄.
- 이러한 논리의 토대가 되는 수학적 근거를 제시했고, 여러가지 실험을 통해 자신의 방법론이 다양한 상황에 잘 적용됨을 보임.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- domain adaptation: labeled source data를 가지고 unlabelled target data를 classify하는 문제.
- source domain과 target domain의 raw dataset을 전처리 전에 mapping해서 같은 공간으로 옮기는 approach
  - source domain sample을 reweighting하거나 일부만 select함.
  - kernel trick, kernel reproducing Hilbert
- source와 target의 representation이 같은 공간에 오게 projection시키는 방법.
  - autoencoder를 이용하는 방법: source sample과 target sample을 label을 빼고 이용함. 다시 각각의 sample을 복원하는 방식으로. autoencoder를 backbone으로 feature를 뽑은 다음, 그걸로 classifier를 학습시킴.
    - two step이 필요함. 우리 모델보다 도 더 복잡함. 
- Adjustment or iterative approach
  - target domain sample에 pseudo-label을 준다.


**Main Idea**
- domain adaptation에서 target error의 upper bound를 source error와 domain divergence로 나타낼 수 있음.
  - target error <= source error + domain divergence
- Eq(1): 
  - H-divergence라는게 있음. 이것은 어떤 hypothesis class H 내의 모든 모델에 대해 source domain distribution과 target domain distribution간의 divergence를 나타낸 것.
    - H는 모델의 capacity라고 보면됨.
  - H가 symmetric일 때 empirical H-divergence를 구한 것이 Eq(1)
  - typo가 있음. (source domain indicator가 1인지 아닌지를 판별해야하고, target domain indicator가 0인지 아닌지를 판별해야함.)
  - H 공간에서 가장 성능이 나쁜 classifier가 평가했을 때 error율=1 이라면 H-divergence = 0 가 됨. (domain을 전혀 구분하지 못함.)
- Proxy A-Distance(PAD): 
  - 용어의 직관적 정의: Eq(2)의 집합 내의 모든 부분집합을 어떤 backbone에 먹여서 rep.를 만듦. 그걸 어떤 domain classifier에 먹였을 때의 결과 중 domain 구분이 가장 잘 되는 부분집합에서의 성능.
  - empirical H-divergence를 실제로 구하기 위해 만든 metric이라고 생각할 수 있음. 
  - Eq(2) dataset을 이용해 PAD계산 가능.
- model architecture:
  - feature를 뽑는 backbone을 두고, 2개의 head를 둠. 하나는 sample의 +/- clasifier, 다른 하나는 domain classifier.  
  - domain classifier는 domain 구분을 잘하도록, backbone은 domain 구분을 못하도록 adversarial하게 학습함.
  - domain classiifer를 gradient reversal layer를 둠. doamin classifier에서 domain classification을 잘하도록 하는 gradient의 부호를 바꾼 후에 backbone에 흘려보냄.

**Contributions**
- domain adaptation에 강한 DANN을 제시함.
- DANN가 domain adaptation에 강함을 수학적으로 증명함.
- 기존 domain adaptation sota(?)모델보다 좋은 performance를 보임.

**Experiments**
- Toy Problem
  - 실험: 
    - 1. 300개의 source domain sample, 300개의 target domain sample을 인위로 만듦(toy data). 각 domain sample은 150/150으로 pos/neg로 분류
    - 2. DANN, NN으로 길이가 15인 representation을 뽑음.
    - 3. DANN, NN을 label classification, domain classification task에서 평가해보고, PCA등으로 2차원에 시각화해서 qualitative analysis를 함.
  - 결과: 
    - label classification: DANN, NN 둘다 source domain에서는 잘하지만, target domain에서는 DANN만 잘함.
    - domain classificaiton: DANN은 전혀 못하고, NN은 꽤 함.
    - representation PCA: 
      - 이 실험의 목적은, domain adaptation regularizer의 역할을 보고싶은 것임. DANN의 target representation은 source rep.에 적절히 퍼져있는 것으로 보아, 
  source에 적절히 동화되어있음. NN의 target rep.는 source rep.를 포함하고 있지 않은 채로 어딘가에 clustering되어있어서 target이 source에 적절히 동화되지 못함.
      - 또한 DANN target rep.는 pos/neg를 잘 classify하고있는데, NN target rep.는 그렇지않음.
    - Hidden Neurons: 15 dimension의 hidden neuron 하나하나를 linear regressor로 보는 것임. NN은 source/target을 구분짓는데, DANN은 source/target 구분을 전혀못함.
- 실제 dataset (Amazon reviews dataset)에 대해 적용해봄. sota(?)가 나옴.
- Proxy Distance
  - Proxy A-distance(PAD) 용어 정의: 
    - 어떤 한 backbone으로 뽑은 source rep.와 target rep.의 simiarity
  - 실험내용: raw data, NN rep., DANN rep., mSDA rep.의 PAD를 살펴봄. 각 rep.마다 PAD가 하나씩 나오는것.
    - dataset은 Amazon reviews dataset을 이용.
  - 결과:
    - DANN이 raw data, NN보다 좋음.
    - mSDA rep.를 뽑고, 그걸로 DANN을 하면 PAD가 더 낮음. 
    - 의문점은, mSDA는 domain adaptation은 잘되지만 PAD는 높다고함.
- 나머지는 생략.. 

**총평**
- 수학적 근거를 제시해줘서 좋았다.

## Study

**읽는데 걸린 시간**
- 읽는데 3:00 정리하는데 1:15분
- pages: p19 / p31 (without references&appendix) 


**비판적 사고**
- mSDA는 domain adaptation은 잘되지만 PAD는 높다고함. 이 논문은, 자기들 방법대로 하면 source rep.와 target rep.가 구분이 안되고, 이걸 PAD가 낮아짐을 통해 보였는데, 
mSDA는 이것에 정면으로 반하는 사례임. 이게 왜 그런지에 대한 컨셉 레벨에서라도 설명이 있었으면 좋겠다.
- Eq (1)에 typo가 있다. source sample은 1이 되게, target sample은 0이 되어야한다고 해야하는것 아닌가?

**이해못한 점**
- 거의 다 이해함.
