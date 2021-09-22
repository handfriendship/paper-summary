# Context-Aware Answer Extraction in Question Answering
- Author : Yeon Seonwoo†
, Sang-Woo Lee‡§, Ji-Hoon Kim‡§
, Jung-Woo Ha‡§
, Alice Oh†
- 2020 EMNLP (Long paper)

**Summary**
- End-to-End QA pipeline에서 reader의 성능향상에 대한 연구임. 주어진 passage에서 answer-text가 여러 번 등장할 때 QA reader는 irrelevant context에서 등장하는 answer-text를 
정답으로 가져오는 경우가 많은데, 이것은 QA reader의 성능을 저해시키는 요인이 될 수 있음.
- 최종 answer-span을 학습시키는 것뿐만 아니라, 그것이 적절한 context에서 발췌되었는지도 학습시킴. context를 labeling시켜놓은 dataset이 잘 없는데, 기존 dataset에서 기입된 
ground-truth answer-span으로부터 context에 대한 label을 직접 생성(self-supervised)하여 학습 label로 사용함.
- context를 반영하기 위해 block attention기법을 추가함. 
- 기존 모델에 context를 학습시키고, block attention을 적용하기 위해서는 parameter도 진짜 쪼금(1538개 정도)만 추가하면 되고, training time, inference time도 거의 똑같음. 
여러 baseine 모델들과 비교해 봤을 때, SQuAD1.1, NewsQA, NaturalQA dataset에서 더 잘했고, answer-text가 2번 이상 등장하는 sample들만 모아서 dataset을 만들었더니 baseline model들보다
더 큰 차이로 잘했음.

**Main Idea**
1. soft-labeling method : ground-truth answer-span으로부터 ground-truth context label(probability distribution의 형태)를 정하기 위한 벙법임. 
각 word마다 부여되는 probability는 그 word가 context에 포함될 확률임. answer-span 주위로 window size를 정함. 
answer-span가 context probability는 1.0임. answer-span으로부터 거리에 따라 probability가 지수적으로 떨어지도록 하는 function을 제시했음.
2. block attention : Transformer Encoder를 거쳐 나온 결과(일반적인 reading comprehension를 구한 값)에다가, context안에 속해있는 position의 word들에게는 그만큼의
 가중치(attention score)를 적용하는 방법임. passage의 모든 word에 대해, 어떤 word가 context의 시작지점일 확률과, 끝지점일 확률을 구함. passage의 첫 단어부터 시작지점
 확률을 cumulate해나가고, 끝 단어부터 끝지점 확률을 cumulate해나감. 두 개를 곱한 값이 cumulate값임. 
 
 - Loss function은 context 예측 loss와 answer 예측 loss를 합쳐서 씀.

**Contributions**
- 기존의 dataset(SQuAD1.1)로부터 context에 대한 label(probability distribution형태임)을 구할 수 있는 self-supervised 기법 제안
- answer-span에 대한 context가 passage의 어디에 위치하는지에 대한 probability distribution을 구할 수 있는 block attention 기법 제안
- 제안한 기법을 적용하기 위해, word embedding 차원이 768인 bert-base를 썼을 때를 기준으로 이 모델에 1538개의 parameter만 추가하면 되고, training time 및 inference time도 거의 똑같음.

**Experiments**
- EM과 F1은 answer-span을 평가하기 위한 metric이라서, answer-span이 relevant한 context에서 나왔는지를 평가하기엔 부적절하다. 따라서 우리가 직접 Span-EM, Span-F1라는 metric을 만들어서 평가했다.
1. 종합적인 Reading Comprehension 능력 테스트 : SQuAD1.1, NewsQA, NaturalQA dataset에 대해서 baseline 모델들과 비교. EM, F1, Span-EM, Span-F1 전부 다 BLANC이 더 잘했음. answer-text가 multi-mentioned된 dataset일수록 
다른 SOTA모델과 BLANC의 performance gap 이 더 커졌음.
2. multi-mentioned 환경에서의 테스트 :  dataset의 sample들을 answer-text의 등장횟수에 따라 n=1,2,3,4,5~ 인 5가지 그룹으로 나누어서 테스트. n이 커질수록 baseline모델들과의 performance gap이 더 커짐.
3. zero-shot으로 Supporting sentence prediction 테스트 : HotPotQA를 잘 curate해서 zero-shot 시켰는데, BLANC가 젤 잘했음.
4. Time & Space Complexity 테스트 : bert-base에 우리의 self-supervised method + block attention을 추가해봤는데,
 추가된 파라미터 개수는 negligible한 수준이고, training time, inference time도 거의 똑같음.

**총평**
- 최근 다루는 모델들이 너무 거대해져서, 대학원생 수준의 GPU로 모델을 training, testing하는게 너무 어려워보인다.

**질문or향후 연구방향 제시(교수님 랩면접 대비)**
- Q1. 논문에서 제안된 attention대신 multi-head attention을 써보면 어떨까요?
- Q2. 연구에 사용된 computational resources가 적다는 이유로 epoch=3으로 BLANC와 기존 baseline 모델들을 학습시켰는데, 너무 적은것 같아요. training time(epoch)을 늘리면 baseline이 더 잘 할 수도
있지 않을까요 ?
- Q3. 일반적으로 context probability distribution을 구할 때 passage의 중간쯤에서 context일 확률이 높게 나오는 문제는 발생하지 않나요? 

## Study

**읽는데 걸린 시간**
- 읽는데 4시간 + 정리하는데 1시간
- 예전에 한 번 읽었던 논문임.

**알게 된 지식**
- -

**아직까지 모르는 부분**
- 논문에서 나오는 answer-span, answer-text이란 용어가 아직까지 정확히 무엇을 지칭하는지 모르겠다.  
