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
- ㅇㅇ

**Main Idea**
- 1)방식은 latent embedding tensor와 같은 size의 gaussian noise를 생성해서 곱하고 더하면 된다.(alpha=multiplicative noise, beta=additive noise)
- 2)방식은 도메인간 변동성, intra-class consistency 둘을 고려하는 covariance matrix를 만드는 것이 핵심이다.
  - Covariance Matrix를 만드는 법(Digit-DG의 실험셋팅을 예로 들어봄):
    - 1) z^i = (N, K, W, H)에서 WxH에 속하는 모든 pixel값들은 평균내서 하나의 값으로 바꾼다.
      - N: source domain의 개수. Digit-DG의 경우 N=3. 
      - K: #mini-batch. batch_size=42짜리인 batch들이 K개만큼 모이면 covariance matrix를 한 번 계산한다. Digit-DG의 실험셋팅의 경우 K=8.
    - 2) 1)의 결과인 (3, 8) matrix의 각 원소는 42개의 sample을 가지고있다. 42개 중 우리가 구하려는 class의 sample들만을 추려낸다. 
      - (이것의 평균을 구하거나 하는 식으로 1차원으로 만드는듯)
      - channel수도 생략되었는데, 적당히 aggregation하는듯.
    - 3) 2)의 결과인 (3, 8) matrix의 covariance matrix를 구한다. 
      - (3, 8).T x (3, 8) = (8, 8)크기의 covariance matrix가 만들어짐.
      - 이것의 직관적 의미는, 3을 instance의 개수, 8을 각 feature의 개수라고 봤을 때 각 feature들간의 상관관계를 구했다고 생각하면 됨.
      - 매 K개의 mini-batch가 모일때마다 class-wise covariance matrix를 update함.
    - 4) 3)의 결과를 covariance로 갖는 multi-variate gaussian distribution에서 k-dim noise를 N개(domain의 개수)만큼 sampling한다.
  - Covariance Matrix의 각 항목은 모든 domain의 feature를 다 고려한 것이다.(K=8일때 8x8이면 feature가 8개 있는 것으로 생각할 수 있는데, 각 domain에 존재하는 해당 feature들을 다 aggregation한게 covariance matrix의 각 항목임.)
  - Q. intra class consistency? 
    - A. 각 class내에서만 도메인간 변동성을 고려하는 방향으로 covariance matrix를 만들었으니, 적어도 현재 class들의 분포보다 더 넘어가지는 않겠군. 
    - 현재 모든 domain에 존재하는 해당 class들의 data가 다 섞여있는 상태.
    - 현재 각 class에 속하는 sample들이 class별로 잘 분류되어있다면, 각 class의 data distribution을 따르는 방식으로 noise를 sampling하더라도 다른 class를 침범하는 sample을 만들게끔 하는 noise를 생성하지는 않을듯.
    - 만약 현재 sample들이 class별로 잘 분류되어있지 않다면, 성능을 더 해칠수도 있을 것 같은데 ..?
  - Q. 도메인간 변동성을 잘 모델링하는 방향으로 데이터 영역을 spanning하는가?
    - A. 
 
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
- noise를 contrastive learning을 통해서 더 intra-class이도록 하는 분포로부터 sampling할 순 없을까?
  - 현재는 하나의 batch안에 같은 domain sample만 들어있도록 주로 구성을 하는 것 같다.(맞나?)
  - 하나의 batch안에 다른 domain sample들도 있도록 구성을 한다음, in-batch contrastive learning을 적용해서 다른 domain에 속하더라도 같은 class이면 같도록 하게 한다.
  - 같은 domain인데 다른 class를 가지는 sample들은 hard-negative로 생각해서 가중치를 더 준다.(이거는 fashion에 사용되었던 논문에서 어떻게 처리했는지 보면 좋을듯)
  - (+multiple anchor를 두는 식으로도 모델링 가능한가?)
- gaussian mixture model?

**질문**
- Q1. SFA-A는 그렇다 쳐도 SFA-S는 왜 잘되나? 이것이 잘되는 것에 대한 intuition이 뭔가? 
- Q1-1. SFA-A가 잘되면 general-purpose regularization technique도 DG에 효과가 있을 수 있다는 것 아닌가?
- Q2. MixStyle은 왜 비교안하냐? 똑같은 data augmentation 종류인데?
- Q3. KxK covariance matrix에서 k-dim noise를 N개(domain의 개수)만큼 sampling하는건가? 아니면 같은 noise가 각 도메인에 더해지는건가?
