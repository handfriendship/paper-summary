# The Origins and Prevalence of Texture Bias in Convolutional Neural Networks
- Author: Oren Nuriel, Sagie Benaim, Lior Wolf
- NeurIPS 2020

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
  - Adversarial training을 통해서 모델을 OOD data에 대해 더 robust하게 만드는 학습 방법
    - OOD에는 좋을지 몰라도 다른 형태의 perturbation에도 강인하다는 보장이 없다.(ImageNet-C 같은걸 생각하면 될듯?)
  - Data Augmentation and Robustness
    - augmentation기법을 적용하면 해당 augmentation에 해당하는 robustness를 얻을 순 있어도 다른 형태의 corrpution에는 잘 대응하지 못할 수 있다.
    - 자연적인 distribution shift에는 잘 대응하지 못할 수도 있다.(우리는 인위적인 perturbation을 쓴 거니깐.)
  - sensitivity of CNNs to non-shape features
    - 최근에 많은 연구들이 CNN이 texture에 bias되어있다고 주장한다. 우리는 이 현상이 나타난 원인을 분석하고자 한다.

**Main Idea(어떻게 풀었는지)**
- 1) 처음에 ImageNet 데이터셋이 문제인지 CNN의 inductive bias자체가 문제인지 테스트해봄.
- 2) ImageNet을 어떻게 사용하길래 문제인지 봄.
- 3) ImageNet을 사용하면서 CNN이 texture bias되지 않는 방법을 제시함.
  - 3-1. random crop -> center crop으로 바꿈.
  - 3-2. self-supervised loss를 추가하는 것이 도움이 됨.

**Contributions**
- CNN이 texture-bias되지 않을 수 있는 방법을 제시함.
- 왜 이런 현상이 발생하는지에 대한 답을 제시함.
  - CNN inductive bias때문이 아니라 ImageNet때문이다.
  - random crop
  - 단순 supervised learning이 ImageNet의 일부 class를 구별하는 것과는 잘 맞지 않다.
  - FC layers의 dimension이 지속적으로 감소하기 때문이이다.

**Experiments**
- 1) CNNs can learn shape as easily as texture
  - 목적: 애초에 CNN이 shape를 배울 수 없는 모델인지, CNN은 shape을 배울 수 있는데 dataset이 잘못된건지 알아보기 위함.
  - 가설: 만약 CNN이 shape을 배우지 못한다면 shape을 label로 줘서 supervised learning을 시켜도 배우지 못할 것이다.
  - 실험방법: AlexNet과 ResNet을 대상으로 GST, Navon, ImageNet-C를 shape label과 texture label로 학습시켜본다.
  - 결과: CNN이 shape을 잘 배울 수 있음. ImageNet의 문제임.
- 2) The role of data augmentation in texture bias
  - 목적: 자주 사용되는 data augmentation들 중 어떤 것이 texture-bias를 일으키는지 혹은 줄이는지 알아본다.
  - 가설&실험방법: random-crop을 적용했을 때의 변화를 본다 / naturalistic augmentation을 적용했을 때의 변화를 본다.
  - 결과:
    - random-crop은 안좋음.
    - naturalistic augmentation은 좋음.
    - naturalistic augmentation은 additive(stack)이 가능함.
    - shape-bias를 높여줄수록 OOD robustness가 올라감.
    - shape-bias를 높여줄수록 imagenet top1 accuracy는 낮아짐.(trade-off)
- 3) Effect of training objective
  - 목적: 현재 training objective가 texture-bias를 일으키고 있는지 진단 & 이를 극복할 수 있는 self-supervised learning
  - 가설: shape에 더 집중해야 잘 풀 수 있도록 하는 self-supervised learning을 쓰면 더 좋아질 것이다.
  - 실험방법:
    - Rotation
    - Exampler
    - BigBiGAN
    - SimCLR
  - 결과:
    - 어떤 모델을 쓰는지와는 상관없더라.
    - 기존 supervised objective에다가 SimCLR objective를 합쳤을 때 shape-bias가 더 높아지는걸로 봐서, 확실히 적절한 self-supervised learning을 쓰는게 도움이 된다.
      - 만약 가장 좋은 성능인 SimCLR를 Supervised learning에다가 합쳤을 때 별 성능향상이 없었다면 현지 supervised loss도 충분히 shape-bias를 잘보고 있는 것
- 4) Effect of architecture
  - 높은 top1 accuracy => high-shape인건 맞지만, high shape => high top1 acc인건 아니다.
  - convolution을 attention으로 바꿔봤을때도 texture-bias가 생기는걸로 봐서 architecture문제가 아니라 dataset문제이다.
- 5) Degree of representation of shape and texture in ImageNet models
  - FC layer에서 dimension이 좁아질수록 shape-bias가 더 줄어드는걸로 봐서 이게 문제이다. encoder(backbone)은 문제가 아니다.

**총평**
- domain generalization

## Study

**읽는데 걸린 시간**
- 읽는데 2:55시간 정리하는데 1:20분
- pages: p8 (without references&appendix) 

**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- ...

