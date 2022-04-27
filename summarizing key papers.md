## CutOut


## Mixup
**previous works의 한계**
- 기존 방법론들 하나의 data distribution에서만 sampling되는 것이라서 다른 domain에 잘 동작하지 못한다.

**Approach**
- 두 raw image를 그냥 섞음, label도 해당 비율만큼 섞음.
- image-level approach

**Intuition**
- Q1. CutMix idea
  - 두 image를 섞음으로써 새로운 distribution의 data를 만들 수 있다는듯?
- Q2. Mixup는 어디에 사용하면 좋은가?
  - 어디에든 사용할 수 있다.
  - multi-label task에 사용하면 더 좋을듯.


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


