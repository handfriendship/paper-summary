# Generalizing to Unseen Domains: A Survey on Domain Generalization
- Author: Pan Li1, Da Li2,3, Wei Li, Shaogang Gong, Yanwei Fu4, Timothy M. Hospedales
- ICCV 2021

**Summary**
- 

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- 

**Main Idea**
- 

**Contributions**
- 

**Experiments**
- 


**총평**
- 

## Study

**읽는데 걸린 시간**
- 읽는데 4:10시간 정리하는데 2:00분
- pages: p8 (without references&appendix) 

**비판적 사고(개선점 찾기 / 비판 / 제안 등)**
- 아이디어1) language model처럼 source domain1학습 -> source domain2에서 평가, source domain-{1,2}에서 학습 -> source domain3에서 평가 .. 이런 식으로는 안되나?

**질문**
- Q1. SFA-A는 그렇다 쳐도 SFA-S는 왜 잘되나? 이것이 잘되는 것에 대한 intuition이 뭔가? 
- Q1-1. SFA-A가 잘되면 general-purpose regularization technique도 DG에 효과가 있을 수 있다는 것 아닌가?
- Q2. MixStyle은 왜 비교안하냐? 똑같은 data augmentation 종류인데?
- Q3. KxK covariance matrix에서 k-dim noise를 N개(domain의 개수)만큼 sampling하는건가? 아니면 같은 noise가 각 도메인에 더해지는건가?
