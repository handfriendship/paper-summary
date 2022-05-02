# Adversarial Teacher-Student Representation Learning for Domain Generalization
- Author: Fu-En Yang, Yuan-Chia Cheng, Zu-Yun Shiau, Yu-Chiang Frank Wang
- NeurIPS 2021

## 세가지가 꼭 들어가있어야 함.
- 어떤 것을 풀려고 했는지(Summary)
- 기존에는 무슨 문제가 있는지, 기존의 방법에 비해 우리가 왜 더 좋은지(Related Works)
- 어떻게 풀었는지(Main)

**Summary**
- 어떤 것을 풀려고 했나요?
  - DG를 data augmentation쪽으로 해서 풀고 싶어했을 것 같다. SWA(stochastic Weight Averaging) + SSL + Data Generation 등등 여러 technique들을 적절히 섞다 보면 sota가 나오겠지 .. 싶었을 것
- perturbated data Generation을 하고 그걸 contrastive learning 비슷한 알고리즘으로 학습시켜서 DG에 사용했다.
- data generation을 하는 encoder와 student model을 adversarial training시킨다. encoder는 더 diverse한 data를 생성하려 하고, student model은 그 data를 인코딩한게 원본 feature랑 비슷하게끔 학습시킨다.
- student model의 exponential moving averaging을 한게 teacher model이다. 

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- 1)기존에는 무슨 문제가 있나요?
  - 딱히 기존의 어떤 점을 개선한건지 잘 모르겠는데 .. 그냥 이것저것 섞다보니 sota가 나온거같은데 ..
- 2)기존의 방법에 비해 왜 더 좋은가요?
  - 기존의 data generation기반의 data augmentation방법은 aversarial sample, adversarial training으로 CNN모델 학습 등을 하는데, 우리는 augmented data와 existing data간의 discrepancy가 maximize되도록 하는 방법을 쓴다.(CL + adversarial training)
  - 기존의 SWA와는 다르게 teacher-student model로 EMA를 나타냈다.

**Main Idea(어떻게 풀었는지)**
- Intuition
  - Domain Augmentation
    - 크게 Contrastive Learning이라고 볼 수 있음. EMA를 해서 student의 ensemble model격인 teacher model의 representation을 original sample이라고 한다면, student model의 representation이 data augmentation을 한 sample이겠지.
    - positive sample만 이용한다고 볼 수 있음.
  - Teacher-Student 구조
    - knowledge distillation이라기 보다는 SWA에 가까움.
    - warm up을 거친 뒤 안정화된 상태의 student들만 ensemble하겠다는 거지.(이것도 SWA)
    - student model의 ensemble이 teacher model
  - Adversarial training
    - augmenter는 z와 차이가 많이 나는 z~를 만들려고 하고, student model은 z와 가까운 z~가 만들어지도록 x~를 encoding함.
  - 처음의 warm-up이 필요한 이유
  - teacher model을 freezing하는 이유
    - warm up을 거친 뒤 안정화된 상태의 student들만 ensemble하겠다는 거지.(이것도 SWA)
  - classifier를 freezing하는 이유
    - 애초에 x~의 질이 안좋거나, 설령 x~가 좋아도 z~의 질이 안좋을 가능성이 많음.

**Contributions**
- related works 2)와 같음.

**Experiments**
- PACS, Office-Home에서 실험.
- backbone을 ResNet18, ResNet50으로 바꿔가면서 실험.
- Ablation Study
  - augmentation algorithm을 바꿔봄.
  - architecture를 바꿔봄.
    - Siamese
    - teacher model대신 student model을 test에 써봄.
- PACS, DomainNet에서 single source DG실험.
- momentum coefficient
- visualization
  - vanilla model에 비해 약간 더 잘 clustering되어있음.

**총평**
- 성능이 잘 나왔다는 건 알겠는데, 그래서 이 모델이 주는 takeaway가 뭔지 모르겠다. 확실한 Motivation과 intuition을 갖고 만들어진 모델이라기 보다는 adversarial training기반으로 이것저것 해보다가 성능이 잘나와서 낸 느낌?

## Study

**읽는데 걸린 시간**
- 읽는데 3:35시간 정리하는데 1:15분
- pages: p10 (without references&appendix) 

**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- ...

**이해못한 점**
- Q1. CrossGrad, DDAIG가 domain variation을 나타낼 수 없다는 말이 무슨뜻이지?
- Q2. Visualization의 결과에서 generation된 data에 대한 plot도 보여주는게 일반적이지 않나..?

