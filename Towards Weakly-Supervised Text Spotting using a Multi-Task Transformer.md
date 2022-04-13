# Towards Weakly-Supervised Text Spotting using a Multi-Task Transformer
- Author : Yair Kittenplon Inbal Lavi Sharon Fogel Yarin Bar, R. Manmatha Pietro Perona
- arxiv 2022

**Summary**
- Text Spotting 문제를 Weakly-supervised하게 학습시켜 풀 수 있는 방법을 제시한 논문이다. 
- 기존의 Text Spotting 모델들은 학습 data로 bounding box + text 를 둘 다 요구하는 경우가 많았고, bounding box대신 polygon(=segmentation)을 요구하기도 했다. 
하지만 이런 fully-annotated data를 만들기 위해서는 많은 cost가 든다는 점을 지적하고 있다.
- TTS는 fully-annotated로도 학습시킬 수 있고, bouding box는 업속 text만 있는 data를 이용하는 weakly-annotated로도 학습시킬 수 있다.
- Deformable-DETR논문의 모델을 architecture로 사용한다. image를 backbone을 통해 feature map으로 만든 후, transformer encoder-decoder를 활용한 모델을 이용한다.
encoder에서는 input인 feature map을 patch로 만든 후 일자로 펼쳐서 넣는다. decoder에서는 query embedding이라는걸 input으로 넣는데, 각 query embedding의 위치에 해당하는 output이 input image에서 text가 존재하는 곳인 RoI가 된다. 
- Hungarian loss를 기반으로 만든 Text Hungarian Loss로 학습시켰고, TTS의 weakly-supervised 버전을  기존의 fully-supervised model들과 필적한 성능을 냈다.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- Text Spotting 분야에서 어떤 연구가 진행되었는지 설명함.
- Weakly Supervised Approach: Text Spotting domain에서 어떤 weakly-supervised scheme이 사용되었는지 소개함.
- Hungarian Matching: 이 objective를 도입한 연구를 설명함.
  - e.g. DETR

**Main Idea**
- Architecture
  - 1) input Image를 backbone에 넣어서 feature map으로 바꿈. 
    - 어떤 backbone을 사용했는지는 안나와있으나, DETR은 ResNet50을 사용해서 (H_0/32, W_0/32, 2048)차원의 feature map을 만드는 것으로 알고있음.
  - 2) Deformable-DETR에서 사용된 encoder-decoder network를 사용함.
    - 1)의 결과를 patch sequence형태로 만들어서 Encoder에 넣음.
    - Decoder는 input으로 query embedding이라는 것을 받음.
      - query embedding은 단순한 positional encoding임.
      - 이것의 연산 결과가 object(=text)의 위치가 되며, image에서 object가 어디에 있는지를 알려주는 것(detection의 결과)이 됨.
      - query embedding의 개수는 image내의 object(=text) 개수보다 충분히 많게끔 사용자가 직접 설정해줘야하는 요소임.(Human Prior)
    - Decoder는 연산과정에서 Encoder의 output에 주의를 기울임.
  - 3) 3개의 head(detection head, recognition head, segmentation head)를 둬서 각 task를 처리함.
    - detection head, recognition head는 mutual하게 학습시킴. (detection head의 parameter를 updating할 때 recognition head의 loss를 사용함.)
    - detection head는 DETR에서 소개된 3-layer FFN임.
    - recognition head는 LSTM을 이용해서 AR language modeling을 하게 만듦.
    - segmentation head는 우리가 polygon을 출력하고 싶을때만 이용함. 이 head를 학습할 때는 transformer를 freezing시킴.
- Objective
  - Hungarian Matching Loss에서 recognition term을 추가한 형태인 Text Hungarian Loss를 제시함.
  - Hungarian Loss가 무엇인지에 대한 설명은 생략함.
    

**Contributions**
- fully-annotated형태인 box + text가 아니라, text정보만 가지고 학습시킬 수 있는 weakly-supervised training scheme을 제시했다.
- text spotting문제에 transformer기반의 아키텍처를 처음으로 사용했다.
- fully-supervised, weakly-supervised 두 가지 방법으로 모두 학습 가능한 모델이어서 accuracy와 annotation cost간의 trade-off를 이끌어낼 수 있다.

**Experiments**
- Dataset
  - 합성 데이터와 real data를 둘 다 이용했다. (합성된 데이터는 데이터가 풍부하고, real data는 실제 우리가 적용할 대상이기 때문에)
  - SynthText -> 실험 비교군이 되는 TTS_synthetic을 만드는데 사용되었다.
  - Total-Text, ICDAR 2015, ICDAR 2013, COCOText, SCUT. 
- previous works들과의 성능 비교 실험
  - ICDAR2015, Total-Text dataset에서 실험.
  - ICDAR2015의 경우, TextDragon이랑 sota의 개수가 비슷하다. 물론 TextDragon보다 대체적으로 성능이 높긴 하지만..
- Data annotation의 양에 따른 성능 비교 실험.
  - ICDAR2015, Total-Text에서 실험.
  - 결과: 
    - TTS_box의 경우 previous sota보다 확실히 잘함.
    - TTS_weak의 경우 previous sota가 더 잘하긴 하는데, weakly-supervised치고는 잘하는 편.
    - fully supervised버전인 TTS_box보다 previous sota모델이 polygon정보를 더 쓰긴 했는데, TTS_box에다가 polygon정보를 더 추가한다고 해서 성능이 더 오르거나 하지는 않음.
      - 왜냐면 segmentation head를 학습할 때는 transformer를 freeze시키기 때문.
- Rotated data에 대해서도 실험.
  - Rotated ICDAR2013에서 실험.
- Qualitative Analysis
- Ablation study
  - detection head를 학습시킬 때 recognition을 제외해봄.
  - recongnition을 할 때 RNN head가 linear head보다 좋다는 것을 보임.

**총평**
- DETR이 쓰고있던 Hungarian Loss가 weakly-supervised Text Spotting문제에 적용하기에 좋아보인다.

## Study

**읽는데 걸린 시간**
- 읽는데 ?시간 정리하는데 00:30분
- pages: p8 (without references&appendix) 


**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- Q1. #params, inference time, training dataset size와 같은 비교도 있었으면 좋겠다.
- Q2. DETR transformer는 query embedding의 개수를 정해줘야 하므로 어느 정도의 human prior가 필요한 것 같다.
- Q3. loss function에서 최적의 weight를 찾는 것이 그리 trivial하지는 않아보인다. 어떻게 찾은 것인지 나와있었으면 좋겠다.

**이해못한 점**
- Q1. 만약 N=10으로 잡았는데, 실제 object가 5개밖에 없었다. 그러면 no object가 5개 생긴다는건데, 각기 다른 ground truth no object가 5개 더 있는건가?
- Q2. p4에서 polygon output이 필요한 경우에, 우리가 segmentation head로 가게끔 하면 된다는거지? (우리가 optional하게 이용가능한거) <-> dataset의 sample에 따라 polygon으로 annotate되어있는게 있다는건가?
- Q3. Figure 3.에 왜 bounding box를 3개씩 그려놓은거야?
- Q4. Table1. 에서 word spotting과 end-to-end의 차이가 뭐지?
- Q5. lexicon 사용량을 none, full로 구분해놓은게 무슨 뜻인지 잘 모르겟다.
- A5. Dataset의 구성에 따라 다른 표기인듯.
Total-Text는 without lexicon/with lexicon으로 구분하는 듯하고, text정보를 쓰냐 안쓰냐인 듯함.
자세한 의미는 dataset을 다운받아봐야 할 것 같은데, 그럴 필요까진 없을 것 같다.
