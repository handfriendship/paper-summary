# Reducing Annotation Artifacts in Crowdsourcing Datasets for Natural Language Processing
- Donghoon Han, Juho Kim, Alice Oh
- Data Excellence Workshop 2020

**Summary**
- NLP task를 위해 Crowdsourcing으로 만들어진 dataset에서 annotation artifacts를 줄이는 방법론을 제시하고, 그 방법론이 어떻게 효과적인지 실험으로 검증함. 
- NLI task를 위한 dataset을 대상으로 실험했고, crowdsourcing 방법으로는 MTurk를 사용함.
- 기존 방법론은 대체로 model을 만들어서 annotation artifacts를 직접적으로(?) detect하고 수정하는 방법인 듯 한데, 이 논문은 dataset을 만드는 시점에서 constraint를 줌.
- Annotators에게 Premise를 주고 Entailment, Neutral, Contradiction 3종류의 hypotheses 문장을 만들도록 함. 이것을 작문할 때 특정 단어를 꼭 넣어서 만들도록 지시했는데, 
이것을 constraint word라고 부름.
- 적절한 evaluation 기법을 직접 만들어서 annotation artifacts가 얼마나 생겼는지 테스트함.
- constraint word를 사용하면 artifacts가 줄었는데, 1개만 사용하면 크게 줄었고, 여러개 중 하나를 선택하도록 만들면 덜 줄었음.

**Related Works**

**Main Idea**
1. condition SW
    - Intuition : worker들이 hypotheses를 작문하는 과정에서 각자 선호하는 방식이 있고, 그것은 word-level에서 선호하는 단어를 자주써서 발생한다.
    - constraint word를 줘서, hypotheses를 작문할 때 이것을 꼭 포함시키도록 하는 방식으로 dataset을 만들어보자.
    - constraint word는, 모든 class에서 최소 10번 이상 나오는 단어들 중에서, PMI가 가장 낮은 단어를 사용하자. intuition은 dataset을 구성할 때 최대한 다양한 단어를 넣어 구성하기 위함이다.
3. condition MW
    - condition SW 방식으로 하는 것은 너무 laborious하다. 그래서 constraint word를 5개정도 주고, 하나를 선택할 수 있게 하자는 방법.
    - SW보다 성능이 안나오는 이유의 intuition : 5개 중 1개를 선택하는 과정에서 worker들의 word selection preference가 개입된다.
5. Evaluation 방식
    - 첫번째 모델은 dataset A에서 학습시킴. 두번째 모델은 random classifier. 두 모델을 dataset B에서 테스트함. dataset A, B 혹은 둘다에 annotation artifacts가 있었다면 
두 모델에서 성능차이가 발생할 것임. 이것을 performance-gap of A on B라고 명명함.

**Contributions**
- annotation을 만들 때 workers들의 단어 선택 preference가 lexical pattern을 띄어 annotation artifacts를 만든다는 가정을 세우고, 그것을 분석으로 검증한 것.
- annotation artifacts를 reducing하는 방법으로 constraint word를 쓰는 방법론을 제안함.

**Experiments**
1) performance-gap of A on B의 (A, B)쌍을 {Baseline, SW, MW} 각각으로 하여 총 9개의 쌍을 만들어 성능테스트를 함. (baseline, baseline) 일때가
performance-gap이 가장 큼.(artifacts가 가장 높음). (SW, SW)가 가장 낮고, 그다음 (MW, MW)
2) MW에서 degree of freedom을 변화시키면서 성능 변화를 관찰함. DoF가 높을수록 작업이 쉬워지는 대신 artifacts가 덜 제거됨.

**Future Work**
1. annotation artifacts를 발생시키는 요소들은 word-level뿐만 아니라 다양한 level에서 존재할 것이고, 다른 level에서도 해보면 좋을 것 같음.
(e.g. syntactic level)  
2. constraint word를 선택하는 기준은, dataset의 diversity를 높이기 위한 방향이었음(PMI기반). 이런 의미에서, 현재 dataset을 구성하는 word들을
embedding space에 나타낸 뒤, 가장 먼 곳에 있는 word를 constraint word로 쓰면 어떨까?
3. Degree of freedom을 option의 수를 줄이는 대신 돈을 더 많이 주는 방식으로 조절할 수 있다.

**총평**
- -

**질문or향후 연구방향 제시(교수님 랩면접 대비)**
- Q1. 지금은 비교적 적은 수의 사람들(MTurkers)이 crowdsourcing에 참여하는 것처럼 보이는데, 이 논문에서 제안한 문제의 intuition은, workers들이 사용하는 단어 선택이 어떤 preference를 띄고,
그것때문에 dataset이 skewed되어 annotation artifacts가 생긴다는 것이다. 만약 훨씬 많은 사람들이 참여하게 된다면 각 개인이 영향을 미치는 비율이
줄어들 것이고, 자연스럽게 diversity가 확보되어 annotation artifacts가 줄어들지 않을까?

## Study

**읽는데 걸린 시간**
- 읽는데 4시간 + 정리하는데 0.5시간 걸림.
- Page : 4

**알게 된 사전지식**
- PMI : 사건 A가 일어날 확률과 사건 B가 일어날 확률 중에 사건 A와 B가 동시에 일어날 확률이다. 여기서는 해당 class에서 빈도수가 가장 적은 단어라고 생각하면 될듯.

**아직까지 모르는 부분**
- performance gap of A on B와 gap of B on A의 의미가 어떻게 다르지 잘 와닿지 않는다.
