# Speaker Sensitive Response Evaluation Model
- Author : JinYeong Bak, Alice Oh
- 2020 ACL

**Summary**
- Open Domain Dialogue System에서 response를 적절하게 제시했는지를 평가하는 Evaluation Model을 만들었다는 논문임.
- Open Domain Dialogue System에서 적절한 response가 너무 많음. ground-truth response가 있지만, 그것과 전혀 다른 response도 문맥에 적절한 response가 될 수 있음.
그래서 대화의 context를 고려하여 evaluation해야함. ground-truth과 전혀 달라도 문맥에 맞는 response라면 높게 평가해야함. 
- 학습 dataset을 twitter의 긴 공개 대화를 크롤링해서 만듦. dataset에서 positive sample은 (conversation, ground-truth response) pair로 만듦. negative sampling으로 negative sample을 4개만 만드는데, 4개 sample 각각의 적절성(relevance to context)를 다르게 구성함. 
- performance가 가장 좋았음.

**Related Works**
- BLUE, ROUGE : 단순히 ground-truth response에서 사용된 word가 얼마나 포함되어있는지로 점수를 매김. ground-truth과 전혀 다르지만 문맥에 적절한 response는 제대로 평가하지 못함.
- EMB : response에 대한 sentence-level embedding을 구하고 유사도를 비교. Human annotated score와 correlation이 없음.
- ADEM, RUBER : context를 반영하는 방법. SSREM과 같은 approach. ADEM은 학습시 Human annotated label이 필요하고, RUBER은 학습시 하나의 positive sample과 하나의 negative sample만 사용함.

**Main Idea**
- training dataset 구성 : 4종류의 negative sample에서 한 종류당 1개의 sample씩 총 4개를 만듦. (SC, SP, SS, Rand). SC > SP > SS > Rand 순으로 적절한 response에 가까울 것으로 가정함. 하나의 ground-truth 당 4개의 negative samples를 붙여
총 5개로 softmax를 해서 구한 classification loss를 이용해 학습함. 각각의 negative sample에는 95% confidence interval을 둔다.(무슨 소리인지 모르겟음.)
  - Same Conversation(SC)는 한 conversation에서 발췌된 utterance. 
  - Same Partner(SP)는 한 partner랑 대화한 여러 conversation에서 발췌된 utterane 중 하나. 
  - Same Speaker(SS)는 한사람이 모든 partner와의 conversation에서 발화했던 utterance중 발췌된 utterance 하나. 
  - Random(Rand)은 dataset에 있는 모든 utterance 중 하나.

- 모델 structure
  1. context, response를 embedding한 후, context, generated response의 유사도를 구함. 
  2. ground-truth response를 embedding한 후, response와의 유사도를 구함.
  3. 1의 결과와 2의 결과를 각각 min-max-norm 한 후, 합침.
  4. 5개의 sample(1 pos, 4 negs)에 대해 softmax를 한 후, ground-truth에 해당하는 loss만 합치는 loss function을 사용함.


**Contributions**
- SSREM은 RUBER랑 비슷함. 다른 점은, RUBER은 1 pos-sample, 1 neg-sample을 사용한 반면, SSREM은 speaker-sensitive sample을 사용함. 따라서 이 논문은 기존 구조를 동일하게 한 채로, speaker-sensitive sample을 사용했더니 더 좋았다! speaker-sensitive sample을 사용하는 것이 중요하다! 는 논문임. 

**Experiments**
- Evaluation metric에는 Pearson상관계수, Spearman 상관계수를 사용함.
- 770K의 dataset 중, 300개의 conversation, 300개의 ground-truth를 추출함. 각 conversation마다 3개의 conversation model을 사용해서 response를 1개씩 총 3개 생성함. 
총 1200개의 response를 사람을 고용해서 Human annotated score를 얻음. SSREM이 평가한 게 human annotated score와 얼마나 correlate한지 봄.
- 실험할 때에는 sentence-level embedding방법을 baseline에서 썼듯이, embedding vector를 avg하는 방법을 사용함.
1. SSREM이 ground-truth, 4종류의 negative sample을 평가한 결과가 human annotated score의 분포와 얼마나 correlate한지 봄. SSREM이 가장 correlate하게 예측
2. SSREM이 ground-truth과 negative sample을 얼마나 잘 구별하는지 봄. SSREM이 가장 큰 similarity gap으로 구별함.
3. 아까까지는 Twitter conversation에서 test dataset을 썼는데, 새로운 domain의 dataset(Movie script)에서의 성능이 어떤지 봄. SSREM이 젤 잘함.

