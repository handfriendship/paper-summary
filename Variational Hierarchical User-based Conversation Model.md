# Variational Hierarchical User-based Conversation Model
- Author : JinYeong Bak, Alice Oh
- 2019 EMNLP

**Summary**
- VAE기반의 Open Domain Dialogue System를 만들 때, utterance의 information 뿐만 아니라, user의 information까지 넣어서 모델링하고자 하는 논문임.
- 기존의 시도들 중에도 utterance, user 정보 모두를 고려하는 연구가 있었지만, 저자에 따르면 user에 너무 많은 가중치가 두어졌다고 함. (VHUCM)
- VHUCM은 초기 user의 embedding을 random하게 초기화하는데, user의 network를 graph로 표현하고, node2vec을 둬서 pre-trained한 embedding으로 초기화하는 방법인 VHUCM-PUE도 제안함.
- Twitter에서 대화형식의 트윗을 크롤링해서 데이터셋을 구성하고, 1) dataset을 train_test_split하여 만들어진 test set, 2) unseen user에 대한 테스트 환경, 총 2가지 환경을 구성하여 실험함.
- 두 실험에서, VHUCM-PUE의 pre-trained user embedding을 사용할 수 있는 상황에서는 VHUCM-PUE가 좋았고, VHUCM-PUE와 VHUCM이 똑같아지는 환경에서는 둘의 차이가 없었음.

**Related Works**
- HRED
- VHRED
- VHCR 
- SpeakAddr : seq2seq 기반.
- DialogWAE : WAE기반.
- VHUCM도 기존의 방식들과 model 구성에서는 크게 다르지 않음. user와 utterance의 가중치를 다르게 줬다는 정도.
- VHUCM-PUE에서 user의 embedding을 pre-trained시킨 방법은 기존의 vae기반 dialogue model과는 다른 방법.

**Main Idea**
##### 1. Initialization
  - user의 embedding h^user은 random으로 초기화되거나(VHUCM), node2vec에 의해 pre-trained된다(VHUCM-PUE).
  - utterance의 embedding에는 2가지 종류가 있다. 1) 이전 utterance만으로 만든 embedding인 h^enc과, 처음부터 이전까지의 모든 utterance를 고려해서 만든 embedding h^cxt.
##### 2. Variational Inference
  - z^conv를 inference하는 q_phi distribution q_phi(z^conv|c, h^user)의 parameter mu, sigma는 conversation을 구성하는 두 user의 embedding h^user에다가,
각 단계의 utterance embedding인 h^enc을 전부 모아 bidirectional GRU를 통해서 구한 h^conv라는 인자를 같이 고려하여 구한다. -> q_phi를 inference할 때 user의 정보 + utterance의 정보를 쓰는 것.
  - z^utt를 inference하는 q_phi distribution q_phi(z^utt|c, z^conv)의 parameter들은 conversation-level의 latent variable인 z^conv에다가 user의 임베딩 h^user, 모든 utterance를 고려해서 만든 embedding h^cxt를 씀.
##### 3. Encoding
  - latent variable은 2개가 있다. 1) utterance-level의 latent variable인 z^utt, 2) conversation-level의 latent variable인 z^conv
    - 1) z^utt의 prior인 p(z^utt)은 conversation-level의 latent variable인 z^conv, 모든 utterance를 고려해서 만든 embedding h^cxt과, 해당 z^utt를 발화한 user의 embedding인 h^user로부터 결정된다.
    - 2) z^conv은 그 conversation에 참여한 user들 각각의 임베딩인 h^user로부터 결정된다.
##### 4. Decoding
  - 최종 response를 생성하는 distribution은 p(x|h^cxt, z^utt, z^conv)를 MLE한다. 
즉, utterance-level의 latent variable인 z^utt, conversation-level의 latent variable인 z^conv에다가 처음부터 이전까지의 모든 utterance를 고려해서 만든 embedding h^cxt를 고려하여 디코딩하는 것.
  - encoder RNN으로부터 만들어진다. (이전 순서의 utterance로부터 recurrent하게 현재 turn의 utterance를 생성해나가는 구조.)

**Contributions**
- VAE기법 기반의 Open Domain Dialogue System에서의 성능향상 (approach의 novelty는 없는 것 같고, 그냥 SOTA(그것도 vae based model중에서만)를 찍었다는데 의의가 있음.)
- VHUCM-PUE가 Known Partners라는 특수한 상황에서 더 좋다. (근데 unseen user 상황에서 Known Partners라는 상황이 잘 오지 않을 것 같아서 굳이 필요한가 싶음.)

