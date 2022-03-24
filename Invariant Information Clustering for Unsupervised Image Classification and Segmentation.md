# Invariant Information Clustering for Unsupervised Image Classification and Segmentation
- Author : Xu Ji, João F. Henriques, Andrea Vedaldi
- 2019 ICCV 

**Summary**
- Mutual Information을 최대화시키는 IIC loss를 통해 기존 unsupervised image clustering과 unsupervised image segmentation에서 sota를 찍었다.
- image clustering task에서 mutual information을 구하기 위해서는, 각 image sample에 대해 그것을 transform시킨 image 쌍을 만들고, CNN based backbone을 이용해 representation을 구한다. 그다음 n개의 모든 sample들에 대해 nxn joint probability matrix를 만들고, mutual information 공식에 집어넣는다.
- Mutual Information을 최대화시키는 방향으로 학습하면 다음과 같은 이점이 있다.
  - 모든 cluster들이 equal하게 선택되도록 하기 때문에 degenerate solution을 피할 수 있다.
  - mutual information수식의 조건부 엔트로피인 H(z|z') term이 없으면 어떤 sample x_i의 soft label이 모든 cluster에 대해 uniform하게 매겨질 수 있다.
- main objective인 IIC-loss에다가 추가로 auxiliary task인 overclustering문제를 푼다.
- unsupervised기반의 image clustering, segmentation에서 좋은 성능을 보여줬고, unsupervised로 overclustering pre-training하고 supervised로 mapping하는 semi-supervised task에서도 좋은 성능을 보여줬다.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- 크게 4가지 방향에 대한 연구가 있었음.
- Co-clustering and mutual information
  - 정보이론을 기반으로 한 다른 연구들도 있었지만 어떤 정보를 maximize하는지, 다루는 정보의 특성이 어떤지(continuous vs discrete, 어떤걸 기준으로 뽑은 feature 등..)가 다 다르다.
  - IMSAT: data와 representation간의 mutual information을 maximize한다. continuous random variable의 mutual information을 구함. information이외에 다른 term을 objective에 추가했다는듯?
  - DeepINFOMAX: spatially-preserved feature와 compact feature간의 information(entropy)을 maximize한다. continuous random variable의 mutual information을 구함. information이외에 다른 term을 objective에 추가했다는듯?
- Semantic clutering vs intermediate representation learning
  - 바로 semantic clustering을 구하는 방식과 중간단계의 representation을 구하는 방식으로 나뉜다.
  - 중간단계의 representation을 구하는 방법은 post-processing을 요하고, 아주 다양한 방법(e.g. clustering, triplets, ...)이 사용된다.
  - DeepCluster: 중간단계의 representation을 구함. 자동으로 semantic clustering을 하지 못함.(후처리를 해야함.) 성능이 안좋음.
- optimising image-to-image distance
  - examplars는 scale up이 잘 안된다. (segmentation문제는 특히 computation이 많이 필요함.)
  - DeepCluster 및 clustering방법론들은 degenerate solution이 발생하는 문제점이 있고, 이를 피하기 위해 많은 잡다한 후처리 pipeline을 추가해야한다.
- invariance as a training objective
  - data에 distortion을 주거나 transformation을 가해서 invaraint하게 만드는 기법은 여러 연구들에서 많이 사용되었다.

**Main Idea**
- 1. IIC의 information theoreic base 
  - 1.1 IIC 설계
    - 각 input sample에 대해 transform을 가한 pair를 만듦.
      - transform은 scaling, skewing, rotation, flipping, 등등 ..
    - 이것을 CNN-based backbone에 넣어 representation을 뽑음.
    - N개의 모든 sample과 N개의 pair에 대해 NxN의 joint probability matrix를 만듦.
    - (P + P.T) / 2 를 하여 symmetric하게 만듦.
    - 이것을 mutual information 수식에 대입함.
  - 1.2 IIC의 의미
    - representation z, z'를 비슷하게 만듦.
    - input x와 x'에 대해서 공통적인 부분만을 뽑는 효과가 있음.
    - 왜냐면 mutual information을 maximize한다는 것이, 하나를 알면 다른 하나를 알기 쉽게 한다는 뜻이기 때문.
  - 1.3 IIC가 degenerate solution을 피할 수 있는 이유
    - mutual information 수식 I(z,z') = H(z) - H(z|z') 에서 H(z)의 의미가, 모든 cluster를 equal하게 고르게 하는 것이기 때문.
  - 1.4 IIC가 단순 entropy만을 썼을 때와의 차이점
    - 현재는 clustering의 결과로 soft clustering나오게됨.
    - 단순 entropy만을 loss로 쓰면 soft clustering이 uniform하게 될 수 있음.
    - IIC는 조건부 엔트로피인 H(z|z')을 maximize하기 때문에 hard clustering이 되게끔 하는 효과가 있다.
- 2. IIC를 이용한 image clustering
  - main head: ground truth clustering의 개수인 k_gt로 clustering을 함.
  - auxiliary head: k_gt보다 더 많은 개수로 overclustering을 하게 함.
    - auxiliary head에서는 dataset에 distractor로 분류된 image도 학습하게함.
- 3. IIC를 이용한 image segmentation
  - idea: 
    - image clustering에서는 transform시킨 pair와 common한 부분을 찾도록 학습되었음. 
    - segmentation에서는 
  - 과정:
  - input image를 patch로 자름.
  - 

**Contributions**
- IIC loss를 제시함.
- IIC loss는 강한 수학적 기반을 가지고 있음.
- IIC loss는 simple함.
- image clustering, segmentation에서 sota를 아주 높은 gap으로 outperform함.

**Experiments**
- CIFAR10, Tiny ImageNet, ImageNet에 대해서 previous sota 모델들과 성능비교를 함. 이 중 Tiny ImageNet, ImageNet에서 FID를 기준으로 유의미한 성능 향상이 있었음.(new sota달성)
- ProjGAN과 ContraGAN의 overfitting 정도를 비교함.(training set과 validation set의 accuracy gap을 기준으로, 얼마나 빨리 overfitting되는지를 봄.)
- ProjGAN과 ContraGAN이 얼마나 빨리 training collapse에 도달하는지 비교함.
- ProjGAN과 ContraGAN이 얼마나 training이 stability한지를 비교함.(spectral norm을 기준으로)
- ablation study를 수행함.
  - batch size를 다르게 했을 때의 변화를 관측함. (batch가 클수록 잘된다.)
  - loss를 바꿔보면서 2C-loss가 가장 좋다는 것을 보임.
  - 2C-loss를 단독으로 사용했을 때와 2C-loss + positive sample를 augmentation을 했을 때를 비교해서 2C-loss의 단독 사용이 더 좋음을 보임.
  - 2C-loss + CR(Consistency Regularization)를 같이 사용했을 때 시너지를 냄을 보임. (둘 중 하나라도 빼게 되면 성능이 떨어짐을 보임.)

**총평**
- 

## Study

**읽는데 걸린 시간**
- 읽는데 4:30 + 정리하는데 1:10분
- pages: 9.5 (without references&appendix)

**비판적 사고**
- 어떤 class의 image를 생성할 때, 다양한 image를 생성하지 못하는 문제가 발생할 수 있다.(mode collapse문제.) 현재의 discriminator는 generator와는 독립적으로 2C-loss를 줄이도록 학습한다. 이것은 class내의 diversity를 줄여서 entropy를 낮추도록 학습하므로 diversity문제를 일으킬 수 있다.
- 현재는 batch내의 각 sample을 기준으로 class embedding 및 같은 label을 공유하는 sample들에 대한 cosine거리를 가깝게하도록 설계되어있다. 
하지만 Comparison with APS에서, class embedding이 anchor역할을 하기 때문에 2C-loss만 단독으로 사용하는 것이 더 좋다고 했다.
그러면 loss를 만들 때, batch내의 각 sample을 기준으로 하지 말고, batch내에 존재하는 class embedding을 기준으로, 해당 class에 해당하는 모든 sample들의 cosine거리를 가깝게 하는 식의 loss를 만들면 안되나?
- 2C-loss + CR 조합이 시너지 효과를 일으키는 것을 단순 실험으로만 보였는데, 이에 대한 설명이 있으면 좋겠다.

**이해못한 점**
- 2C-loss + positive sample augmentation을 했을 때 왜 더 나쁜지에 대한 저자의 speculation을 이해하지 못함.
- GAN에 대한 이해가 아직 부족하여 GAN 자체에 관한 지식들을 아직 다 이해하지 못함.
  - adversarial loss
  - Spectral Normalization
  - Consistency Regularization
