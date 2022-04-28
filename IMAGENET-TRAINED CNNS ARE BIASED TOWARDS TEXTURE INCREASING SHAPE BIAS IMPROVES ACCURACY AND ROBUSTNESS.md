# IMAGENET-TRAINED CNNS ARE BIASED TOWARDS TEXTURE; INCREASING SHAPE BIAS IMPROVES ACCURACY AND ROBUSTNESS
- Author: Pan Li1, Da Li2,3, Wei Li, Shaogang Gong, Yanwei Fu4, Timothy M. Hospedales
- ICLR 2019

## 세가지가 꼭 들어가있어야 함.
- 어떤 것을 풀려고 했는지(Summary)
- 기존에는 무슨 문제가 있는지, 기존의 방법에 비해 우리가 왜 더 좋은지(Related Works)
- 어떻게 풀었는지(Main)

**Summary**
- CNN의 의사결정과정이 human과 일치하는지를 보고자 했음. 
- texture에 더 bias된다고 분석한 뒤 human과 일치하게끔 만들어줘서 많은 task에서 더 robust하게 동작하는 CNN모델을 만들고자 했음.
- texture, shape중 어디에 더 bias되는지 검증하기 위해서 cue-conflict sample을 이용함.
- human의 의사결정 과정(shape-bias)을 가지도록 학습시키기 위해 Stylized-ImageNet을 만들고 사용해서 training함.
- shape-bias를 가지도록 학습시키는게 여러 vision task에서 실제로 성능향상을 가져옴을 보임.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- 기존에는 무슨 문제가 있나요?
  - 기존에는 CNN이 texture-bias라고 설명하는 연구들도 있었고, shape-bias라고 설명하는 연구들도 있었음.
- 기존의 방법에 비해 왜 더 좋은가요?
  - 이것에 대한 명시적인 설명은 논문에 없음.
  - 기존의 방법과는 다른 새로운 실험 방법임.

**Main Idea**
- 1) conflict cue sample이라는 것을 만들어서 shape, texture중 어디에 더 correlate되는지 알 수 있도록 하는 실험을 구성하여 human과 CNN의 결과를 비교함.
- 2) CNN을 stylized-ImageNet에 training시켜 shape-bias를 가질 수 있도록 함.
- 3) 다양한 셋팅에서 실험해서 실제로 shape-bias를 가지는 것이 도움이 됨을 보임.

**Contributions**
- CNN은 shape이 아니라 texture에 bias됨을 여러 실험으로 밝힘.
- Stylized-ImageNet을 제시함.
- CNN이 shape-bias를 가지도록 학습시키면 실제로 성능향상이 됨을 보임.

**Experiments**
- Texture vs Shape bias in humans and imageet-pretrained CNNs
  - 목적: human과 CNN이 각각 texture와 shape중 어디에 더 bias되는지 알고싶음.
  - 가설: human은 shape / CNN은 texture
  - 실험방법:
    - Cue-Conflict sample을 풀게 해보고 어느쪽으로 맞춘는지 봄.
  - 결과:
    - CNN models는 texture쪽으로, human은 shape쪽으로 맞춤.
- Overcoming the texture bias of CNNs
  - 목적: CNN이 texture에 bias안되도록 학습시키고 싶었음.
  - 가설: Stylized ImageNet는 random texture를 입힌 것이니, 거기에 training시키면 texture에 bias하지 않을 것이다.
  - 실험방법:
    - Stylized-ImageNet에 training된 ResNet-50이 Conflict-Cue sample을 풀게끔 해봄.
    - Stylized-ImagenNet이 texture-biased되어있지 않다는 것을 검증하기 위해 BagNet을 Stylized-ImageNet에 training시켜봄.
  - 결과:
    - Conflict-Cue sample을 풀게했을 때 shape쪽으로 맞춤.
    - Stylized-ImageNet에 trained된 BagNet의 성능이 처참한 것을 보아, Stylized-ImageNet은 texture-biased되어있지 않음.
- Robustness ...
  - 목적: shape-ResNet이 실제로 성능 향상을 가져오는지, 여러 task에 robust한지 보고싶음.
  - 가설: CNN을 shape-biased시키면 일반적인 데이터셋(imagenet) 성능도 오를 것이고, shape representation은 human insight와 동일하니 여러 vision task에 robust할 것이다.
  - 실험방법:
    - imagenet에서 평가.
    - Object Detection task를 풀게해봄.
    - 여러 타입의 noise에 robust한지 봄.
  - 결과:
    - good
    - object detection(Pascal VOC)잘함.
      - object detection의 ground truth bounding box가 애초에 global object shape를 에워싸도록 만들어지기 때문
    - 여러 noise에 강건함.
      - low-pass filtering, blurring에는 약함.(shape-ResNet이 sharp edge나 high frequency signal을 over-represent하는 경향이 있기 때문.)

**총평**
- ...

## Study

**읽는데 걸린 시간**
- 읽는데 3:00시간 정리하는데 0:30분
- pages: p8 (without references&appendix) 

**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- ...

**이해안되는 점**
- dataset이 문제라고 했는데, imagenet이 아니라 OpenImageV2에서 학습시켰을 때도 texture에 bias되는 이유는 뭐냐?
- Section 2.2의 dataset 설명파트에서 5개 유형의 데이터셋 각각의 개수가 일치하지 않는데? 
