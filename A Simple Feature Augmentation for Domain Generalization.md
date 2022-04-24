# A Simple Feature Augmentation for Domain Generalization
- Author: Pan Li1, Da Li2,3, Wei Li, Shaogang Gong, Yanwei Fu4, Timothy M. Hospedales
- ICCV 2021

**Summary**
- 아주 단순하게 Gaussian Noise를 latent feature에 잘 더해주는 augmentation방식이 DG에 잘 동작하더라는 것을 보이는 논문.
- Gaussian Noise를 두 가지 다른 방식으로 더해줌. 1)그냥 더하기 2)data간의 covariance matrix를 만들어서 도메인간 변동성과 intra-class category를 잘 분류할 수 있는 방향으로 noise를 주는 방식.
- 1)의 intuition은 general-purpose regularizer처럼, 현재 가지고있는 data domain에서만 잘하도록 하는 것이 아니라, noise를 약간 준 변형된 data에서도 잘되게끔 하는것.
- 2)의 intuition은, - 우리는 현재 가지고 있는 data domain을 확장하고 싶다.(unseen data에도 잘 대응하기 위해서는 현재 training data만 딱 분류하기 보다는, 더 많은 영역도 잘 분류할 수 있어야 하기 때문.)
data domain을 확장할 때 도메인간 변동성을 잘 모델링하는 방향으로 확장하면서, 동시에 다른 class category영역을 침범하지 않도록 하는 것.
- 2)를 구현하기 위해서는 도메인간 변동성, intra-class consistency 둘을 고려하는 covariance matrix를 만드는 것이 핵심.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- 논문에서 차용한 idea
  - IN(Instance Normalization)
  - AdaIN(Adaptive Instance Normalization)
- DG분야의 related works
  - source domains간에 feature align을 하려는 시도
  - adversarial loss를 이용할 때 domain classifier를 둬서 auxiliary loss로 이용함. domain augnostic feature를 배우기 위함.
  - domain-specific parametrization을 하려는 시도
  - meta-learning을 이용한 시도. pseudo-train, pseudo-test를 이용.
  - Data augmentatin을 이용한 시도.
    - domain classifier로부터 얻어진 adversarial gradient를 이용. soruce data와 멀리 떨어진 domain을 만들려는 시도인듯.
    - pseudo/intermediate domain을 만들려는 시도
      - DLOW(for DA): image translation model을 이용해서 intermediate domain을 만듦.
      - L2A-OT: optimal transport를 maximizing하는 방식으로 psuedo-domain을 만듦.
      - mixstyle과 차이점:
        - 이 두가지는 target domain을 explicit하게 만드는데 반해 MixStyle은 implicit하게 만듦.
        - mixstyle은 feature-level augmentation임 vs 위 두가지는 image-level augmentation.
    - general-purpose regularization technique이 DG분야에서는 잘 안되는 이유:
      - regularization때문에 discriminative pattern(예측 성능에 결정적인 영향을 미치는 pattern)을 찾는데는 도움이 될 수 있으나, DG에 도움이 된다고 보장할 순 없음.
      - 반면에 mixstyle은 새로운 style을 synthesis하는 것이므로 DG에 도움이 됨.
- RL분야에서 generalization을 연구한 related works
  - 관심 없으니 생략..

**Main Idea**
- 1)방식은 latent embedding tensor와 같은 size의 gaussian noise를 생성해서 곱하고 더하면 된다.(alpha=multiplicative noise, beta=additive noise)
- 2)방식은 도메인간 변동성, intra-class consistency 둘을 고려하는 covariance matrix를 만드는 것이 핵심이다.
  - covariance matrix는 KxK matrix이고, K는 mini-batch의 개수이다.(gradient accumulation을 생각하면 됨. batch_size=42, K=8이면 42크기의 batch가 8개 모인것)
 
**Contributions**
- mixstyle기법을 제안했다. 완전 새로운 idea는 아니고, style transfer분야에서 적용하던 기법을 들고와서 DG분야에 접목한 것이다.
- 간단하고 구현하기 쉽고 plug-and-play가 가능하다.

**Experiments**
- 여러 DG benchmarks에서 실험.
  - related works와 비교하는 실험, DG에 사용하는 다른 data augmentation기법을 적용했을 때와 비교하는 실험으로 나눌 수 있음.
  - classification task를 잘 푸는지 보기 위함.
  - PACS(같은 사진의 sketch, cartoon, art-painting, picture버전), Digits-DG(서로 다른 style의 숫자), Office-Home(office, home에서 볼 수 있는 물체)에서 실험함.
  - related works와 비교했을 때는 PACS에서는 SOTA / Digits-DG, Office-Home에서는 previous sota와 약간 안좋은 성능.
- Person re-ID와 비교
  - instance retrieval task에도 잘 적용될 수 있는지 보기위함.
  - person re-ID에서 실험. 서로 다른 view에서 촬영된 사람이 같은 사람인지 판단하는 task. domain generalization의 일종이라고 볼 수 있음. 
  - 다른 data augmentation기법보다 더 좋은 성능을 보임.
- deep RL task에도 적용해봄.
  - unseen environment에서도 agent가 잘 동작하는지 보기 위함.
  - 잘 동작했음. RL은 관심없으니 자세한 설명은 생략.
- CNN model의 어느 layer에 plug해야 좋은 성능을 보이는지 실험.
  - 앞쪽 layer의 결과가 style정보를 가지고 있기에 앞쪽에 적용하는 것이 좋음.
  - 뒷쪽 layer의 결과(activation maps)는 content정보를 가지고 있기에 적용하면 별로 안좋음.(왜냐면 마지막 layer의 경우, 바로 pooling이 적용되서 classwise logit이 나오기때문.
  - 앞쪽 layer와 뒷쪽 layer의 결과는 visualization을 통해 style정보를 가지고 있는지 content정보를 가지고 있는지 파악했음.
- 기타 여러 ablation성격의 실험.
  - mixing vs replacing
  - random vs fixed shuffle
  - sensitivity of hyperparameter
  - 같은 domain내에서 augmentation을 적용해도 효과가 있음을 보이는 실험.


**총평**
- augmentation-based DG를 하기 위해서 noise를 준다는 생각은 누구나 할 수 있지만, noise를 어떻게 줘야할까를 고민해봤을 때 그것에 대한 대답으로 
도메인간 변동성, intra-class consistency 둘을 고려해야 한다는 생각은 어떻게 할 수 있을까? 이 분야 논문을 많이 읽다 보면 이것은 그냥 consensus인건지, 아니면 저자만의 idea인건지는 아직 잘 모르겠다.

## Study

**읽는데 걸린 시간**
- 읽는데 4:10시간 정리하는데 0:30분
- pages: p8 (without references&appendix) 


**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- 

**질문**
- Q1. SFA-A는 그렇다 쳐도 SFA-S는 왜 잘되나? 이것이 잘되는 것에 대한 intuition이 뭔가? 
- Q1-1. SFA-A가 잘되면 general-purpose regularization technique도 DG에 효과가 있을 수 있다는 것 아닌가?
- Q2. MixStyle은 왜 비교안하냐? 똑같은 data augmentation 종류인데?
- Q3. 한 epoch이 끝나도 batch의 구성은 바뀌지 않아야 하는거지? batch의 순서는 바뀔 수 있어도 ..
- Q4. 처음 시작할때는 K개의 mini-batches만 본 상태일텐데, 이걸 가지고 whole covariance를 만들 수 있는건지? -> covariance수식을 찾아볼 필요가 있을듯.