**Experiments**
- dataset을 구성할 때, each dyad마다 여러 conversation이 있을텐데, 그것을 chronological order로 배열함. 그 안에서 random하게 conversations를 추출해서 80/10/10 의 비율로 
train/dev/test set을 구성함. 
1. dataset의 test set으로 평가한 결과(이미 user의 정보를 아는 상태)
    - 논문에서 제안한 모델은 VHUCM-PUE을 썼고, 나머지 VAE기반 related works의 모델과 성능을 비교함.
    - 테스트 방식으로 1) BLEU, ROUGE-L 등의 automatic evaluation model, 2) Human Evaluation, 3) Personalized Responses, 세가지를 사용함.
    - 1), 2) 에서 모두 VHUCM-PUE가 더 좋았음.
    - 3) 에서는 model이 speaker의 정체성을 가지고 얼마나 일관되게 자신의 personal 정보를 토대로 답변하는지 평가함.
 또한 model은 다른 여러 user들과 여러 종류의 사회적 관계를 맺고 있을텐데, 그 관계를 고려하여 대답하는지도 평가함.
    - 3) 은 다른 모델들과 비교하진 않았고, 그냥 해석만 했는데, VHUCM-PUE가 적절한 수준으로 대답했음.
 2. unseen user가 등장했을 때의 성능을 평가한 결과
    - 원래 dataset에서 해당 user가 포함된 conversation을 test set에서만 남겨두고, train & dev set에서는 다 지움.
    - 지워진 conversation을 갖고있는 user의 유형을 지워진 정도에 따라서 Known Users, Known Partner, New Users의 3가지 유형으로 분류함.
    - New Users일때는 user embedding을 구할 수 있는 어떠한 단서도 없는 상태라 그냥 random embedding을 부여함. (VHUCM와 같음)
    - Known Partner일때는 user emedding을 partner의 embedding으로부터 평균을 구하고 noise를 추가하는 형식으로 부여함. (VHUCM-PUE의 차별점)
    - 1) unseen user에서도 잘 대응하는지, 2) VHUCM-PUE의 차별점이 VHUCM보다 더 좋은 성능을 이끌어내는데 일조했는지 두가지를 실험함.
    - 1) 에서는 VHUCM-PUE > VHUCM > Related Works 순으로 잘 했고,
    - t-SNE를 해봤더니 2)에서는 VHUCM-PUE에서 쓴 embedding기법의 intuition이 실제 embedding space에서도 잘 반영되었고, 성능향상으로까지 연결되었다는걸 보여줌.

**총평**
- Contributions에서 말한대로.

**질문or향후 연구방향 제시(교수님 랩면접 대비)**
- Q1. VHUCH-PUE가 VAE를 사용한 모델 말고 다른 approach를 사용한 모델들도 outperform하는가?

## Study

**읽는데 걸린 시간**
- 읽는데 11시간 + 정리하는데 0.5 시간
- VAE 공부하는데 5시간씀.
- Pages : 9 (Excluding References, Appendix)

**알게 된 지식**
- Variational AutoEncoder(VAE)
```
 - 이미지 생성 문제를 예로 들자면, 
 -> 어떤 이미지 x를 생성하고 싶음
 -> x의 prior인 p(x)를 알면 x를 생성할 수 있음.
 -> 그래서 p(x)를 구하기 위해, 아까 loss function을 maximum likelihood estimation 관점에서 해석했던 것처럼, p(x|f(z))를 maximization하는 문제로 생각해봄. 
 -> 어떤 모델을 probability distribution이라고 가정한다면, f(z)가 주어졌을 때(=모델에 해당하는 probability distribution의 mean, standard deviation과 같은 model parameter) x가 나올 likelihood인 p(x|f(z))를 극대화하는 문제로 해석함.
 -> f(z)의 parameter를 구하기 위해 MLE를 바로 적용해보니깐 잘 안됬음. 
 -> 왜그럴까 생각해보니 MLE를 directly 적용하는 것은 어려운 점이 있음.
 -> MLE는 loss function을 squared loss쓰는 격인데, raw image인 x는 엄청 고차원임. 고차원에서 squared loss를 쓰는 것은 바람직하지 않음. 왜나면, 고차원에서의 x와 x_hat의 차이(squared loss는 x-x_hat을 구하는 것이므로)는 실제 semantic한 의미의 차이를 말하지 않기 때문(mnist 숫자2의 예시를 생각해 보면 됨)
 -> 그럼 이제 f(z)의 parameter를 구하기 위해 MLE가 아닌 다른 접근 방식이 필요함.
 -> f(z)의 parameter인 z를 sampling할 수 있는 z의 prior인 p(z)를 구하자! 
 -> p(z)를 구하기 위해 현재 x를 given했을 때 z가 나올 posterior인 p(z|x)를 구해보자!
 -> p(z|x)는 z에 대한 true posterior이고, 이것을 바로 구할 수 없으니 p(z|x)를 approximation하는 분포를 만들어서, 그것으로부터 z를 구하고, 그것으로부터 f(z)를 만들어서 우리의 최종목표인 p(x|f(z))를 만들 것임.
 -> p(z|x)를 approximation하기 위한 방법으로 식을 풀고 풀다 보면, 최종 식이 나오고, 그것을 토대로 optimization problem을 풀게됨. 그 식의 형태가 ELBO + KL-Divergence(p(z|x) || q_phi(z|x)) 임.

 - conditional probability을 bernoulli로 두면 cross-entropy가 되고, gaussian으로 두면 MLE가 된다고 하는데, q_phi를 그렇게 두는걸 말하는건지? p(x)를 그렇게 두는걸 말하는건지?
 - variational inference가, 어떤 posterior를 approximation하기 위해서, 어떤 distribution을 잡고, 그것의 parameter를 조금씩 바꿔가면서 그 posterior를 추정하는거래. 여기서는 p(z|x)를 찾을 때, q_phi(z|x)를 어떤 distribution으로 잡고, x를 계속 넣어가면서 z를 추정하는거임. 근데 우리가 나중에 p(z|x)를 마음대로 조정하기 쉽도록 하기 위해서, p(z)를 simple한 distribution으로 둔대. e.g. gaussian or normal distribution
 - q_phi(z)에 대한 mu와 sigma(mean, std deviation)을 구하면 이제 그걸로 MLE가 가능한데, MLE를 sampling으로 구할 수 있는 방법이 monte-carlo 어쩌구임. sampling을 하나씩 해야하는데, sampling을 하게 되면 backpropagation을 할 수가 없음.(backpropagation으로 mu와 sigma를 update하는 것임)
```

**아직까지 모르는 부분**
- p4에서 q_phi function의 인자 c가 무엇인지 모르겠다. -> h^conv임.

