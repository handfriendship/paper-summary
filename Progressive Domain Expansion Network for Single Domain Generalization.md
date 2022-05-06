# Learning to Diversify for Single Domain Generalization
- Author: Zijian Wang, Yadan Luo, Ruihong Qiu, Zi Huang, Mahsa Baktashmotlagh
- ICCV 2021

## 세가지가 꼭 들어가있어야 함.
- 어떤 것을 풀려고 했는지(Summary)
- 기존에는 무슨 문제가 있는지, 기존의 방법에 비해 우리가 왜 더 좋은지(Related Works)
- 어떻게 풀었는지(Main)

**Summary**
- 어떤 것을 풀려고 했나요?
  - DG문제를 Mutual Information관점에서 바라보고 풀고 싶다. Mutual Information을 maximzie하는 방식으로 semantic information은 보존하고 style은 diverse하도록 학습하는 문제를 풀게 하고 싶어서, 그러한 시스템으로 문제를 구성하고 싶다.
- style을 synthesize하는 network는 MI를 minimize해서 교집합이 없게(source domain sample과 다르게) 학습하고 싶고, domain-invariant feature extractor인 backbone은 MI를 maximize해서 semantic information을 공유하도록 학습시킴(min-max)
- style-complement module은 MI를 minimize, Backbone은 MI를 maximize


**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- 1)기존에는 무슨 문제가 있나요?
  - Single-DG분야에서의 접근방법들은 다 color/texture에 한해서 augmentation을 하는 것이다.
- 2)기존의 방법에 비해 왜 더 좋은가요?
  - 우리의 domain diversifying전략은 style/texture/apperance관점에서 domain을 확장하는 것이다.
    - 솔직히 말은 이렇게 하는데, 정말 이걸 위해서 이전 연구들에 비해 더 신경쓴 부분이 뭔지 모르겠다.(AdaIN을 적용할 때 feature map의 pixel-wise로 mean, val을 적용했다는거?)
- adversarial image를 생성하는 방법론은, 어짜피 adversarial image도 같은 domain의 다른 class가 될 확률을 높이는 것이라서 새로운 target distribution을 커버할 때까지 domain을 diverse하는데에는 한계가 있다.


**Main Idea(어떻게 풀었는지)**
- Intuition
  - Q1. 이 논문의 아이디어가 domain을 점진적으로 확장해나가는건가?
    - ㅇㅇ. 왜냐면 feature-map의 pixel-wise mu, sigma가 서서히 학습되어가는거기 때문에, 더 많이 학습될수록 domain을 점점더 확장해나간다고 볼 수 있음.
  - Q2. 두 개의 sample에 대해서 MI를 구하는 거였나?
    - ㅇㅇ수식이 그럼. 임의로 선택된 synthesized sample과 original sample사이의 mutual information을 다룸.
  - Q3. min-max처럼 MI를 maximize했다가 minimize했다가 한다는데, 이게 생각처럼 균형이 맞게 학습이 되나? G는 original sample과 generated sample간의 diversity가 높아지기 위해서 MI의 upperbound를 작게 만들어야 하고, task model은 latent space에서 같은 semantic의 sample들끼리 가까워져야 하므로 MI를 maximize해야한다. 만약 G에서 MI의 upperbound를 작게 만드는것이 diversity가 높아지는게 아니라 semantic information이 멀어지게 하는 것이면 어떡하나? task model에서 MI를 maximize하는 것이 semantic을 가깝게 하는게 아니라 generated sample의 style정보를 가깝게 하는것이면 어떡하나?
    - 그럴까봐 장치를 걸어둠. G에게는 Semantic Consistency(class conditional maximum mean discrepancy를 minimize시키는것) loss를 두고, task model에게는 contrastive learning loss를 둠.
- 학습과정

- Objective
- Architecture 

**Contributions**
- MI를 이용한 framework을 제시함.
- 기존의 AdaIN방식을 조금 바꿔서 feature map에서의 pixel-wise한 방법을 적용함.

**Experiments**
- Digits-DG, PACS, Corrupted CIFAR-10
- t-SNE visualization
- hyperparameter tuning
- ablation study
  - style-complement network, min MI., max MI., style modification(AdaIN의 변형) 

**총평**
- Mutual Information을 적용해볼까 하는 생각은 있었는데, 어떻게 적용하는지 이걸 보니깐 깨닫게 됬다. 

## Study

**읽는데 걸린 시간**
- 읽는데 3:35시간 정리하는데 2:30분
- pages: p8 (without references&appendix) 

**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- 아이디어1. AdaIN을 적용할 때 feature map의 pixel-wise로 mean, val을 적용했다. 기존 연구들은 feature map당 하나의 mean, var가 존재하고. 이거의 적절한 중간 지점을 찾을 순 없을까?
  - 지금 드는 생각으로는, 그럴려면 feature map내에서 비슷한 pixel들끼리 aggregate할 수 있어야 할텐데, 이걸 하려면 뭔가 network이 더 필요할 것 같기도 하다..

**질문(의문점)**
- Q1. min-max처럼 MI를 maximize했다가 minimize했다가 한다는데, 이게 생각처럼 균형이 맞게 학습이 되나? G는 original sample과 generated sample간의 diversity가 높아지기 위해서 MI의 upperbound를 작게 만들어야 하고, task model은 latent space에서 같은 semantic의 sample들끼리 가까워져야 하므로 MI를 maximize해야한다. 만약 G에서 MI의 upperbound를 작게 만드는것이 diversity가 높아지는게 아니라 semantic information이 멀어지게 하는 것이면 어떡하나? task model에서 MI를 maximize하는 것이 semantic을 가깝게 하는게 아니라 generated sample의 style정보를 가깝게 하는것이면 어떡하나?
- Q2. 하나의 feature map당 하나의 mean, var를 두는게 아니라 pixel level로 mean, var를 두는게 더 좋은 이유에 대한 직관은 무엇인가?
  - 더 diverse하게 domain을 augment할 수 있다.
- Q3. K개의 transformation을 뒀는데, 그냥 transformation module은 하나만 두고 channel을 늘리는 것보다 더 좋은 효과가 있나?
- Q4. 이 논문의 아이디어가 domain을 점진적으로 확장해나가는건가?
