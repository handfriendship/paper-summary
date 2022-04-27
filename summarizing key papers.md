## CutOut
**previous works의 한계**
- dropout은 CNN에서 잘 동작하지 않는다.(CNN은 NN에 비해 적은 params개수를 가지고 있기 때문에, CNN의 경우 local region을 보고 판단하는거라, 어느 한 node가 drop되도 주변의 node로부터 정보를 손쉽게 얻을 수가 있어서 dropout을 했을 때의 regularizer효과가 떨어진다.)

**Approach**
- image의 random영역을 없앰.
- image-level approach
- patch의 모양보다는 size에 영향을 더 받는 것이 실험을 통해 관측됨. 모양은 다 정사각형으로 처리.
- patch의 위치는 image의 밖에서부터 시작하도록 하는 것도 허용해야 더 잘됨.
- 원래 매 iteration마다 activation이 가장 크게 되는 영역을 구하고, 다음 iteration에서 해당 부분을 masking하는 방식으로 augmentation을 하려고했으나, 성능 증가 폭이 크지 않았대.

**Intuition**
- 원래 모델이 image의 특정부분(e.g.개의 얼굴)만을 보고 classification한다면, 해당 부분이 없을 때도 같은 label을 내뱉게 학습시킴. 결국 어떤 prediction을 할 때 많은 공간적 영역을 보고 결정하게끔 함.
  - 실험에서 cutout을 적용했을 때 어떤 image에서 activation되는 영역이 더 넓다고 함.
- dropout은 random drop인데 비해 cutout은 continuous한 영역을 drop한다.
- Occlusion에 더 강건한 모델을 만들겠다.


## Mixup
**previous works의 한계**
- 기존 방법론들(ERM-based) 하나의 data distribution에서만 sampling되는 것이라서 다른 domain에 잘 동작하지 못한다.
- 그나마 VRM(training data distribution에서 약간만 벗어나는 분포에서도 동작할 수 있게 하는것, =augmentation)이 좀 더 낫다. 

**Approach**
- 두 raw image를 그냥 섞음, label도 해당 비율만큼 섞음.
- image-level approach

**Intuition**
- mixup은 두 클래스 간의 decision boundary가 ERM에 비해서 부드럽습니다. 이는 실제로 mixup이 ERM에 비해서 과적합이 덜 발생한다.
- mixup은 기존 classifier에 대한 regularizer라고도 볼 수 있다.
- 적대적 데이터에도 강건함을 가질 수 있게 도운다.
- mixup 이 calibration 의 관점에서도 성능 향상을 보인다는 연구 또한 존재한다. calibration 이란 모델이 예측에 대해 확신하는 정도와 예측이 맞을 확률의 관계에 대한 것으로, DNN 모델의 over-confident 한 문제를 다루기 위해 calibration을 향상시키려 한다.


## CutMix
**previous works의 한계**
- CutOut은 이미지의 일부를 쓸 수 없다.
- Mixup은 locally unrealistic image를 생성할 위험성이 있다.(CNN은 local inductive bias가 있으므로)

**Approach**
- 한 이미지의 일부를 crop해서 다른 이미지에 그대로 붙여넣음.
- loss function은 더 가까운 image를 맞추도록 하는게 아니라, augmented image에 들어있는 class들의 비율을 맞추도록 해야함.
- image-level approach
- 어떤 두 이미지를 조합할지도 random, 이미지를 얼마의 비율로 조합할지도 random, 이미지의 어느 부분을 잘라서 다른 이미지의 어느 부분에 붙여넣을지도 random

**Intuition**
- Q1. CutMix idea
  - 모델이 data의 where(각 class에 해당하는 object가 어디에 있는가), what(어떤 class의 image인가), how large(두 image가 얼마만큼의 비율로 섞여있는가)를 학습하는 과정에서 더 어려운 task를 푸는것.
  - CutMix를 적용하면 task가 많이 어려워지기 때문에 길게 학습해야 효과를 볼 수 있다. 
- Q2. CutMix는 어디에 사용하면 좋은가?
  - A2. label을 섞을 수 있는 task에 사용해서 풀면 좋다.
  - Object Detection에서는 bounding box label을 섞는 것이 힘들기 때문에 잘 동작하지 않음.
  - Segmentation에서는 image mixing에서 얻는 gain은 있을 수 있으나, label이 섞이진 않기 때문에 거기에서 얻는 gain은 없어서 성능 증가가 덜 기대된다.


## image를 segmentation해서 갖다붙이는 것?


## DG를 위한 아이디어
- 아이디어1) 다른 domain, 다른 class에 대해 mixup or cutmix를 해보면 어떨까? 그리고 domain label은 DANN에서처럼 반대로 흘려주고.
- 아이디어2) ViLT에서 pre-training objective로 적용했던 word-image alignment처럼, IPOT를 이용해서 서로 다른 두 domain간에 unsupervised image aligning을 할 순 없을까?
