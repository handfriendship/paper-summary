# ContraGAN: Contrastive Learning for Conditional Image Generation
- Author : Minguk Kang, Jaesik Park
- 2020 NeurIPS

**Summary**
- Conditional GAN에 contrastive learning을 적용한 연구이다.
- 기존의 최신 Conditional GAN 방법론은 ACGAN, ProjGAN이 있는데, 둘 다 data-to-class relation만 사용한다. 
data-to-class relation과 함께, data-to-data relation을 같이 사용하면 더 좋은 성능을 낼 수 있을 것이라는 motivation에서 착안한다. 
data-to-data relation을 사용하는 방법으로는 여러가지가 있겠지만, 최근 가장 좋은 성능을 보이는 contrastive learning을 사용한다.
- 기존 SimCLR에서 소개된 NT-Xent loss를 기반으로 한다. NT-Xent loss는 unsupervised learning에서 사용되는 방법인데, 
이를 conditional GAN에 사용할 수 있도록 label을 leverage할 수 있는 형태인 2C loss를 제안한다.
- 2C-loss를 사용한 ContraGAN은 ImageNet, Tiny ImageNet에서 Sota가 나오고, overfitting문제 및 collapse되는 시기 등에서 좋은 모습을 보인다.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- ACGAN(Auxiliary Classifier GAN): softmax classifier를 두어 real image, fake image를 classification하도록 한다. discriminator가 adversarial objective이외에 cross entropy objective도 같이 푼다.
- ProjGAN(Projection GAN): ACGAN은 GAN이 classification하기 쉬운 이미지를 생성하는 경향을 보였다고 한다.(classification을 잘 하도록 학습하니깐)
그래서 이미지와 class embedding간의 inner product를 수행하여 그 값이 커지도록 학습시킨다.
- ContraGAN: ProjGAN은 data-to-class 정보만을 이용하니, data-to-data relation까지 capture하는 모델을 만든다.

**Main Idea**
- SimCLR에서 사용한 NT-Xent를 변형한 2C-loss를 제안했다. 
- SimCLR (Simple Contrastive Learning)
  - 1. batch_size=N인 batch의 input data를 두 가지의 다른 방법으로 augmentation함.
  - 2. 1의 결과를 encoder에 넣고, contrastive loss space인 d차원으로 projection시킴. 이러면 총 크기가 2N인 batch가 만들어짐. 여기서 같은 image에 대한 sample은 2개.
  - 3. 같은 image를 짝지어놓은 모든 pair(총 2N개)에 대해서, 같은 sample끼리의 cosine similarity는 가까워지고, 다른 sample끼리의 cosine similarity는 멀어지도록 하는 loss인 NT-Xent loss를 만듦.
  - 4. NT-Xent의 loss를 minimize하도록 encoder와 projection head를 학습함.
- 2C-loss (ContrGAN)
  - NT-Xent가 Conditional GAN에서 사용될 수 있도록 label정보를 jointly하게 학습하도록 변형함.
  - 1. batch_size=N인 batch의 input data를 준비함.
  - 2. 1의 결과를 discriminator(=encoder)에 넣고, contrastive loss space인 d차원으로 projection시킴. batch의 크기는 N으로 변함없음.
  - 3. batch내의 모든 sample에 대해, sample과 class embedding간의 cosine similarity를 구함. 또한, sample을 2개 짝지어 만들 수 있는 모든 pair에 대해 cosine similarity를 구함.
    - 2C-loss: 각 sample은 해당 class embedding과의 similarity가 가까워지도록 함. 이때 sample과 같은 class를 가진 samples들에 대해서는, 두 sample의 similarity가 가까워지도록 함.
  - 4. 2C-loss를 adversarial loss의 auxiliary loss로 두고 discriminator와 generator를 학습함.
    - discriminator를 학습할 때는 real images들에 대해 2C-loss를 적용함.
    - generator를 학습할 때는 fake images들에 대해 2C-loss를 적용함.

**Contributions**
- data-to-class relation과 data-to-data relation을 모두 이용한다.
- StudioGAN이라고 gan모델들을 비교할 수 있게 21개의 sota gan모델을 reimplementation해놨다. pytorch로 구현했다.

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
- 읽는데 5:47 + 정리하는데 1:10분
- pages: 9.5 (without references&appendix)

**비판적 사고(개선점 찾기 / 비판 / 제안 / 향후연구 등)**
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




