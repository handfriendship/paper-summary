# Domain Generalization via Feature Variation Decorrelation
- Author: Chang Liu et al.
- ACM MM 2021

## 세가지가 꼭 들어가있어야 함.
- 어떤 것을 풀려고 했는지(Summary)
- 기존에는 무슨 문제가 있는지, 기존의 방법에 비해 우리가 왜 더 좋은지(Related Works)
- 어떻게 풀었는지(Main)

## 무엇을 캐치하려고 읽었나?
- 1)내 idea가 이미 나왔던 아이디어인지 체크하려고(ISDA논문을 citation하고 있길래..)
- 2)channel간 covariance를 이용하는 아이디어이면, 이걸 어떤 식으로 활용하는지 아이디어를 좀 얻으려고.
- 3)feature space에서의 연산이 동작하는 메커니즘에 대한 직관과 이해를 좀 얻으려고.

**읽으려는 목적에 맞는 정보를 얻었나?**
- 1)내 아이디어랑 다름.
- 2) 안나와있음.
- 3)
  - class단위(모든 domain을 다 union)의 mean을 구하는 방법이 나와있음.
  - 어떤 sample에서 class-prototype정보를 제외하고 semantic variation정보만 남긴 것을 벡터의 뺄셈으로 정의함.
    - variation: semantic meaning(shape, color, visual angle, background), domain variance
  - 어떤 sample을 augment할 때 다른 class에 속한 variation을 입히는 방식으로 augmentation시킬 수 있음.
    - 어떤 sample의 label을 맞추는 classification을 푼다고 할 때, 해당 sample이 다른 background, color, illumination .. 등의 상황에 있다고 해도 맞출 수 있어야 한다는 것이 intuition
  - variation만을 보고는 어떤 label의 variation이었는지 맞출 수 없게 함.

**Summary**
- 

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- 1)기존에는 무슨 문제가 있나요?
  - DG를 위한 새로운 접근 방법을 제시함.
- 2)기존의 방법에 비해 왜 더 좋은가요?
  - DG를 위한 새로운 접근 방법을 제시함.
  
**Main Idea(어떻게 풀었는지)**
- ㅇㅇ

**Contributions**
- ㅇㅇ

**Experiments**
- ㅇㅇ

**총평**
- ㅇㅇ

## Study

**읽는데 걸린 시간**
- 다 안읽음.

**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- ...

**이해못한 점**
- ㅇㅇ

