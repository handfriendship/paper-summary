# Robust Contrastive Learning Using Negative Samples with Diminished Semantics
- Author: Songwei Ge, Shlok Mishra, Haohan Wang, Chun-Liang Li, David Jacobs
- NeurIPS 2021

## 세가지가 꼭 들어가있어야 함.
- 어떤 것을 풀려고 했는지(Summary)
- 기존에는 무슨 문제가 있는지, 기존의 방법에 비해 우리가 왜 더 좋은지(Related Works)
- 어떻게 풀었는지(Main)

**Summary**
- 어떤 것을 풀려고 했는지: 피상적인 요소에 bias되지 않고 semantic에 더 집중하는 강건한 CNN모델을 만들고 싶다.
- CL에서 patch-based, texture-based되어있는 negative sample을 생성해서 모델이 그 부분에 대한 short-cut이 생기지 않게 한다. 그게 결국 OOD에 도움이 된다.
- CNN model은 texture-shape trade-off가 있는데, texture-biased되면 shortcut을 보고 배우는 것이고, shape-biased가 되면 더 semantic을 보고 평가하는 거라 robust해진다.
- CL에서 negative sample을 만들 때 semantic feature는 다 빼버리고 superfluous요소만 가지고 있게끔 만들어보자.
  - 반면 positive sample은 semantic feature를 담고 있도록 만들고.
  - 이렇게 하면 모델은 superfluous 요소는 배우지 않고 semantic feature만을 학습하도록 만들어진다.
- 많은 실험들을 통해 CNN model이 어떻게 동작하는지 제시했다.

**Related Works(기존의 방법론, 기존에는 무슨 문제가 있는지, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- Contrastive Learning
  - 기존의 방법론들은 
  - 1) 모든 shortcut을 피하고 semantic만 담은 positive sample만을 만들기는 어렵지만 semantic을 파괴하고 shortcut만을 담은 negative sample을 만들기는 쉬우니, 그것을 neg sample로 두고 CL을 해보자.
  - 2) 기존의 CL로 유명한 SimCLR, MoCo는 negative sample을 그냥 random하게 sampling한다는 점과, 이것을 위해 너무 큰 batch_size/memory size가 필요해서 expensive computation문제가 있다.
    - 최근에는 hard negative를 활용하려는 움직임이 대세이다.
    - 최근 관련 연구:
      - hard negative에는 더 많은 weight를 부여한다.
      - 가장 hardest negative 두개의 linear combination을 해서 hard negative sample을 synthesizing한다.
      - adversarial하게 positive, negative image를 만들어서 활용한다.
      - foreground, background를 조작해서 pos, neg image를 만든다.
    - 우리 연구가 최근 관련 연구와 다른점: 우리는 OOD문제에 더 집중한다.
- Robustness, OOD problem
  - CL을 이용한 OOD연구는 거의 없다.
  - CNN은 많은 superfluous feature에 bias되어있다. 그 중에서 texture-shape trade-off문제가 최근에 가장 많이 논의되고 서서히 consensus가 만들어지고 있으니, 이것을 debias시키면 
더 OOD robust 해질것이다라는 가정을 갖고 이것을 CL셋팅으로 풀어보겠다.

**Main Idea**
- 어떻게 풀었는지: 우리가 bias되고싶지 않은 요소를 negative sample에 담아서 contrastive learning을 했다.
- 1) Contrastive Learning을 어떻게 적용했는지
  - 두 가지 방향으로 negative sample을 생성해본다: texture-based, patch-based
  - 기존 방법(MoCo, BYOL, etc..)에서 사용한 negative sample에다가 우리 방법으로 만든 negative sample을 추가한다. 우리 방법껀 weight를 더 준다.
  - training dataset의 각 image sample에 대해 query image를 만들고, query에 대한 semantic 정보가 보존되도록 positive image를 만든다. 
- 2) texture-based negative sample을 어떻게 만들었고, 어떤 것을 알아보려고 썼는지
  - 2-1. query image에서 두 가지 patch를 crop한다. 하나는 image의 중앙부로부터, 다른 하나는 image의 random한 부분으로부터.
  - 2-2. texture-based image를 만드는 툴로 texture-based negative sample을 만든다.
- 3) patch-based negative sample을 어떻게 만들었고, 어떤 것을 알아보려고 썼는지
  - 3-1. 224x224 negative image를 만들꺼다. 
  - 3-2. patch_size=d 이고 d는 매 iteration마다 변한다. d가 작으면 patch개수가 많을꺼고, d가 크면 patch개수가 적을꺼다. 매 iteration마다 다른 negative sample이 만들어진다

**Contributions**
- CL을 통해 local-feature, texture에 bias되지 않도록 하는 방법을 제시했다.
- CL을 통해 OOD문제를 푸는 방법을 제시했다.
- ImageNet-Texture 데이터셋을 제시했다.
- shape-texture trade-off에 대한 많은 실험을 통해 분석을 제공했다.

