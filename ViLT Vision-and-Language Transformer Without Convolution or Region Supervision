# ViLT: Vision-and-Language Transformer Without Convolution or Region Supervision
- Author: Wonjae Kim, Bokyung Son, Ildoo Kim
- ICML 2021

**Summary**
- VLP의 visual embedder를 single patch projection layer로 대체하여 훨씬 가볍고 연산량을 줄이고 inference time을 줄이고 성능을 보존한 모델.
- 보통 visual embedder, text embedder를 통해 image, text를 각각 embedding시키고 modality interaction module에 보내는데, 대부분의 연구들에서 visual embedder가 다른 module들에 비해 압도적으로 크고 무겁다는 단점이 있었ㅇ므.
- visual embedder을 ViT에서 소개된 patch projection layer를 이용해 아주 가볍게 처리함.
- modality interaction module로는 transformer를 사용하고, image-text-match, IOPT, word-patch alignment, MLM 등을 사용해서 학습시킴.
- 크게 classification, retrieval문제를 downstream task로 풀었는데, FLOPs, inference time을 훨씬 줄였고, 그러면서도 성능이 조금만 떨어졌다는 것을 보임.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- 1) embedding modules, modality interaction module의 크기에 따른 분류
  - 1-1. VE > TE > MI
  - 1-2. VE = TE > MI
  - 1-3. VE > MI > TE
  - 1-4. MI > VE = TE
- 2) Modality Interaction Schema에 따른 분류
  - 2-1. single-stream: visual token과 text token을 concat해서 하나의 modality interaction module에 넣음. 두 modality간의 self-attention이 가능.
    - e.g. ViLT, UNITER
  - 2.2. dual-stream: visual token과 text token을 처리하는 modality interaction module이 두 개가 있고, 두 module이 서로 상호작용.
    - e.g. ViLBERT
- 3) Visual Embedding Schema에 따른 분류
  - Region Feature를 사용(Faster R-CNN 등). e.g. ??
  - Grid feature를 사용. e.g. X-LXMERT, Pixel-BERT, VQA-specific models ...
  - 둘 다 사용안함. e.g. ViLT

**Main Idea**
- Architecture
  - image는 patch projection layer를, text는 from scratch부터 학습시키는 embedding layer를 이용해서 embedding을 구함.
  - positional encoding을 더해줌.
  - 두 modality의 embedding을 concat해서 transformer에 넣음. 
  - layer normalization이 먼저 오도록 하는 ViT의 transformer구조를 차용.
- pretraining objective
  - Image Text Match(ITM): 50%의 확률로 (image, text)의 positive or negative match를 넣은 후 binary classificaiton하는 문제.
  - Word Patch Alignment: 
    - modality interaction module의 output으로 나온 text representation들과 patch representation의 kl-divergence가 가까워지도록 학습시키는 방법.
    - IPOT를 이용.
    - label이 없이도 image patch들과 text들을 align시킬 수 있음.
  - Masked Language Modeling: text에 대해서만 적용(그대신 text의 self-attention과정에서 image에 attention은 되겠지.)
  - (Masked Patch Prediction)
    - image patch중 15%를 masking하고, RGB value의 mean을 맞추도록 학습.
    - 이건 넣으니깐 성능이 더 떨어져서 뺐다고 함.
- data augmentation
  - RandAugmentation을 사용.
    
**Contributions**
- 무거운 CNN-based visual embedder를 patch projection layer로 대체하여 성능을 보존하면서도 가볍게 만듦.
- CNN-based visual embedder를 없애서, 기존의 modality interaction module의 잠재력이 visual vocabulary에 갇히지 않도록 함.

**Experiments**
- pre-training으로 4개의 dataset을 사용했다.(MSCOCO, Visual Genome, SBU Captions, Google Conceptual Captions)
- downstream task로는 크게 classification, retrieval을 수행.
- classification: VQAv2, NLVR2
  - VQAv2: 원래 question answering 문제긴 한데, 이것의 answer들을 3129개의 answer class들로 묶어서 
분류 문제로 바꿀 수가 있다.
  - NLVR2: 두 개의 image와 하나의 question text를 입력으로 받는 binary image classification 문제. (question, image1), (question, image2)로 나눠서 
- retrieval: MSCOCO, Flickr30K
  - image-to-text, text-to-image task라고 보면 됨.
  - ITM head를 이용해서 계산함. positive sample(image와 retrieval되어야 할 text), 15 negative samples(image와 random sampling된 text)을 넣고 cross-entropying.
  - retrieval task들은 1)zero-shot, 2)fine-tuning 두 가지 버전으로 실험을 함.
  - zero-shot에서 잘 된다는 것은 좋은 representation을 얻었다고 볼 수 있음.
- ablation study
  - 학습시간을 더 오래 학습하도록 늘려봄.(ViLT는 더 가벼우니깐 늘리기가 쉬움.)
  - RandAugmentation를 빼봄.
  - MPP objective를 빼봄.
  - whole word masking을 빼봄.
- 모델의 크기, 속도에 대한 실험.
  - #Params, #FLOPs, Time을 다른 모델과 비교.
- Visualization
  - word-patch alignment, IPOT가 잘 되었는지에 대한 qualitative analysis
  
**향후연구**
  - Scalability: ViLT는 BERT처럼 모델을 scalable할 수 있는 구조인데, 나중에 VLP dataset이 더 생기면 scalable해보자.
  - Masked Modeling for Visual Inputs: image에 대해 MLM을 적용할 수 없을까?
    - grid RoI기반을 이용해서 masked modeling을 하되, region proposals를 처음에 한번만 clustering하는것이 아니라, 학습시켜나가면 어떨까?
  - Augmentation strategies: gaussian blur를 적용해보자.

**총평**
- VLP쪽을 하다보니 실험이 어려워서 이런 motivation을 가지게 되어서 이런 research를 수행하게 되지 않았나 싶음.

## Study

**읽는데 걸린 시간**
- 읽는데 ?시간 정리하는데 1:05분
- pages: p9 (without references&appendix) 


**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- image text matching때 좀 더 hard negative를 써보면 어떨까?
- 그래도 region feature의 장점이 있을 것 같은데 .. 아주 가벼운 region feature extractor를 써보는건 어떨까?
- pre-training때 사용했던 dataset으로 downstream task의 성능 평가를 해도 되나?

**이해못한 점**
- visual embedder가 크고 무겁다는 것은 알겠는데, 논문에서는 이걸 bottleneck이라는 단어로 표현하고 있음. 이 단어가 완전히 와닫지는 않네.
