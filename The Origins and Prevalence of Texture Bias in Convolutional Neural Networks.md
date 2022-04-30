# The Origins and Prevalence of Texture Bias in Convolutional Neural Networks
- Author: Oren Nuriel, Sagie Benaim, Lior Wolf
- CVPR 2021

## 세가지가 꼭 들어가있어야 함.
- 어떤 것을 풀려고 했는지(Summary)
- 기존에는 무슨 문제가 있는지, 기존의 방법에 비해 우리가 왜 더 좋은지(Related Works)
- 어떻게 풀었는지(Main)

**Summary**
- 어떤 것을 풀려고 했나요?
  - Geirhos가 CNN이 texture-bias된다는 현상을 발견했으니, 이 현상이 왜 일어나고, 어떤 요소가 가장 많이 영향을 미치는지 밝히고 싶다.
- 크게 CNN의 inductive bias자체가 문제인지, dataset이 문제인지 알아본 결과 dataset이 문제인 것 같다.
- 이후 실험들은 dataset을 어떻게 활용하고 있길래 문제가 되는지를 밝히는데 중점을 뒀다.
- dataset을 random crop시키는 방식으로 이용하니깐 문제였다.
- ImageNet을 supervised learning으로 학습시키는게 문제였다.
- 문제를 어떻게 해결하는지도 제시했다.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- 1)기존에는 무슨 문제가 있나요?
  - 기존에는 CNN이 texture-bias라고 설명하는 연구가 지배적이었다. 
  - 근데 robust한 CNN모델을 만들려면 꼭 texture-debias시켜야 할까? texture recognition성능도 같이 높일 수는 없을까?
- 2)기존의 방법에 비해 왜 더 좋은가요?
  - Bias toward texture
    - 기존 방법론은 성능 감소가 발생하고, 특히 texture recognition성능이 안좋아지는데 우리의 방법은 robust + texture recognition성능안감소 이다.
    - 기존 방법론과 conclusion도 다르다.(우리 주장은, texture recognition과 성능 향상이 함께할 수 있다는 것이다.)
    - architecture or new objective를 제시하는 방법론은 간접적으로 texture-bias를 줄이는데, 반면 우리가 global statistics bias를 줄이는 방법은 직접적(mean, var를 바로 건드림)이다.
  - Normalization을 하는 연구들
    - 기존 normalization technique들은 효과적인 training을 위한 것인데, 우리 방법은 network가 shape, fine detail에 집중할 수 있도록 하는 방법론이다.(global statistics에 bias되지 않기 때문에 다른 cue를 보게됨.
  - AdaIN을 사용하는 연구들
    - 기존 AdaIn계열 연구들은 generative modeling, style transfer를 위한 것인데, 우리 방법은 image recognition을 더 잘하고 더 robust하게 하기 위함이다.

**Main Idea(어떻게 풀었는지)**
- pAdaIN
  - 각 sample에 대한 pair를 짝지어 준다.
  - in-batch내의 sample을 뽑아 pair로 쓰고, 기준 sample의 mean, var를 pair의 mean, var로 교체한다.
    - in-batch에서 뽑는 이유는, 같은 distribution에서 뽑기 위함이다.(random한 mean, var로 교체하지 않는 이유.)
  - channel-wise로 적용된다.
  - CNN model의 모든 layer마다 적용된다.

**Contributions**
- texture-bias와 다른 해석을 내놓음.
- CNN model의 robustness와 texture recognition performance를 같이 가져갈 수 있다.

**Experiments**
- 1) Batch Normalization을 쓰는게 pAdaIN에 영향을 미치지는 않나?
- 2) Image Classification 성능
- 3) p를 다르게 줬을 때 성능이 어떻게 변하는지
  - p가 너무 크면 성능을 해칠 수 있음.
- 4) Texture Recognition 성능
  - feature representation이 얼마나 shape, texture에 대한 정보를 담고 있는지 평가하는 식으로 해당 모델에 대한 texture recognition을 평가한다.
  - texture surface dataset에서 평가를 해본다.
  - model은 freeze를 시키고 feature representation을 입력으로 하는 linear classifier를 texture surface dataset의 10%만 써서 학습시키고 나머지 90%로 평가한다.
- 5) DA (on segmentation)
  - 실험 방법(FADA 방법을 따름.):
    - 5-1. source dataset에서 학습시킴.
    - 5-2. target dataset으로 unsupervised + source dataset으로 supervised시킴.
      - class-wise로 domain invariant feature를 생성(DANN처럼)
      - source dataset은 cross-entropy로
    - 5-3. 5-2.모델로 target dataset를 pseudo-labeling한 다음 supervised learning으로 품.
    - 5-2. 과정 중에서 source dataset을 target dataset으로 pAdaIN augment시킴.
- 6) DG
- 7) Robustness towrad corruptions
  - noise, lur, pixelated, JPEG corruption은 성능 향상이 조금밖에 없음.(global statistics는 보존되고 fine-detail에 대해 corrpution이 가해진 경우인데, 우리는 global statistics은 보지 않고 fine-detail, shape같은걸 보고 판단하기 때문에)
  - weather, contrast corruption에 대해서는 성능 향상이 많음.
- 8) Ablation study
  - x에 대한 gradient를 backprop시킬 때 pi(x)에 대한 weight도 update하면 어떨까?
    - x에 대한 gradient가 pi(x)도 update시키기 때문에 부적절하다.(AdaIN(x)를 할 때 pi(x)에 대한 mean, var에 pi(x)가 포함되어 있어서 gradient가 여기로도 흘러들어감.)

**총평**
- domain generalization

## Study

**읽는데 걸린 시간**
- 읽는데 2:55시간 정리하는데 0:30분
- pages: p8 (without references&appendix) 

**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- ...

