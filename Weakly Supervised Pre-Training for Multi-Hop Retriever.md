# Weakly Supervised Pre-Training for Multi-Hop Retriever
- Yeon Seonwoo†, Sang-Woo Lee‡§, Ji-Hoon Kim‡§, Jung-Woo Ha‡§, Alice Oh†
- ACL 2021

**Summary**
- Multi-Hop Question Answering은 어떤 question에 대답하기 위해서 여러 개의 document을 참조해서 정보를 얻어야 하는 task이다. 이걸 하려면 complex question, supporting documents, answer를 사람이 직접 labeling해놓은 dataset이 필요한데, 이걸 모으는게 너무 costly하다. 그래서 weakly supervised 방법을 쓰고자 한다. End-to-End QA pipeline 중 어떤 component를 weakly supervised 방법으로 학습시킬껀가? QA system은 document retriever, (reranker), reader로 구성되는데, 여기서는 retriever(dense retriever)를 학습시킬 것이다. retriever를 학습시키기 위해 위에서 말한 weakly supervised 방법으로 pre-trained시킨 후, HotPotQA dataset으로 fine-tuning시켰다. 
- 이 논문은 pre-training시키는 방법을 제시한 논문임.

**Main Idea**
1. pre-training task : Next Document Prediction
-새롭게 retrieve할 nth Document를 결정하기 위해 (1~n-1)th document와 question을 사용함. 

2. data generation method : Bridge Entity Re-Phrasing
-Multi-Hop QA에서는, question에 대답하기 위해 필요한 정보인 bridge entity가 있고, 이것을 알기 위한 sub-question을 만들어 낼 수 있다. Weakly supervised 방법을 쓰기 위해 dataset을 스스로 만들어야 한다. dataset을 만들기 위해, Wikipedia에서 hyperlink가 있는 entity를 bridge entity라고 하기로 가정한다. Bridge Entity를 찾았으면, 그것을 설명해놓은 문서에서 나와있는 정의 설명 문장을 가져다가 bridge entity자리에 넣는다.(Bridge Entity Re-phrasing) 그리고 최종 answer로 사용할 entity를 what으로 치환하여 question을 만든다.

3. pre-training model : Xiong이 제안한 multi-hop dense retriever를 쓰는데, 이건 DPR기반임. DPR은 question encoder와 document encoder 두 가지를 사용함. 둘 다 dense retriever이고, RoBERTa-base임. question + (1~n-1)th document를 다 concatenate시켜서 question encoder에 넣어서 나온 값이랑, nth document를 document encoder에 넣어서 나온 값을 내적했을 때 가장 큰 값이 나오게 하는 document를 찾아서 nth document로 선택함. Loss function은 softmax 기반 cross-entropy인 것 같은데, softmax를 구하는게 costly하므로 in-batch-negative를 쓴다.(batch크기만큼 negative sample을 뽑는다는 의미인가?) 우리가 training시키고자 하는 모델은 question encoder, document encoder임.

4. 그 외
-fine tuning : retriever 부분을 fine-tuning시키는데, 정답 document는 TF-IDF로부터 가져오고, negative sample은 TF-IDF negatives로부터 random하게 2개를 추출해서 가져와서 fine-tuning한다는 것 같다.(사실 잘 모르겠음.)
-LOUVRE를 multi-hop QA pipeline에서 적용해 본 부분 : multi-hop document retrieve를 하는 전략에는 3가지가 있음. sparse retrieval, wikipedia hyperlinks, reranking. LOUVRE-Wiki는 LOUVRE로 pre-trained된 모델을 wikipedia hyperlink 방식으로 fine-tuning시켰다는 말이고, LOUVRE-reranking은 LOUVRE로 pre-trained된 모델을 reranking방식으로 fine-tuning시켰다는 말인 듯.

**Contributions**
- Open Domain QA에서 weakly supervised를 수행하기 위한 연구가 많이 있는데, 대체로 single-hop QA에 관한 연구다. Multi-hop QA에서 weakly supervised를 수행하기 위한 연구들은 기존의 HotPotQA dataset을 사용해서 dataset을 만드는 방법이거나, GPT2를 사용해서 question을 generation하는 방법이다. 이 연구는 더 general한 방법을 사용해서 weakly supervised 방법을 수행한다. (결국 이 이유 때문에 SOTA가 나왔고, computational efficient하고, few-shot learning에 robust하고, 등등 .. 하다는 소리임.)

