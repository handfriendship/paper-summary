# DOMAIN GENERALIZATION WITH MIXSTYLE
- Author: Kaiyang Zhou, Yongxin Yang, Yu Qiao, Tao Xiang
- ICLR 2021

**Summary**
- 서로 다른 domain의 image의 style vector를 추출하고, 그것을 convex combination하여 새로운 임의의 style을 가지는 image를 만드는 식으로 data augmentation을 하여 학습에 활용하면 domain generalizing이 잘 된다는 아이디어의 논문이다.
- 저자들은 먼저, 서로 다른 domain은 다른 style을 가지는 image group들이라는 것을 t-SNE를 통해 보인다.
- idea: 서로 다른 domain의 image의 평균과 분산을 구한 뒤, 그것을 convex combination하여 새로운 평균, 분산을 구한다. 그리고 그것이 input image의 평균과 분산이 되도록 바꾼다.
- 아이디어는 AdaIN에서 style transfer는 한 image의 평균과 분산을 style을 입힐 image의 평균과 분산이 되도록 수행한다는 것에 착안했다.
- 크게 related works와 비교하는 실험, DG에 사용하는 다른 data augmentation기법을 적용했을 때와 비교하는 실험으로 나눌 수 있는데, previous sota와 비교했을 때 비슷하거나 약간 더 좋은 성능이 나왔고, previous sota보다 훨씬 간단하고, 구현하기 쉽고, plug-and-play가 가능하다는 장점이 있었따.
- 다른 augmentation기법과 비교했을 때 우리 방법이 가장 좋은 성능을 냈다.

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
- RL분야에서 generalization을 연구한 related works
  - 관심 없으니 생략..

**Main Idea**
- 1) Conv layer의 output activation fiber에 IN을 써서 각 activation map마다 평균, 분산을 구함.
- 2) 서로 다른 domain의 image를 pair로 만들어 각 평균, 분산을 convex combination함.
  - 이때 convex combination에 사용되는 weight lambda는 activation_map-wise가 아니라 instance-wise임. 
  - domain label(image가 어떤 domain에 속하는지)를 아는 경우와 모르는 경우로 나누어 방법을 제시함.
  - domain label을 아는 경우는, 각 domain에서 하나씩 sampling해서 pair를 만듦.
  - domain label을 모르는 경우는 같은 domain에 속한 image끼리 pair가 될 수도 있는데, 이 경우에도 효과가 있다고 함.
- 3) 2)에서 만든 새로운 평균, 분산을 input image에 activation_map-wise로 적용한 새로운 image를 만듦.
- 세부사항
  - test time에서는 적용하지 않음.
  - training time에서도 이 과정은 50%확률로 적용됨.
  - lambda는 beta-distribution에서 sampling됨.
 
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
- 실험하기 쉽다는 말이 진짜인 것 같다는 생각이 들었다. 진짜로 domain=style인가? 하는 생각이 들었다.

## Study

**읽는데 걸린 시간**
- 읽는데 1:50시간 정리하는데 0:30분
- pages: p9 (without references&appendix) 


**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- Song et al.의 Few-Shot font generation을 이용할 순 없을까? 이것의 핵심 idea는 각 나라별 문자 중에 radical을 가지는 문자들은 component로 factorize한 후, 각각의 style을 다 더해서 최종 character를 생성하겠다는 거다. few-shot font generation이기 때문에 reference set이 모든 radical들을 포함하고 있지 않을 수 있는데, 이때는 reference set의 radical들을 component factor + style factor로 factorize한 후, style factor를 source glyph radical의 component factor와 조합하여 unseen radical을 생성한다. 지금 당장 떠오르는 방법으로는, PACS에서 하나를 source dataset, 하나를 reference dataset으로 잡고 여러 다른 style을 가진 synthesized data를 생성하여 추가로 학습에 이용한다?는 것이다. 이런 방법으로 randomness를 갖는, 즉 새로운 많은 style들을 가진 image이면서 각각이 전부 다 조금씩 다른 style을 가지는 image들을 얻을 수 있으면 좋겠다. 추가적으로 이 분야의 논문을 더 읽어보면서 다른 논문들은 어떻게했는지 알 필요가 있을 것 같다.

**질문**
- Domain이 다르다는 것은 image의 content는 같고 style이 다르다는 말이라고 논문에서 주장하는데, 이러한 사실은 DG분야에서 consensus가 있는건가? 아니면 단순히 논문에서 주장하는것인가?
- alpha를 왜 꼭 beta distribution에서 뽑아야하는가? [0, 1]범위의 random number를 뽑으면 안되나?
- Q3. 논문에서는 왜 image-level보다 feature-level이 더 좋다고 했나?
- Q4. 논문에서는 general-purpose regularization technique이 DG에 적용이 잘 안되는 이유가 뭐라고 했나?
