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
- Domain Generalization
  - 우리 방법은 성능은 sota와 필적하면서 간단함.
- Feature Interpolation and Augmentation
  - 우리의 interpolation방법은 semantic정보는 보존한다. (기존 방법들은 semantic정보를 안보존)
  - MixStyle과 비교: mixstyle은 domain label이 필요하지만 우리 방법은 안필요함.
- Domain Randomization
  - 다양한 rendering style을 만들어서 test data가 새로만든 data 중 하나와 비슷할 거라고 기대하는 방법
- Stochastic Neural Network
  - VAE처럼 training시 random variable을 추가해서 학습하는 방법

**Main Idea**
- 방법론의 큰 분류: 이것은 raw image에 대한 augmentation이 아니라, feature에 대한 augmentation임.
  - model의 중간 layer의 output인 activation에 대해 noise를 정해주는 것이므로.
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
    - A. 모든 domain을 다 고려한 intra-class variance를 계산한 것이므로 그렇다고 할 수 있을듯?
- Model Architecture
  - (Conv-Relu-Maxpool)x4 + (softmax)x1
  - loss: ERM + lambda x (ERM-SFA)
  - 처음 500step은 SFA를 안쓰고 warm-up을 한 후에 시작함.

**Contributions**
- gaussian noise를 잘 가해주는 data augmentation기법을 제안했다. 
- 간단하고 구현하기 쉽고 plug-and-play가 가능하다.

**Experiments**
- Digit-DG, VLCS, PACS 데이터셋에서 실험함.
  - leave-one-domain-out방식으로 실험함.
  - 하이퍼파라미터(K, lambada, iteration, etc..)는 task-specific하게 정해줌.
  - 결과: SOTA만큼은 아니지만 필적할 만한 성능이 나왔고, 우리께 더 간단하니깐 좋다는 내용임.
- noise distribution을 laplace distribution, uniform distribution으로 바꿔보면서 실험함.
  - 결과: 크게 상관없대.
- 어디에 SFA layer를 넣는게 좋은지 실험함.
  - 결과: 크게 상관없대.
- noise strength실험.
  - 결과: 크게 상관없대.
- ablation study(alpha, beta, ksaai)
- visualization
  - SFA를 적용하고 나면 데이터 영역이 훨씬 더 확장됨.(더 많은 도메인에 대해 generalize할 수 있다는 의미.)
  - adaptation없이 그냥 SFA 모듈을 적용했을 때는 class boundary가 blurry해지는데 SFA를 training시키고나면 명확해짐.
  - test domain은 train data domain보다 더 blurry한데, SFA 모듈을 training시키고나면 좀더 clear해짐.


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
  - intuition: 
    - multi-task learning에서 in-batch에 여러 domain의 sample들이 있도록 하는 경우도 있는데, 그때의 intuition을 찾아보기. (T5같은데서 찾을 수 있을듯)
    - data-to-data relation을 사용가능한듯?
      - 현재 batch = [cartoon(horse), cartoon(cat), cartoon(cow), cartoon(dog), cartoon(horse)]이렇게 구성되어 있다면 각 sample에 대해 seven category중 어느 곳에 속하는지를 softmax한 후 class-label에 해당하는 softmax value에 대해 cross-entropy를 하는 것이다. 이건 data-to-data relation을 이용하는건 아니지.
- 현재 covariance를 update 해나가는 방식은 momentum처럼 decay factor를 줘서 조금씩 기존껄 떨어뜨리고 있다. 이렇게 하면 long-term dependency가 포착이 안될수도 있지 않을까? Residual Connection을 준다거나 LSTM처럼 memory cell을 둔다거나, transformer처럼 병렬적으로 더할 수 있는 방식이 있으면 좋을듯.
  - 1) 우선 머릿속으로 문제점이 뭔지 구체화해보고, 아이디어가 말이 되는지, 실현가능한지를 구현 레벨 수준으로 정리해보기.
    - Q1. 어떤 data에 대한 long-term dependency?
      - ㅇㅇ
    - Q2. 가장 ideal한 covariance matrix는 어떻게 구할 수 있는가?
    - Q3. covariance matrix의 update과정이 어떻게되는가?
  - 2) 이런 문제가 있다는 것을 입증하려면 우선 모든 데이터를 다 고려할 수 있는 방법으로 covariance를 구한 뒤(mini-batch가 아니라 whole-batch로), 그게 성능이 더 좋다는걸 보이면 될듯
  - 방법1) 그러면 어느 곳에 residual connection을 줄지를 결정해줘야 하는데, 그럴려면 batch data간 가중치를 주는게 좋을듯.
    - 가중치는 어떤 방식으로 결정할 수 있을까 ..
  - 방법2) momentum -> nesterov momentum으로 바꿔본다.
  - 결론: 안되는 아이디어
    - 단계1)까지 시도해봤음.
    - 이유1) 애초에 long-term dependency를 포착할 필요가 없음.(왜냐면 이전 데이터들은 덜 optimize된거라 부정확한 covariance matrix라서 이걸 포착할 필요가 없음.
    - 이유2) 가장 ideal한 covariance matrix를 만드는 것은 그냥 mini-batch size를 튜닝하는 정도인듯.
      - 가장 ideal한 covariance matrix는 같은 class를 가지는 sample들을 batch로 묶지 말고 하나하나씩 공분산을 측정하는 것.
      - 예를 들어 숫자 9 class에 속하는 sample들이 domain당 2000개씩 있다고 하면, (2000, 3) x (3, 2000)을 해서 (2000, 2000)을 만드는 것임. 물론 step수를 무한히 한다면 이게 가장 좋겠지만, mini-batch의 장점도 있음.
- sample간의 중요도를 반영하기(hard negative)
- class anchor를 두고 clustering처럼 풀기?(+multiple anchor를 두는 식으로도 모델링 가능한가? domain과 무관한 class anchor를 두는것.)
  - 이렇게 하려면 결국 class anchor끼리는 멀어지게 하고 intra-class sample들은 class anchor를 기준으로 가까워지게 해야해서 contrastive loss를 써야함.
- gaussian mixture model?

**질문**
- Q1. SFA-A는 그렇다 쳐도 SFA-S는 왜 잘되나? 이것이 잘되는 것에 대한 intuition이 뭔가? 
- Q1-1. SFA-A가 잘되면 general-purpose regularization technique도 DG에 효과가 있을 수 있다는 것 아닌가?
- Q2. MixStyle은 왜 비교안하냐? 똑같은 data augmentation 종류인데?
- Q3. KxK covariance matrix에서 k-dim noise를 N개(domain의 개수)만큼 sampling하는건가? 아니면 같은 noise가 각 도메인에 더해지는건가?
