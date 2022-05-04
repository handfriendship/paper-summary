# UNCERTAINTY MODELING FOR OUT-OF-DISTRIBUTION GENERALIZATION
- Author: Xiaotong Li, Yongxing Dai, Yixiao Ge, Jun Liu, Ying Shan, Ling-Yu Duan
- ICLR 2022

## 세가지가 꼭 들어가있어야 함.
- 어떤 것을 풀려고 했는지(Summary)
- 기존에는 무슨 문제가 있는지, 기존의 방법에 비해 우리가 왜 더 좋은지(Related Works)
- 어떻게 풀었는지(Main)

**Summary**
- 어떤 것을 풀려고 했나요?
  - feature-level augmentation으로 DG를 풀고자 했다. AdaIN의 feature statistics manipulation기반 알고리즘으로 이것저것 아이디어를 던지고 시도해보면서 DG를 풀고자 했다.
- channel-wise feature statistics를 deterministic하게 딱 하나의 value로 정하지 않고 확률적으로 달라지는 값으로 정함.
- test time에 어떤 domain shift가 일어날지 모르니, 많은 unseen distribution에 대응하고자 기존 sample을 중심으로 여러 style측면의 variation이 추가된 sample을 생성하는 식으로 augment하겠다는 것이 intuition
- instance-level로 각 mu, sigma를 구한 뒤에 across batch로 mu들의 평균과 분산, sigma들의 평균과 분산을 구한다. 
  - mu들의 평균과 분산을 이용해 multivariate gaussian을 만들고 거기서 mu를 sampling, sigma들의 평균과 분산을 이용해 multivariate gaussian을 만들고 거기서 sigma를 sampling한다.
  - AdaIN의 formula와 같이, 기존 sample의 평균과 분산을 sampling된 평균과 분산으로 교체한다.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- 1)기존에는 무슨 문제가 있나요?
  - 기존 feature-level augmentation기반의 알고리즘은 feature statistics를 deterministic하게 정한다는 문제점이 있다.
- 2)기존의 방법에 비해 왜 더 좋은가요?
  - feature statistics를 probabilistic하게 둔다.
  - pAdaIN <= Mixstyle <= DSU (가장 일반화된 방법)
  - direction관점: pAdaIN, Mixstyle은 synthesized image를 생성하는 direction이 한정되어있다.
  - intensity관점: pAdaIN, Mixstyle은 image를 linear interpolate하는데, 그렇게 만들 수 있는 intensity가 한정되어 있다.
    - p16보면 한방에 이해됨.
- pAdaIN에서 ablation study로 feature statistics를 normal distribution에서 sampling된 random값으로 교체하는 실험을 했었는데 결과가 안좋았다고 한다.
  - pAdaIN에서는 feature statistics가 from natural image distribution로부터 추출된 것이 아니라 normal distribution에서 추출되었으므로 shift가 일어나서 결과가 그렇게 되었다고 했다.
  - 이 관점에서 보면, DSU는 feature statistics가 존재하는 distribution을 natural images가 존재하는 분포의 부분집합이 가지는 distribution으로 둔 것이다.
  
**Main Idea(어떻게 풀었는지)**
- Intuition:
  - pAdaIN, Mixstyle을 일반화해보자.
  - Q1. method에서 across batch로 평균과 분산을 구해야하는 이유가 뭘까?
    - batch를 표본으로보고, 표본량으로 모집단(source domains)를 모사하기 위해.
- instance-level로 각 mu, sigma를 구한 뒤에 across batch로 mu들의 평균과 분산, sigma들의 평균과 분산을 구한다. 
  - mu들의 평균과 분산을 이용해 multivariate gaussian을 만들고 거기서 mu를 sampling, sigma들의 평균과 분산을 이용해 multivariate gaussian을 만들고 거기서 sigma를 sampling한다.
  - AdaIN의 formula와 같이, 기존 sample의 평균과 분산을 sampling된 평균과 분산으로 교체한다.

**Contributions**
- pAdaIN <= Mixstyle <= DSU 더 일반화된 방법.
- feature statistics를 probabilistic하게 둔다.

**Experiments**
- uncertainty distribution을 fixed sigma를 가진 guassian으로, 또 uniform distribution으로 바꿔봄.
- PACS, Office-Home에서 성능 비교(image classification)
- semantic segmentation에서 성능 비교
- Instance Retrieval에서 성능 비교(person re-ID)
- robustness towards corruptions(Imagenet-C)
- ablation study
  - ResNEt에서 어느 layer에 DSU를 적용해야할지
  - probability p에 대한 hyperparameter tuning
- qualitative analysis
  - mean, var의 training/testing분포를 DSU적용 전과 후의 경우로 나눠서 plot해서 비교해봄.
- t-SNE visualization

**총평**
- AdaIN계열 augmentation의 끝판왕 논문. 이게 모든 방법론을 다 generalize할 수 있는 방법론 같아서, 여기서 어떻게 더 improve될 수 있을지 떠오르지가 않는다..

## Study

**읽는데 걸린 시간**
- 읽는데 3:21시간 정리하는데 1:00분
- pages: p9 (without references&appendix) 

**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- 아이디어1. DSU는 pAdaIN과 Mixstyle의 generalized된 버전이다.
  - (pAdaIN은 batch내의 다른 image의 feature statistics를 갖다쓴다는 것이고, Mixstyle은 어떤 두 domain에서 추출된 image들의 interpolation인데, interpolation weight를 1.0으로 주면 pAdaIN이 되기 때문.)
  - 포함관계: DSU >= Mixstyle >= pAdaIN 
  - DSU를 더 generalize시킬 순 없을까? 
  - 현재 general-purpose augmentation기법을 이용해도 성능이 오르니깐, 그것까지 포함할 수 있는 general한 버전의 DSU가 말야 ..
    - GMM
    - A Simple Feature Augmentation for Domain Generalization에서 나온 covariance matrix를 계속해서 만들어나가는 방법(variance를 batch내에서만 구하지 말고, 전체 데이터에 대해서 구하자는 뜻.)
      - 통계학에서 표본의 개수가 30개 이상이면 모집단을 근사할 수 있다는 것이 알려져있는데, batch내의 sample만 이용해도 충분할 것 같다는 생각이 든다.
      - 만약 이게 성능이 오른다고 하더라도 조금 오를 것 같고, 이게 contribution이 될까?라는 생각이 든다.
    - convex hull, conic hull같은걸 이용할 순 없을까?(Mixstyle이 convex combination인데, 그걸 넘어설 수 있는 sample이 있으면 좋으므로)
    - 현재 분산을 구하는 방식이 manifold를 semantic한 방향으로 가장 잘 확장하는 방법이라고 볼 수 있을까?

**완벽하게 이해안된 점**
- Q1. method에서 across batch로 평균과 분산을 구해야하는 이유가 뭘까?