**Experiments**
- patch-based, texture-based negative sample이랑 query sample이랑 similarity가 일반 negative sample보다 더 높다. 따라서 이 둘은 hard negative임.
- Extension to other non-semantic features
  - 보통 CNN model은 color distribution에 bias되는 경우가 많다. 
  - patch-based neg sample을 쓴다고 해서 color distribution이 변하는것도 아닌데, color distribution shortcut이 예방되더라.
  - Q. 이것에 대한 직관적인 해석이 뭐냐?
- 보통은 memory-bank size가 줄면 성능이 하락한다.
  - 하지만 patch-based negative sample을 사용한 경우, memory-bank사이즈와는 상관없거나, 사이즈가 줄면 오히려 성능이 증가했다. (왜냐면 patch-based neg sample에 대한 가중치가 높아지기 때문.)

**총평**
- Contrastive Learning을 어떻게 구현해야 하는지에 대해 많은 생각을 하게 해 준 논문
- CL을 쓸 때 negative sample을 뭘 쓰냐에 따라 모델이 어떤 점에 강건해질 지가 정해지고, neg sample을 너무 많이 쓰면 computational issue도 생긴다.

## Study

**읽는데 걸린 시간**
- 읽는데 5~6시간 정리하는데 1:52분
- pages: p8 (without references&appendix) 

**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- 아이디어1. 이 메소드를 적용했을 때 OOD에 더 적용이 잘되는 이유는, 피상적인 bias를 보는게 아니라 semantic에 집중하도록 모델을 학습하기 때문에 더 잘된다는 거지?
	-> 내가 DG에 contrastive learning을 적용하려면 정말 어떤 hard negative를 골라야할지를 신중하고, 근거를 가지고 고민해봐야할듯.
	-> 예를 들면 domain을 잘 구분하도록 하는 방향으로 학습시킨 adversarial image를 hard negative로 학습시킨다던지 .. (근데 class-level로 적용되어야함.)

**이해못한 점**
- Q1. shape-texture tradeoff?
  - Human은 shape에 더 bias되는 반면, CNN model은 texture에 더 bias된다. texture는 피상적인 요소인 반면에, shape은 semantic한 요소라서 shape을 보고 판단할수록 더 robust한 모델을 만들 수 있다.
- Q2. alpha가 커지는게 왜 shape-texture trade-off이지?
  - alpha를 키우면 texture-biased neg sample을 더 많이 penalize한다는 거기 때문.
- Q3. class label을 쓰는건가? 안쓰는건가?
  - ??
- Q4. patch-based실험이랑 shape-texture trade-off 실험이 각각 무엇을 말하고자 하는지에 대해 완벽하게 머릿속에서 정리가 안되는 느낌이다. 
  - patch-based vs texture-based는 두 가지중 patch-based의 성능이 더 좋아서 이걸 가지고 모델 성능평가를 했다는 것이고,
  - shape-texture trade-off는 neg sample에 대한 weight인 alpha를 키웠을 때 모델이 더 shape-based된다는 것이다.(여기서 neg sample은 texture-based를 썼다는 거겟지?)
- input image로부터 추출된 두 개의 patch로 texture-biased image를 만든다는데 어떻게 하는건가?
- Eq(2)에서 식 전개가 이해안된다.
- ImageNet-Sketch를 curate할 때 google검색에서 아무거나 가져와도 되는가?


**알게된 점**
- Calibrated의 뜻: model의 confidentiality를 조절한다는 뜻인 듯
- 데이터셋
  - original ImageNet: 14M samples, 22K classes
  - ImageNet: 통상적으로 imagenet-1k를 말하는 것임. 1M samples, 1K classes
    - (https://datascience.stackexchange.com/questions/47458/what-is-the-difference-between-imagenet-and-imagenet1k-how-to-download-it)
  - ImageNet-Sketch: 50000 samples, 1K classes, each class has 50 samples, collected from the query 'sketch of _____'
    - (https://www.kaggle.com/wanghaohan/imagenetsketch)
  - ImageNet-100: subset of ImageNet-1K. 100 classes, each class has 1300 samples for training, 50 for validation
    - (https://www.kaggle.com/datasets/ambityga/imagenet100)
  - ImageNet-C(orruption): ImageNet에 15개의 corruption 타입을 적용. 각 타입마다 5-level severity가 존재함. 
  - Stylized-ImageNet: texture 정보를 제거한 imagenet. (biased towards texture논문에서 나온 것)
  - ImageNet-R(endition): 각 class마다 여러버전의 rendition image를 준비한 것(art, cartoons, deviantart, graffiti, embroidery, graphics, origami, paintings, patterns, plastic objects, plush objects, sculptures, sketches, tattoos, toys, and video game). 30,000 images, 200 classes(subset space of imagenet) 
