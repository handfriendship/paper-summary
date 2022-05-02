# Deep Domain-Adversarial Image Generation for Domain Generalisation
- Author: Kaiyang Zhou, Yongxin Yang, Timothy Hospedales, and Tao Xiang
- AAAI 2020

## 세가지가 꼭 들어가있어야 함.
- 어떤 것을 풀려고 했는지(Summary)
- 기존에는 무슨 문제가 있는지, 기존의 방법에 비해 우리가 왜 더 좋은지(Related Works)
- 어떻게 풀었는지(Main)

**Summary**
- 어떤 것을 풀려고 했나요?
  - data-augmentation기반의 DG를 위해서 perturbation을 생성할 때 아무 sparsely sampled된 perturbation이 아니라, semantic한 방향으로 영역을 넓히는 perturbation을 생성할 순 없을가?
- DG를 할 대 CNN을 이용해서 perturbation을 생성해보자.
- CNN모델로 생성한 perturbation을 통해 image를 transformation시켜서 DG를 한다.
- DoTNet은 adversarial training으로 domain을 구별하지 못하면서 class별로는 잘 clustering되는 perturbation을 생성하는 쪽으로 학습시킨다.
- 기존 adversarial training(e.g. CrossGrad)보다 많은 점(perceptive, dealing with geometric transformation, DG성능, re-ID성능)에서 좋다는 것을 보였다.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- 1)기존에는 무슨 문제가 있나요?
  - Data Augmentation을 위해 생성한 perturbation이 탐색한 unseen space이 꼭 semantic하게 의미있지 않을 수 있다.
- 2)기존의 방법에 비해 왜 더 좋은가요?
  - perturbation만 보고도 label은 구별할 수 있으면서 domain은 구별할 수 없기 때문에, 이 perturbation을 가했을 때 manifold에서 더 semantic한 spacㄷ가 cover됨을 알 수 있다.
  - adversarial perturbation은 perceptive하지 않은데, 우리 방법은 perceptive하다.(feature level로 적용되는게 아니라 image-level perturbation임)
  - CNN기반의 network를 쓰기 때문에 다른 transformation도 cover할 수 있다.
    - 보통 DG는 다른 style domain에서도 잘 동작하게끔 만드는 것인데, 다른 transformation domain에서도 잘 동작하게 만들 수 있다.(같은 CNN모델이라서 그냥 결합하면 되서.)
- DG방법 분류, 설명
  - Domain Alignment
    - Domain Adaptation에서 많이 쓰이던 방법이다.
    - representation space에서 domain간의 거리를 가깝게 하는 방식이 있다.
    - domain-invariant feature를 생성하는 방식도 있다.(DANN)
    - source domains에 overfit될 수 있다는 위험이 있다.
  - Meta Learning
    - 단점: black-box, source domains에 overfit될 수 있다는 위험이 있다.
    
**Main Idea(어떻게 풀었는지)**
- label은 잘 맞추게, domain은 구별못하게 하는 perturbation을 만들면 semantic manifold를 잘 cover할 수 있을 것이다.
- gradient을 기반으로 image optimization을 수행하면서 perturbation을 만드는 것이 아니라, CNN으로 직접 perturbation을 생성하자.
- 학습과정
  - 1) DoTNet으로 perturbation을 구한다.
  - 2) perturbated image로 domain classifier, label classifier의 결과를 구한다.
  - 3) DoTNet을 update한다.
  - 4) label classifier는 warm up 동안에는 original image로, 그 이후에는 original, perturbated 둘다로 update한다.
  - 5) domain classifier는 original image로만 update한다.
- Architecture
  - 1) DoTNet
    - n x 2-conv residual blocks -> global average pooling -> 원본 크기로 expand -> residual block의 결과랑 concatenate -> 1x1 conv kernel
    - transform시킨 것의 label을 맞춤 + transform이 시킨 것이 어떤 domain에 속하는지를 맞추는 것의 반대방향의 gradient
      - Domain Transformation Network를 학습시킬 때 label classifier와 domain classifier의 weight 조절을 안하는 이유는, 
adversarial training이라서 label classifier와 domain classifier의 균형을 맞춰야하기때문에 그런듯?
    - dealing with geometric transformation을 위해 SYN을 추가할 경우 STN -> DoTNet 순서로 진행함.
  - 2) label classifier: 
    - original image의 label을 맞춤 + perturbated image의 label을 맞춤.
    - 둘의 loss를 alpha로 조절하고, warm-up step을 줘서 조절함.
  - 3) domain classifier: original image의 domain만 맞춤.

**Contributions**
- adversarial approach가 아니라 CNN model로 perturbation을 생성하는 방법을 제시.

**Experiments**
- 1) Office-Home, PACS, Digits-DG에서 성능 평가
- 2) re-ID에서 성능 평가
  - source domain과 target domain의 label space가 같지 않다는 점에서 heterogeneous DG라고 부름.
- 3) Dealing with Geometric transformation
  - rotated domain도 잘 처리할 수 있는지 봄.
- 4) visualization
  - perturbated image는 original image와 겹치지 않음.
  - semantic manifold를 잘 덮을 수 있음.


**총평**
- perturbation을 CNN으로 생성하겠다는 idea가 여기서 처음 나온 것인가? 참신한 것 같다.

## Study

**읽는데 걸린 시간**
- 읽는데 3:50시간 정리하는데 1:20분
- pages: p7 (without references&appendix) 

**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- Q1. Figure 6. visualization을 보면 class끼리 잘 구별되어있어보이진 않는다. class-wise로 하면 어떨가?
  - DANN을 class-wise버전으로 바꾼 논문이 있다고 들었는데, 그걸 보면 어떻게 class-wise로 바꿀 수 있는지 알 수 있을듯.

**이해안되는 부분**
- Q1. versatility라는 단어가 계속 나오네. 모델의 main contribution이 perturbation을 가해서 versatility를 높이는 것인가?
- Q2. domain의 정의가 뭔가? rotated dataset이면 다른 domain인가?