**Future Work**
1. SSREM뿐만 아니라 baselines 모두 다 adversarial attack에 취약함. 
    - stopwords를 제거하고 단어를 동의어로 바꾸는 attack 
    - copy mechanism (context내의 utterance를 그대로 복사해서 response로 제시하는 것) : SSREM, RUBER(baseline) 모두 ground-truth utterance보다 copied utterance가 더 적절한 것으로 평가했음. 하지만 SSREM은 copied utterance가 ground-truth보다 쪼끔 더 적절하다고 판단한 데 비해 RUBER은 더 많은 gap으로 copied utterance가 더 적절하다고 판단함. 
2. 현재 사용한 classification loss 대신 ranking loss를 사용해 hierarchical similarity를 가지는 sample들 간의 difference를 학습할 수 있음.
3. SSREM을 task-oriented dialogue와 같은 다른 곳에 사용해볼 수도 있음.

**총평**
- speaker-sensitive sample을 사용한 게 novel한 것 같음.

**질문or향후 연구방향 제시(교수님 랩면접 대비)**
- 현재 context를 단순 concatenate시켜서 response랑 similarity를 계산하는 것 같은데(코드 확인하기), 이러면 context의 모든 부분을 동일한 가중치로 반영하게 될 것이다. (논문에는 word embedding방법은 나와있지 않고, experiment에서는 word vector를 avg하는 방법을 사용하였음.)
  - context의 모든 sentence를 concatenate시키는게 아니라, context 내의 모든 word의 embedding을 average시킨다고 논문(p5, average라고 검색)에 나와있음. 
- SSREM의 구조는 함수 f, g의 결과를 mix시키는 것인데, f는 context와 generated response가 얼마나 비슷한지 보는 것이고, g는 generated response와 ground-truth response가 얼마나 비슷한지 보는 것이다. f의 성능을 향상시킬 수 있다면 더 좋은 SSREM이 되지 않을까?
  - 단순히 시간 순서대로, 최근의 utterance일수록 더 큰 가중치를 둬서 context에 response를 반영하게 해도 되고, 
  - attention을 써서(Seonwoo et al. 2020)의 Context-aware Answer Extraction in Question Answering처럼 유사도를 학습하게 해도 되고,
  - graph neural network를 써서 (정교민 교수 et al. 2020) Detecting Supporting Sentences for Question Answering via Graph Neural Networks 처럼 supporting sentence의 유사도를 학습하게 한다.

## Study

**읽는데 걸린 시간**
- 읽는데 3.5시간 + 정리하는데 1:45 시간
- Pages : 9 (Excluding References, Appendix)

**알게 된 지식**
- Dialogue System은 크게 Open-Domain Dialogue System과 Task-Oriented Dialogue System으로 나누어짐. 후자는 항공권 예매, 아마존 챗봇 customer service와 같이 수행해야 할 task, 해결해야할 문제가 명확하게 있는거고, 후자는 그런거 없이 노주제로 자유롭게 얘기하는걸 말하는거임.
- Open-Domain이든 Task-Oriented이든 response를 했으면 그게 적절한지를 평가해야함. Open-Domain 문제는 특히 문맥에 적절한 response 수가 너무나 다양하고 많기 때문에 적절히 평가하는게 어려움.
- confidence interval : 신뢰구간
  - 샘플링된 표본이 모집단을 얼마나 잘 대표하는지에 대한 수치.
  - 95% confidence interval이라고 하면, 모집단의 평균이 표본구간에 포함될 확률이 95%인거임.
- stopword : 불용어(I, my, me, over, 조사, 접미사 같은 단어들은 문장에서는 자주 등장하지만 실제 의미 분석을 하는데는 거의 기여하는 바가 없는 단어들)
- 통계학에서 p-value : 우리가 얻은 데이터에 있는 두 표본 집단이 같은 모집단에서부터 나온거라고 치자. 그랬을 때, 우리가 이런 검정 통계량(가령, t-value)을 얻었는데 이게 얼마나 말이되는거냐?

**아직까지 모르는 부분**
- Future Work에서 걱정하는 copy mechanism은 이미 negative sample로 SC에서 추출한 sample을 포함하기 때문에 괜찮지 않나? negative sample로 발탁되지 않은 utterance중 SC에 속하는 다른 utterance를 attack response로 썼을 때 문제가 된다는 건가?
  - SC에 속하는 negative sample로, SC의 모든 utterance를 concatenate시켜 하나의 neg sample in SC로 구성하면 안되려나?
- BM25 이 sparse retriever인 건 알겠는데, 정확히 어떤 원리인지는 모르겠음.
- autoencoder
- referenced metric, unreferenced metric, reference sentence 등등.. 에서 reference가 무엇을 나타내는 말인가?

