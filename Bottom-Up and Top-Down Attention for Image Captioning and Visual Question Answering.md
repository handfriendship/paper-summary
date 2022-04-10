# Bottom-Up and Top-Down Attention for Image Captioning and Visual Question Answering
- Author: Peter Anderson, Xiaodong He, Chris Buehler, Damien Teney, Mark Johnson, Stephen Gould, Lei Zhang
- CVPR 2018

**Summary**
- image captioning과 VQA를 할 때 bottom-up정보와 top-down 정보를 이용하는 것을 말한다.
- top-down: 주어진 문제, 사람 입장에서 주어진 문제를 풀기 위해 참고하는 외부 정보(image captioning에서는 지금까지 생성된 caption, VQA에서는 question)를 leverage한다는 뜻이다.
- bottom-up: image를 CNN backbone으로 처리한 region proposal에 대한 features를 leverage하는 것을 말한다.
- top-down정보를 이용해 RoI features들의 attention score를 구하고, 그것을 이용해 RoI features를 convex combination한다.
- image captioning(MS COCO), VQA(VQAv2.0)를 이용해 여러 환경을 구축해서 평가를 했는데, 기존 모델들의 성능을 outperform했다.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- bottom-up top-down(ours)의 motivation: human이 문제를 풀 때 가지는 자연스러운 접근방법(grid feature가 아니라 bottom-up정보를 이용하는것)을 따른다.
- one or more layers of a CNN의 output에서 구한 region들에 대해 top-down으로부터 구한 attention score를 적용하는 방법.
- 반면에, 특정 object에 대해 attention score를 구하는 방법들은 많이 연구되지 않음.
  - selective search를 이용한 방법 -> hand-crafted feature를 이용하는 방법.
  - edbe box나 spatial transformer network를 이용하는 방법 -> differentiable region proposal(딱 하나로 떨어지는 region proposal을 말하는듯? <-> 반면에 faster r-cnn은 bouding box가 관심 물체에 대해 여러 scale로 생김.)  

**Main Idea**
- Bottom-up feature extractor
  - ImageNet pretrained된 ResNet을 이용해 Faster R-CNN을 initialize한 후 이용함.
  - pretraining objective
    - RPN과 object classifier 각각에 대해 bounding box regression, object classification문제를 둘 다 적용해서 품. (총 4개의 objective가 쓰이는 것)
    - attribute를 맞추도록 하는 task도 같이 품.
- Captioning Model
  - 첫번째 LSTM
    - 아래 input에 들어가는 3가지 정보를 모두 이용하기 위함.
    - input: [caption generation LSTM의 이전 step의 hidden state;mean pooled bottom-up features;지금까지 생성된 caption의 embedding vectors]
    - output: bottom-up features의 attention score를 구하기 위한 재료가 되는 hidden states.
  - 두번째 LSTM:
    - caption generation을 하기 위한 Langauge Model
    - input: [attention weighting이 적용된 bottom-up features;1st LSTM의 hidden states]
    - output: generated caption
  - attention layer: bottom-up features의 attention score를 구하기 위함
  - pretraining objective
    - 1) autoregressive teacher forcing objective
    - 2) CIDEr: 모델이 뱉은 token들의 score를 구해서 expectation시킨 것. (SCST논문에서 CIDEr로 학습했을 때의 gradient를 approximation하는 방법을 제시했는데, 그 방법을 따라서 계산했음.)
    - 1), 2)를 섞는게 아니라, 두 가지 loss로 모델을 학습시켜 평가해봄.
- VQA Model
  - Question을 처리하는 module: word embedding을 구한 뒤, GRU의 마지막 hidden state를 question단위의 representation으로 생각함.
  - attention score를 구하는 module:
    - question vector와 bottom-up features를 가지고 gated tanh를 이용해 구함.
  - attention weighting이 적용된 bototm-up feature와 question vector를 hadamard product한 뒤, answer를 출력함.
  - data augmentation: 
    - Visual Genome을 이용함.
    - 구체적으로, visual genome으로 bottom-up model을 pretraining시킨다. 이때 사용된 VQA model의 answer vocabulary에 등장하는 VQAv2.0의 sample들만 fine-tuning하는데 사용한다.
- 기타
  - 이 논문은 captioning model과 VQA model은 별도로 pretraining하지 않는 것처럼 보인다.
  
**Contributions**
- 기존의 top-down정보 + grid feature를 top-down정보 + bottom-up정보를 이용하는 것으로 바꿨다.

**Experiments**
- Pretraining bottom-up attention model
  - Visual Genome Dataset을 이용.
- pretraining VQA model
  - ??
- finetuning captioning model
  - MSCOCO를 이용
- finetuinng VQA model
  - VQAv2.0를 이용
- baseline으로 사용하기 위해, 자신들의 architecture와 실험환경에 그대로 따르면서 bottom-up model대신 resnet을 사용한 모델을 제시해서 성능을 비교한다.
- 한 모델을 평가할 때는 Cross-Entropy loss, CIDEr objective 각각으로 optimize시켜 둘을 모두 평가한다.
- bottom-up feature의 output size로 무엇이 적당한지를 ResNet으로 찾음.
- MSCOCO Karpathy test split, MSCOCO test server, VQAv2.0 test server에서 평가.
- Qualitative Analysis

**총평**
- 

## Study

**읽는데 걸린 시간**
- 읽는데 ?시간 정리하는데 1:05분
- pages: p8 (without references&appendix) 


**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- vision-language task를 이용해 학습된 word embedding으로 word lookup table을 initialize시키면 안되나?
- attribute class를 예측할 때 multi-label을 사용하도록 학습하면 안되나? attribute class label을 정할 때, hierarchy가 있는 class에 대해 merge/remove하지 않았다고 했다.
이런걸 잘 grouping하면 multi-label을 만들 수 있을 것 같은데?

**이해못한 점**
- data augmentation을 할 때, model's answer vocabulary가 어떤 dataset으로부터 만들어진 것인지, visual genome dataset이랑은 어떤 연관을 가지는지 모르겠다.