**Experiments**
1) retriever, reranker를 test했는데, fine-tuning은 똑같이 한 채로, pre-training model을, 하나는 LOUVRE, 다른 하나는 RoBERTa-base로 했을 때 LOUVRE가 더 잘됬다.
2) LOUVRE는 비교대상 baseline모델보다 더 많은 epoch(=15epoch)을 가지고 training되었는데, baseline모델(RoBERTa-base)을 이보다 더 많은 50epoch까지 늘려봤는데도 LOUVRE가 더 잘 됬다. 따라서 LOUVRE가 잘되는 이유는 training time이 길어서 그런게 아니라, weakly-supervised method가 novel하기 때문이다.
3) LOUVRE에 여러 가지 variation을 줘서 training시켜보고, baseline과 비교해봤다.
4) pre-trainined 된 LOUVRE를 가지고 few-shot learning을 시켜보고, baseline과 비교해봤다.
5) Computational Efficiency를 #BERT 시간 단위로 측정하고 baseline과 비교.
6) End-to-End pipeline으로도 구성시켜서 test해봤다.
등등 ...

**총평**
- fine-tuning부분을 잘 이해하지 못함.

**질문or향후 연구방향 제시(교수님 랩면접 대비)**
- Q1. 이 논문에서 제시된 실험은 죄다 LOUVRE랑 기존 연구랑 비교할 때, 기존 pre-trained된 language model을 HotPotQA dataset으로 fine-tuning시키는 것 보다 LOUVRE를 논문에서 제시한 weakly supervised방법으로 pre-trained시킨 후, HotPotQA dataset으로 fine-tuning시키면 더 잘된다! 하는 실험밖에 없는데, 기존 pre-trainined된 language model에다가 논문에서 제시한 방법으로 pre-trainined시킨 후, HotPotQA dataset으로 fine-tuning시키면 더 잘될 것 같은데요?
    - 본 실험은 LOUVRE라는 pre-training method가 다른 method보다 뛰어난지 보고자 하는 것이므로, 둘 간의 비교를 위해서는 본 실험 구성대로 하나씩 적용해서 테스트하는 것이 맞다.   

- Q2. weakly supervised를 하기 위해 dataset을 만들 때, Document B의 맨 첫 문장에서 Bridge Entity를 지운 채 re-phrasing을 하고, Document B에서도 Bridge Entity(Subatomic particles)를 지운다고 했다. 하지만 Bridge Entity를 지워도 그것이 추출된 document의 문장 내 다른 단어들을 너무 많이 그대로 포함하고 있는데, 이러면 extractive한 정보가 너무 많이 들어가있어 retrieval이 너무 쉽게 되는 문제점을 가지고 있진 않던가요?
    - 1) Document B에서 첫 줄을 전부 삭제 or BERT의 masking기법을 이용하면 어떨까요?
    - 2) 혹은, 어짜피 dataset을 만드는 과정은 prior 과정일텐데, 다른 모델을 가져와서 abstract한 방향으로 paraphrasing하거나 하면 안될까요?
    - Loss Function을 보고 retrieval이 쉽게 되는 과정이랑 document encoder가 학습되는 거랑의 관련성을 보기
    - Dataset을 보고 2)이 말이 되는지 확인하기

- Q3. 실험에서 Evaluation Metric을 통일해야 하는 것 아닌가요?(잘 되는 evaluation에 대해서만 figure를 그린 느낌?)

## Study

**읽는데 걸린 시간**
- 읽는데 7시간쯤 걸림. + 정리하는데 3시간 걸림.

**알게 된 사전지식**
- Open Domain QA : 정해진 Document를 읽고, question에 답하는 문제가 아니라, question에 답하기 위해 retriever로 적절한 document들을 찾고, 그 후 그 안에서 reader를 통해 찾는 것
- Dense Retriever : Document를 표현하는 방법에는 sparse, dense가 있는데, 그 중 dense는 retriever를 학습하여 document를 embedding으로 표현하는 것. document의 searching space가 방대하기 때문에 dense retrieve는 어려운 문제이다.

**아직까지 모르는 부분**
- reranker가 정확히 뭐지?
- retriever에 대한 fine-tuning을 HOTPOTQA로 했다는 건지, 무슨 데이터셋으로 한건지 잘 모르겠다. fine-tuning하는 방식은 TF-IDF negatives로 했다는데 정확히 무슨 방식인지 잘 모르겠고, LOUVRE-Wiki, LOUVRE-reranker이 어떤 방식인지도 잘 모르겠다. TF-IDF은 beam size(document candidate를 몇 개 까지 가져올껀지)를 정해줘야 하는데, reranker는 beam size 이외에도 reranker의 input size도 설정해 줘야 한다.



