# SelfReg: Self-supervised Contrastive Regularization for Domain Generalization
- Author: Daehee Kim, Youngjun Yoo, Seunghyun Park, Jinkyu Kim, and Jaekoo Lee
- ICCV 2021
- Learning Strategy -> Self-supervised learning 에 속함.

**Summary**
- 논문의 main은 contrastive learning을 이용한 regularizer를 걸어줘서 DG성능을 올렸다는 논문이고, 추가적인 테크닉을 많이 썼다.
- 논문의 motivation은, 기존 CL은 negative sample을 어떤 것을 사용하느냐에 따라 성능이 많이 좌우되는데 positive sample만을 쓰되 잘 쓰면 이러한 문제를 없앨 수 있다고 했다.
- contrastive learning을 두 가지 유형으로 사용한다. ind, hdl. 그리고 이 두가지를 각각 feature layer, logit layer에 사용한다.
- positive sample을 augmentation하는 방법으로는 CDPL, mixup을 사용했다.
- 추가적인 테크닉으로는 Stochastic Weights Averaging, curriculum learning을 사용했다.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- Domain invariant representation learning
  - DANN
  - DANN은 class-agnostic인데, class별로 distribution이 가깝도록 하는 것.(자칫 discriminative ability를 해칠 수도 있으므로.)
  - shared feature space상에서 distribution이 가깝도록 하는 것.(distribution사이의 거리를 측정하는 metric에 따라 여러 방법론이 나옴.)
- Contrastive Learning
  - CCSA (ICCV 2017) 인용수 470
  - Domain Generalization via Model-Agnostic Learning of Semantic Features (NIPS 2019) 인용수 260
  - 위 논문들은 negative sample을 멀어지도록 했다.
- Inter-domain Mixup
  - 서로 다른 domain의 sample들의 linearly interpolated의 class도 잘 맞출 수 있도록 하는 접근법.

**Main Idea**
- 1) Contrastive Leanring
  - intuition(for DG): same feature space에 explicitly aligning시키기 위해서.(다른 domain들도 이 feature space에서 가깝게 만들고자함.)
  - 기존의 방법론과 다른점: 기존 CL은 negative sample을 어떤 것을 사용하느냐에 따라 성능이 많이 좌우되니깐 positive sample만을 써서 CL을 하자는 것이다. 
하지만 positive sample만 쓸 경우 representation collapse문제가 일어날 수 있는데, 이를 위해 positive sample에 적절한 perturbation을 줘서 변형을 하는 식으로 풀어가고자 한다.
  - 1-1) Individualized in-batch dissimilarity Loss 
    - 한 batch안에는 여러개의 domain에서 추출된 sample들 중 같은 class의 sample들만 있음.
    - batch의 모든 원소에 대해 각각 pair를 만들고, pair원소는 CDPL이라는 perturbation을 준 뒤, 두 개의 feature space상에서의 거리가 가까워지도록 하는 것.
    - perturbation은 2 MLP layers(hidden_size=1024) + BN + ReLU
  - 1-2) Heterogeneous In-batch Dissimilarity Loss
    - 한 batch안에는 여러개의 domain에서 추출된 sample들 중 같은 class의 sample들만 있음.
    - batch의 모든 원소에 대해 pair를 만듬.
    - z_i를 perturbation한 것과 pair를 perturbation한 것의 convex combination를 구하고, 두 개의 feature space상에서의 거리가 가까워지도록 하는 것.
  - 1-1, 1-2는 feature layer에도 적용할 수 있고 logit layer에도 적용할 수 있다. 두군데에 다 적용함.
- SWA
  - intuition(for this method): SWA가 flat minima를 찾게끔 유도하고, DG를 잘하려면 flat minima를 찾는 것이 중요하기 때문. (CL과는 orthogonal한 기법임.)
  - training시 일정 간격을 두고 weight를 capture한 후 averaging하는 것. 
  - flat minima에 도달하는 이유는, 여러 weight를 averaging하기 때문에 어느 한 시점에서 sharp minima에 간다고 해도 다른 weight들이 상쇄시켜주기 때문인듯.
  - CS231n에 나왔던 기법.
- curriculum learning
  - intuition(for this method): multi-task fine-tuning처럼 multi-domain training을 곧바로 적용하게 되면 training이 unstable해질 수 있다.
(gradient가 중간중간에 conflict한 방향으로 optimize될 수 있음.)
  - ImageNet pretrained backbone(ResNet18)을 사용해서 source domain을 leave-one-domain-out 방식으로 학습시켜보고 가장 성능이 좋은 순서로 source domain을 나열함.(이때 validation set을 이용해서 성능을 매김.)
  - source domain1로 학습시키고 validation set으로 평가하다가, {source1, source2}로 학습시키고 validation set으로 평가하다가, {source1, source2, source3}으로 학습시키고 validation set으로 평가함.

**Contributions**
- positive sample만 써서 CL을 구현했다.
- SWA, curriculum learning을 적용했다.
- sota가 나왔다.

**Experiments**
- PACS, DomainBed에서 평가
- t-SNE시각화
  - baseline들은 하나의 class가 여러개의 cluster로 나뉘는데, SelfReg는 여러 domain의 sample들도 class가 같다면 하나의 cluster안에 속함.
- pair의 각 요소간 distance를 시각화.
  - feature layer에서 만들어진 pair의 각 요소간 distance는 학습이 진행될수록 가까워짐. (같은 positive끼리는 가깝도록 만드니깐.)
  - logit layer에서 만들어진 pair의 각 요소간 distance는 학습이 진행될수록 멀어짐. (discriminative한 속성이 있어서 그런듯?)
- ablation study


**총평**
- augmentation-based DG를 하기 위해서 noise를 준다는 생각은 누구나 할 수 있지만, noise를 어떻게 줘야할까를 고민해봤을 때 그것에 대한 대답으로 
도메인간 변동성, intra-class consistency 둘을 고려해야 한다는 생각은 어떻게 할 수 있을까? 이 분야 논문을 많이 읽다 보면 이것은 그냥 consensus인건지, 아니면 저자만의 idea인건지는 아직 잘 모르겠다.

## Study

**읽는데 걸린 시간**
- 읽는데 4:10시간 정리하는데 1:20
- pages: p8 (without references&appendix) 

**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- 아이디어1) class가 있는 self-supervised learning이니깐 ContraGAN의 2C loss를 써보면 어떨까 ..
  - 1-1) 일단 다른 논문들을 읽어보고 어떻게 하고있는지 파악해야함.

**이해안되는 점**
- Q1. Mixup이 CL과 왜 잘맞지?
- Q2. (p3) DG분야에서는 학습 도중에 model architecture가 변경되어야하나?
- Q3. individualized in-batch dissimilarity loss를 쓸 때 positive pair를 뽑아야하는데 uniformity of the representation distribution을 고려했다고 했는데, 무슨 말인지 모르겠다.
- Q4. SWA를 적용했을 때 lr decaying을 어떻게 했다는 것인지 이해못함.
- Q5. training/validation split을 어떻게 한다는건지 전혀 감이 안온다.
