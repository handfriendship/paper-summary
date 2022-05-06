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
  - 1-1. DG를 위한 data augmentation을 사용할 때, 사용할 augmentation기법으로 무엇을 선택할지, 얼마만큼의 강도로 augmentation을 줘야할지 결정하는 것이 어렵다는 문제가 있다.
  - 1-2. 또한 augmented data가 safe한지, effect한지 gaurantee하기 어렵다는 문제가 있다.
- 2)기존의 방법에 비해 왜 더 좋은가요?
  - 1-1.의 문제를 해결했다. 이 논문에서 사용한 data augmentation기법은 AdaIN인데, 이 framework은 꼭 autoencoder가 아니고 STN, HRNet이어도 된다. 즉, 어떤 Generator를 사용하든 상관없이 동작한다.
    - DG분야에서는 어떤 domain이 target으로 올지 결정할 수 없는데, 그걸 몰라도 풀 task(e.g. image classification)만 알면 쓸 수 있는 기법.
  - 1-2.의 문제를 해결했다. safe, effect를 보장하기 위한 여러 objective를 제시했다.

**Main Idea(어떻게 풀었는지)**
- Intuition
  - Q1. style component의 transformation network가 여러개 있는 이유?
    - ?
G에서 생성한 sample이 그러한 semantic을 가진 sample이 되도록 G가 학습되는 것.
  - Q2. Backbone은 MI를 maximize, Style-component은 MI를 minimize?
    - 생략
  - Q3. adversarial training을 하는 이유?
    - 생략.
- 학습과정
  - 1) M, G를 jointly optimize시킨다.(M, G의 parameter update)
  - 2) G에서 생성한 synthesized images에 M을 fine-tuning시킨다.
  - 3) G에서 생성한 synthesized images를 source domain과 합친다.
  - 4) M을 unified domain에 다시 tuning한다.
  - 5) G를 새롭게 initialize하고 M과 함께 optimize시킨다.
- Objective
- Architecture 

**Contributions**
- safe, effect한 data augmentation을 잘 적용할 수 있도록 하는 framework을 제시함.

**Experiments**
- Digits-dataset, CIFAR10-C, SYNTHIA dataset에서 평가.
- Evaluation protocol
  - Digits-dataset의 경우, MNIST만 source, 나머지를 target domains. source dataset을 돌아가면서 해보지는 않음.
  - CIFAR10-C의 경우 CIFAR10을 source, CIFAR10-C를 target
  - ..생략
- hyperparameter tuning 
  - # iterations, loss scaling parameter
- Few-shot Domain Adaptation
  - target domain의 sample을 몇개만 보여줌(parameter update ㅇㅇ)

**총평**
- augmented data가 가져야 할 조건(safety, effectiveness)에 대해서 다시 한 번 생각해 보게 한 논문.

## Study

**읽는데 걸린 시간**
- 읽는데 3:35시간 정리하는데 0:35분
- pages: p10 (without references&appendix) 

**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- 아이디어1. evaluating protocol을 제시하면 어떨까? 새로운 데이터셋을 synthesize해서 거기서 evaluate를 한다던가 ...

**질문(의문점)**
- Q1. 중간에 task model M이 overfitting되어서 성능을 저해할 위험은 없는가? M은 early stopping을 하는가? training procedure가 어떻게되지?
- Q2. G를 안버리고 계속 쓰면 어떻게되나? 
- Q3. L_unseen으로 학습하면 L_div도 task model M을 학습할 때 적용되는건가? 아니면 L_div는 G에만 적용되는건가?
- Q4. MNIST-test set에 validate을 해도 되는건가?

