# SlowFast Networks for Video Recognition
- Author : Christoph Feichtenhofer, Haoqi Fan, Jitendra Malik, Kaiming He
- ICCV 2019
- 
**Summary**
- semantic한 정보를 처리하는 slow pathway와 motion을 중점적으로 처리하는 fast pathway, 이렇게 2가지의 pathway를 가지는 SlowFast모델을 제시함.
- semantic한 정보는 시간이 많이 흘러도 비교적 적게 변하고 보존되므로 video에 저장된 frame을 띄엄띄엄 sampling하고, 그대신 꼼꼼하게(many channels) 보면 된다는 직관에 근거함.
- motion 정보는 자주 video에 저장된 frame을 자주자주 sampling하고, 그대신 lightweight하게(a few channels) 보면 된다는 직관에 근거함.
- fast pathway, slow pathway의 output을 어떻게 합칠지도 연구함.
- Kinetics-{400,600}, Charardes, AVA dataset에서 평가함.
- 다른 sota모델과의 성능 비교, 다양한 ablation을 통해 자신들의 idea, model이 좋다는 것을 보임.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- retinal ganglion cell 연구에서 영감을 받음.
  - 생체 visual system도 80%는 high temporal frequency에 반응하고 20%는 spatial detail에 반응한다고 함.
  - 이것을 neural network에 적용해보자.
- 기존의 two-stream method와의 차이점
  - 형태만 two-stream이지, 사실 idea적으로는 많이 다름.
  - 우리 모델은 optical flow를 사용하지 않고, end-to-end이며, 
  - slow pathway는 무거운 backbone을 사용하지만 fast pathway에는 경량화된 backbone을 사용(two-stream method는 두 stream에 모두 무거운 backbone을 사용)
- Spatiotemporal filtering
  - HOG3D, cuboid와 같은 filter를 사용하거나,
  - spatial orientation과 temporal orientation을 비슷하게 처리하는 3D ConvNet을 사용함.
  - spatial과 temporal을 다르게 처리하는 방법도 있었긴 한데, 우리것 만큼 thoroughly하게 decomposite하진 않았대.(어떻게 다르지는 설명 안해놨음.)
- Optical flow for video recognition
  - optical flow: deep learning이전에 여러 hand-crafted feature를 이용하는 방법
  - two-stream method: optical flow + another input 이렇게 2가지를 input으로 받는 방법


**Main Idea**
- 

**Contributions**
- 기존의 보편적인 방식과 비슷한 slow pathway와 별도로 high temporal frequency를 capture하는 high pathway를 두었음.
- 다른 연구들에 비해 큰 폭의 성능 향상
- computational cost를 많이 추가하지 않음.
- imagenet pre-training을 쓰지 않음.

**Experiments**
- 기존의 sota모델과의 성능 비교(Kinetics, AVA, Charades)
- Accuracy와 complexity의 tradeoff
  - (1) fast pathway를 추가하고 accuracy와 complexity가 얼마나 증가하는지 봄.
  - (2) T(단위시간당 frame을 sampling하는 양)을 늘리면서 accuracy와 complexity가 얼마나 증가하는지 봄.
  - (3) R50 -> R101로 바꿔보면서 accuracy, complexity가 얼마나 증가하는지 봄.
- SlowFast fusion방법론에 관한 실험
- Channel capacity ratio(fast pathway에서 channel을 얼마나 줄일지)을 바꿔보는 실험
- 현재 fast pathway를 lightweight하고 spatial information을 줄이는 기법으로 channel의 개수를 줄이는데, 이거 말고 다르방법으로 spatial information을 줄일 수 있을지에 대한 실험.
- imagenet pretraining을 시키지 않는 자기들의 training기법에 대한 평가
  - 자기들의 data augmentation, SGD 등등의 방법론
  - 다른 모델에 자기들의 training기법을 적용해봄.
- 기타 여러 덜중요한 실험들 ..

**총평**
- retinal ganglion cell에서 영감을 받았든 안받았든 아이디어는 좋아보임.

## Study

**읽는데 걸린 시간**
- 읽는데 3:15 정리하는데 0:30분
- pages: p8 (without references&appendix) 


**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- Q1. middle pathway를 사용하면 어떨까? Figure 3.에 보면 slow-only -> slowfast로 바꿨을 때 어떤 것이 영향을 많이 받고, 어떤 것이 영향을 적게 받는지 나와있다.
여기서 중간 정도로 성능이 올랐다는 것은, fast pathway와 slow pathway의 중간 정도의 motion적 성격과 semantic적 성격을 가지는게 충분히 있을 수 있다는 것을 의미한다. 
middle pathway를 두면 spatial representation을 적당히 캡쳐하면서 조금 느린 motion을 캡쳐함.(long term motion)
- Q2. 현재는 multi-scale, horizontal flipping augmentation기법을 쓰고 있다. 논문에서는 아주 많은 ablation을 제시했는데도 불구하고 이에 대한 ablation은 없다.
비전 분야에서 data agmentation은 아주 큰 영향을 미치는 요소이다. SimCLR같은 논문에서는 brute-force로 실험을 해봤을 정도니.. 
- Q3. from scratch부터 학습한 데에 대한 이유가 있나? imagenet pretraining을 했을 때 성능변화가 없는 이유에 대한 분석이 나와있지 않은게 아쉽다.
- Q4. randomly crop해도 되는가? temporal orientation방향으로는, 이전에 crop했던 part와 유사한 part에 crop을 해야하는거 아닌가?
- Q5. 실제로 retinal ganglion cell에서 영감을 받았는지, 아니면 결과를 먼저 내놓고 근거를 찾다가 retinal ganglion cell을 알게 되어서 끼워맞춘건지는 모르겠다.

**의문점**
- Table 5-(b)에 beta=1/4~1/8구간에는 왜 증가?

**이해못한 점**
- 거의 다 이해함.
