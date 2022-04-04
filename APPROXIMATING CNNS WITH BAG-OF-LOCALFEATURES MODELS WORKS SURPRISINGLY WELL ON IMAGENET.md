# APPROXIMATING CNNS WITH BAG-OF-LOCAL FEATURES MODELS WORKS SURPRISINGLY WELL ON IMAGENET
- Author : Wieland Brendel, Matthias Bethge
- 2019 ICLR

**Summary**
- Bag of words에 기반한 모델인 BagNet을 만들어서 interpretability에 사용한 연구이다.
- 이 모델의 특징은 patch간 spatial relationship을 사용하지 않는다는 것이다.
- 모델의 구조는, backbone으로 patch별 representation을 구한 후, 그것을 linear averaging한 뒤에, linear classifier에 태우는 것이다.
- 이 모델이 interpretable한 이유는, non-linear구조가 없기 때문에 patch-wise representation이 그대로 최종 output에 반영되기 때문이다.
- 이 연구는 현재 DNN모델이 어떤 task를 푸는 과정에서 거의 대부분 local한 정보를 쓰지, spatial relation정보를 쓰지 않는다는 것을 검증하고자 한다.
- DNN모델이 BagNet의 성능과 별 차이가 없기에 여러 실험에서 DNN모델이 BagNet과 비슷한 결과를 보이는 실험을 함으로써 위 가설을 검증한다. 

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- BoF models and DNNs
  - BoF 개념을 이용할 때 image word의 reperesentation을 DNN을 통해 구하려는 시도들이 있었다.
  - 우리 연구는 BoF의 decision-making과 DNN의 decision-making사이의 관계를 밝히려는 첫 시도임.
- Interpretable DNNs
  - 우리 연구는 patch를 자를 때 object단위로 자르는 것이 아니라 random하게 잘랐다.
  - 우리 연구는 image단위의 heatmap을 구하는 것이 아니라 patch-wise heatmap을 구하는 것이기 때문에 각 patch의 기여도를 interpret할 수 있다.
- Scattering network
  - 생략
- region proposal model과의 차이점
  - region proposal model은 image patch내의 정보뿐만 아니라 전체의 정보를 다 이용함.
- interpreting 방법론
  - sapiency map, Int. Gradients, Epsilon-LRP, DeepLIFT
  - heatmap을 계산할 때 모델의 모든 것을 안 상태로 계산함.(<-> BagNet은 그렇지않음.)

**Main Idea**
- 가설: 현재 DNN모델의 decision-making과정이 거의 local key patch를 이용하는 것이며, patch간 spatial relationship은 많이 이용하지 않는다.
- BagNet의 철학: linear averaging, linear classifier를 이용하면 전체 image에서 각 patch가 얼마나 기여하는지를 보일 수 있다.
  - non-linear 컴포넌트가 있으면 중간에 특정 patch의 정보가 누락되어버릴 수 있어서, 최종 image에 대한 각 patch의 기여도를 볼 때 누락되는 정보가 있을 수 있기 때문인듯.
- BagNet Architecture
  - input image를 patch 단위로 자름.(9x9, 17x17, 33x33)
  - CNN backbone을 이용해서 각 patch의 2048차원 representation을 구함.
    - CNN에서 3x3 convolution을 1x1 convolution으로 바꿔서 spatial relation을 사용하는 것을 제한함.
  - patch representation을 class에 대한 1차원 heatmap으로 바꿈. ( np.dot(1x2048, 2048xc) = 1xc )
  - 모든 patch heatmap들을 element-wise 평균내서 전체 image의 heatmap으로 바꿈.
  - 전체 heatmap(=logit)을 linear classifier를 태워서 각 class에 대한 최종 점수를 구함.

**Contributions**
- interpretable model인 BagNet을 제시함.
- BagNet을 통해서, 현재 DNN모델의 decision-making과정이 local key patch를 이용하는 식이라는 것을 검증함.

**Experiments**
- image를 작은 부분으로 나누고 scrambling시켜봄.
  - 가설: spatial relation을 이용하지 않는 모델은 큰 성능 변화가 없을 것임.
  - 결과:
    - BagNet, VGGNet은 성능이 많이 떨어지지 않음.
    - human, ResNet, DenseNet은 성능이 떨어짐.
