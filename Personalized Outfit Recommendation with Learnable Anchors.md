# Personalized Outfit Recommendation with Learnable Anchors
- Author : Zhi Lu, Yang Hu, Yan Chen, Bing Zeng
- 2021 CVPR

**Summary**
- multi anchors embedding을 통해 각 user를 표현하고, 같은 space에 outfits를 embedding한 후 similarity를 기반으로 personalized outfit recommendation, new user profiling을 수행한 연구이다.
- outfit의 경우, set transformer의 self-attention을 두 번 쌓아서 각 item들의 representation을 뽑은 후, pooling을 통해 단일 representation으로 나타낸다.
- user의 anchors matrix P의 경우, random initialize한 후 학습을 통해 만들어나가는 것 같다.
- outfit representation과 user anchors matrix가 만들어지면, 서로의 cosine similarity를 통해 user preference score를 매긴다.
- personalized outfit recommendation을 위한 모델 LPAE-u, new user profiling을 위한 모델 LPAE-g를 제시한다.
- new user profiling의 경우, new user anchors matrix를 구하는 세가지 방법 SO, SA, SU 중 하나로 anchors matrix를 얻은 후 진행한다.
- 두 task에서 모두 좋은 성능을 보였다.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- 다른 많은 연구들은 생략.. 
- outfit내 items들을 pair로 만들어 pairwise interaction을 구하는 approach
  - high order interaction을 capture하는데 실패.
- bidirectional LSTM approach
  - high order interaction은 capture가능.
  - not permutation invariant
- Motivation
  - 현재 fashion-domain에는 각 user당 존재하는 outfit data가 너무 작다는 문제가 있다.
  - 이전 연구들은 collaborative learning을 이용해 해결하려고 했다.
    - 하지만 이전 연구들은 다른 user의 outfit을 fashion의 content를 update하는데만 썼다.
    - 다른 user의 outfit을 current user의 embedding을 update하는데도 쓸 필요가 있다.
  - LPAE는 ...이다

**Main Idea**
- 각 user는 embedding space에 여러개의 anchors로 표현됨. 
- 각 user의 anchors, 모든 user의 general anchors 두 개가 있음.
  - intuition: new user의 personalized 정보가 충분치 않다면 LPAE-g가 성능이 더 잘나왔으면 좋겠고, 정보가 충분하다면 LPAE-g의 성능이 LPAE-u와 비슷해졌으면 좋겠다.
  - LPAE-u: personalized recommendation을 위한 모델. 각 user의 anchors만 사용됨.
  - LPAE-g: new user profiling을 위한 모델. 각 user의 anchors + general anchors 둘 다 사용됨.
- user의 각 anchor에 대해 outfit representation과 cosine similarity를 구한 후, 평균내어 (user_i, outfit_j)에 대한 preference score를 구함.
- 그 외:
  - model input: 각 item이 하나의 input 단위임. item을 patch로 쪼갠다거나 하지는 않음. outfit 내의 item은 순서 상관X
  - model architecture: AlexNet(backbone) + Set Transformer
    - AlexNet: AlexNet-Large (for teacher model), AlexNet-small (for student model)
    - Set Transformer: Encoder를 두 개 쌓아서 각 item별 representation(총 3개)를 만든 뒤, 
randomly initialized된 seed를 query로 둬서 하나의 encoder를 더 쌓아서 해당 query의 representation을 outfit의 representation으로 사용.
  - objective: Bayesian personalized ranking
    - orthogonal regularization을 함께 사용.
    - user가 선호하는 outfit은 user에 가까이 embedding됨.
    - user와 선호도가 비슷한 다른 user의 anchors들은 가까이 embedding됨.
      - 왜냐면 user의 embedding parameter도 similarity가 커지도록 update가 되므로
  - new user의 anchors matrix를 학습하는 방법:
    - SO(Learning by optimization): 바로 학습시키는 방법
    - SU(Learning by user-search): 모든 user를 순회하면서 r_ij를 구해서 그것이 가장 높은 user를 찾는다.
    - SA(Learning by anchor-search): new user의 outfit z_i를 바탕으로 closest user의 anchor를 ranking한 뒤에 top s개의 anchor를 고른다.(z_i가 하나뿐일때는 모든 anchor를 다 씀.)

**Contributions**
- 각 user는 embedding space에 여러개의 anchors로 표현하는 idea를 제시함.
- general anchors를 둬서 cold-user profiling에 대응함.

**Experiments**
- Dataset: Polyvore-{630,519,56,32}, IQON-3000 사용.
  - IQON-3000은 filtering을 거쳐 일부만 사용.
- 다른 모델들과 성능 비교하는 실험
- user를 표현하는 anchor의 개수에 변화를 주는 실험
- new user profiling task에서 SO, SU, SA의 성능 비교
- Evaluation Metric: AUC, NDCG
- LPAE-u, LPAE-g의 여러 상황별 성능 비교

**총평**
- 

## Study

**읽는데 걸린 시간**
- 읽는데 3:30 + 정리하는데 0:55분
- pages: 8 (without references&appendix)

**비판적 사고(개선점 찾기 / 비판 / 제안 / 향후연구 등)**
- ContraGAN 같은 논문은 data-to-label 정보뿐만 아니라 data-to-data relation을 이용하는데, 이 논문은 data-to-user (contragan의 data-to-label정보)만 이용함. data-to-data정보도 이용하면 성능이 오를 수도 있을듯.
- Class를 label로 놓고 multi-label classification문제로 환원해서 푸는 연구도 재밌을듯.

**이해못한 점**
- Eq (6)에서 a_ik와 z_j의 크기를 1로 만들어야 이유가 있나요?
- orthogonal regularization (log-determinant divergence(LDD)) 이해 못함.
- Bayesian personalized ranking(BPR) 이해 못함.
