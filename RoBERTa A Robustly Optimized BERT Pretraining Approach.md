# RoBERTa: A Robustly Optimized BERT Pretraining Approach
- Author : Yinhan Liu, Myle Ott, Naman Goyal, Jingfei Du, Mandar Joshi, Danqi Chen, Omer Levy, Mike Lewis, Luke Zettlemoyer, Veselin Stoyanov
- 2019 arxiv

**Summary**
- 기존의 BERT를 더 많은 데이터셋으로 더 오래, 그리고 여러 hyperparameter들 및 input 요소들을 바꾸어 실험했더니 더 좋은 성능을 낼 수 있었다는 논문
- 이 논문의 교훈은, 많은 연구들이 architecture나 objective를 개선하는데 치안이 되어있는데 반해, dataset, training strategy와 같은 mundane한 요소들을 신경써서 모델을 제대로 학습시키는 것도 중요하다는 것임.
- 더 많은 데이터, 더 오래 학습, masking 기법, 더 큰 batch size, NSP loss를 없앰, pretraining시 input format을 바꿈, byte-level BPE, 기타 여러 hyperparameter 수정.
- GLUE, SQuADv1.1/v2.0, RACE에서 좋은 성능을 보임.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- 

**Main Idea**
- BERT가 만들어질 때 사용됬던 여러가지 setting들을 바꿔봄.
- 내가 생각하는 중요도 순서로 나열
- 더 큰 데이터셋
  - BERT는 16GB의 dataset을 사용. RoBERTa는 160GB의 dataset을 사용.
  - (기존) BookCorpus, Wikipedia + (추가) CC-NEWS, OpenWebText, Stories
  - training iteration은 동일하게 두고 dataset만 늘렸을 때도 성능 향상.
- 더 오래동안 학습
  - 기존에 100K step동안 학습시켰던 것을 300K, 500K로 늘려서 학습시킴.
  - step size를 늘릴수록 성능 향상.
  - batch size는 밑에 실험에서 찾았던 8K로 뒀음.
  - 500K까지 늘렸음에도 아직도 saturate되지 않았고, 더 길게 학습시키면 성능 향상이 더 있을 것이라고 주장함.
- 더 큰 배치사이즈
  - \# of passes (training iteration)은 동일하게 둔 채로 batch size를 늘려봄.
  - (batch_size, steps)를 (256, 1M), (2K, 125K), (8K, 31K)로 실험해봄. 
  - (2K, 125K)일 때가 가장 좋았음.
  - 하지만 향후 다른 실험에서는 (8K, 31K)를 사용함. 이유는 batch size가 커질수록 분산 병렬 학습이 더 쉬워진대. (gradient accumulation 때문에)
- Input format & NSP loss
  - NSP loss가 꼭 필요한지 의문을 제기하여 빼봄.
  - input format을 4가지 유형으로 실험함.
  - (1) Segment pair: 기존 BERT가 쓰던 유형. 2개의 segment를 [SEP]토큰을 사이에 두고 이어붙인 후, 같은 document에서 왔는지를 맞추는 것. 각 segment는 하나 이상의 문장일 수 있음.
  - (2) Sentence pair: 각 segment는 하나의 문장으로 이루어짐.
  - (3) Full sentence: NSP loss를 없앰. input을 최대한 512에 가깝게 만듦. 한 doucment에서 sequence를 가져오는데, 만약 document가 중간에 끝나버리면, [SEP]토큰을 두고 다른 document에서 sequence를 마저 가져옴.
  - (4) Doc sentence: NSP loss를 없앰. input을 최대한 512에 가깝게 만듦. 한 docuent에서만 sequence를 가져옴. 만약 document가 중간에 끝나버리면 끝난대로 짧은 input을 씀.
  - 4번 결과가 가장 좋았음. 그래도 다른 model과의 비교를 위해 3번을 사용.
- Dynamic masking
  - 기존 BERT는 preprocessing time에 미리 masking을 해둠.
  - input 하나당 문장을 10가지 다른 유형으로 masking함.
  - 40 epoch동안 학습됨.
  - RoBERTa에서는 매번 다른 방식으로 dynamically하게 masking을 함.
  - dynamic masking이 더 좋은 성능을 보임.
- Text Encoding Method
  - 기존에는 character-level BPE가 사용됨.
  - 하지만 이것은 단일 character가 너무 많은 size를 차지한다는 문제가 있음.
  - GPT2에서 소개된 byte-level BPE는 size가 적당하면서도 많은 subword를 잘 표현한다고 생각했음.
  - 이거 때문에 base 모델의 경우 15M, large모델의 경우 20M개의 parameter가 더 사용됨.
  - byte-level BPE의 성능이 약간 안좋았다는데, 향후 실험에서는 이것을 씀.
- 기타 hyperparameter setting
  - Adam의 beta value, epsilon
  - weight decay
  - learning rate, warm-up step
  - dropout

**Contributions**
- 더 많은 데이터, 더 오래 학습, masking 기법, 더 큰 batch size, NSP loss를 없앰, pretraining시 input format을 바꿈, byte-level BPE, 기타 여러 hyperparameter 수정 했더니 성능이 올랐다.
- dataset, training strategy와 같은 mundane한 요소들도 중요하다는 question을 던졌다.

**Experiments**
- GLUE
  - multi-task fine-tuning을 하지 않고, 평가할 downstream task을 대상으로 single-task fine-tuning만 수행함.
  - 성능이 잘나옴.
- SQuAD
  - 다른 model(e.g. XLNet)은 다른 QA dataset을 가져와서 downstream task를 풀 때 사용했는데, 우리는 SQuAD만 사용힘.
  - 성능이 잘나옴.
  - XLNet을 기반으로 한 어떤 sota모델 하나는 못이겼음.
- RACE
  - 데이터셋 설명: 중국의 중,고등학생이 푸는 영어 독해 4지 선다 문제.
  - 각 4지 선다 보기를 (question+passage, candidate)로 묶음.
  - RoBERTa를 4개 두고, 각 4지 선다 보기에 대한 [CLS] representation을 구함.
  - 4개의 [CLS] representation을 FC-layer에 넣어서 최종 답을 고름.
  - 성능이 잘나옴.

**총평**
- 실제 Kaggle이나 Project를 할 때 BERT를 사용할 일이 많을 것이다. 그 때 어떤 요소나 hyperparameter가 BERT의 성능 향상에 영향을 미치는지 알 수 있었다.

## Study

**읽는데 걸린 시간**
- 읽는데 ?:00 + 정리하는데 1시간
- pages: 9

**비판적 사고(개선점 찾기 / 비판 / 제안 / 향후연구 등)**
- Q1. development set accuracy를 산출하는 것이 공정성에 문제가 없는가? (튜닝 횟수에 따라서 dev set에 과적합할 위험이 있지 않나?) dev-set으로 평가를 했다는게 무슨 의미인지 모르겠다.
- Q2. batch size를 2K로 두는 것이 더 좋은데, 8K로 둬도 되는 것인가?
- Q3. byte-level BPE를 최종적으로 쓴 것 같은데, 성능이 안나왔는데도 써도 되나?
- Q4. XLNet은 8K 125K 정도인데, 비슷한 # of passes를 가진 RoBERTa와 비교했을 때는 XLNet이 더 좋긴 함. 

**이해못한 점**
- GLUE의 QNLI 실험 셋팅, GLUE의 WNLI 실험 셋팅.
- gradient accumulation
- GPT2 byte-level BPE
- ensemble model이 뭐지?
- byte-level BPE를 쓰는 것이 BERT-base와 BERT-large에서의 차이를 일으키는 이유?