- image를 여러 부분으로 나누고, 그 중 한 부분씩 가려보며 그때의 logit의 합과, 모든 부분을 동시에 가렸을 때의 logit 결과를 비교함.
  - 가설: image patch사이의 interaction이 거의 없는 모델은 두 실험셋팅에서 큰 차이를 안보일 것임.
  - 결과:
    - VGGNet은 patch 사이의 거리가 30xp이상인 경우, 두 실험셋팅에서 큰 차이가 없다.
- Error Distribution 비교
  - 가설: BagNet과 여러 다른 DNN모델들(VGGNet, ResNet, DenseNet)간의 top-5 error를 비교해서, 틀리는 것이 비슷하다면 서로 비슷한 것을 틀리고, 서로의 decision-making과정이 비슷하다는 것을 보이고싶음.
  - 결과: 모든 DNN모델이 BagNet과 비슷한 것을 틀림.
- Qualitative 분석
  - activation map같은걸 만들어서, 여러 다른 알고리즘들과 비교함. 
  - 가설: 중요한 object에 더 큰 heatmap 점수를 보이면 image에서 key요소를 잘 반영했다는 뜻임.
  - 결과: BagNet의 qualitative performance가 가장 높음.
- BagNet이 예측한 key patch를 위주로 가려보는 실험.
  - BagNet이 에측한 key patch들을 가린 image를 여러 DNN모델에 input으로 줘봄.
  - 가설: BagNet과 decision-making과정이 비슷할수록, spatial relation을 덜 이용할수록 성능이 더 많이 떨어질것임.
  - 결과:
    - BagNet의 성능 하락 폭이 가장 컸음.
    - 더 깊은 모델(ResNet, DenseNet)은 공간적 관계 정보를 더 많이 고려하기 때문에 local patch를 좀 없애도 다른 곳의 정보를 참조할 수 있다. 즉 성능 하락 폭이 VGGNet에 비해 더 적다.
- 그 외 여러 다른 Qualitative 분석 ..

**총평**
- DNN모델의 decision-making과정이 local key patch를 이용하는 식이라는 것을 실험으로만 보였는데, 좀더 직관적, 이론적, 수학적 해석이 있었으면 더 좋았을 것 같다.

## Study

**읽는데 걸린 시간**
- 읽는데 3:45 + 정리하는데 1:00
- pages: 8.5 (without references&appendix)

**비판적 사고(개선점 찾기 / 비판 / 제안 / 향후연구 등)**
- 이 연구는 오직 imagenet에 대한 실험만 진행함. 저자들은 bagnet이 spatial information을 levearge할 수 없는 BagNet이 좋은 성능을 보이므로 imagenet이 쉬운 데이터셋이라고 함. imagenet이 비전분야의 코어 데이터셋은 맞지만, 다른 데이터셋도 많으니, .. 다른 데이터셋에서는 어떤지 분석해주면 좋겠음. 
- BagNet으로 각 데이터셋에 대한 성능을 낸 다음, 어떤 데이터셋에서 잘하고, 어떤 데이터셋에서 못하는지 measure하면 데이터셋들의 특성을 파악하는데 좋을 수도 있음.
- BagNet은 spatial relation을 이용하지 못하는 모델을 밝혀내고 그것들의 decision-making과정을 말해줬는데, ViT같은 모델은 local patch뿐만 아니라 전체적인 정보를 잘 이용해서 성능이 잘나오는 모델인데, 이렇게 spatial relation을 잘 이용하는 모델을 찾아주는 것도 의미있는 연구가 될듯.
- 

**이해못한 점**
- linear classifier가 patch당 분석이 가능해지는데, non-linear classifier를 쓰면 patch당 분석을 못하게 되는 이유?
- BagNet은 heatmap을 계산할 때 모델의 모든 부분을 모른다고 하는 것 같은데, 정확하게 무슨 뜻인지 모르겠다.
- backbone에서 3x3 conv를 1x1 conv로 바꿨다고 했는데, 결국 max pooling단계에서 정보들이 섞이지 않나?
- prototype representation?
- DC component ?
- 
