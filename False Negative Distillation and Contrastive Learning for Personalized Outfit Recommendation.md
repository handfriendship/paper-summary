# False Negative Distillation and Contrastive Learning for Personalized Outfit Recommendation
- Author : Seongjae Kim
- 2021 arxiv

**Summary**
- personalized outfit recommendation을 하는 모델이고, 기존 baseline(LPAE)에다가 knowledge distillation과 contrastive learning을 적용해 polyvore dataset에서 sota를 찍은 모델이다.
- 기존 personalized outfit recommendation에는 두 가지 문제가 있다고 한다. (data sparsity 문제, huge memory & time cost 문제)
- time cost문제는 false negative distillation로 해결했다. teacher model이 내뱉은 결과 중 false negative라고 의심되는 sample에 대해 더 강하게 penalize를 하는 student model을 만드는 기법이다.
- data sparsity문제는 각 user당 주어진 outfit개수가 부족한 문제인데, 이것은 하나의 positive sample당 10개의 negative sample을 두고, 그것들을 두가지 방법으로 augment하여 총 20개의 negative samples를 사용하는 contrastive learing을 적용하여 해결했다
- polyvore류의 dataset에서 기존 baseline보다 높은 성능이 나왔다.

**Related Works(기존의 방법론, 기존의 방법에 비해 우리가 왜 더 좋은지)**
- 크게 personalized outfit recommendation이랑 non-personalized outfit recommendation으로 나뉜다.
- baseline: LPAE (Learnable Personalized Anchors Embeddings)
  - personalized outfit recommendation을 푸는 모델이다.
  - Key contribution은 각 user를 여러개의 anchors로 모델링하는 것이다.
  - 자세한건 LPAE도 읽고 정리했으니 github 참고.
- bidirectioanl LSTM기반 방법론
- attention기반 방법론

**Main Idea**
- False Negative Distillation
  - idea: teacher model을 학습시키고, teacher model에게 training sample들에 대해 inference를 시키면 false negative스러운 녀석들을 찾을 수 있음. 그걸 더 penalize하는 방향으로 student model을 학습시킴.
  - teacher model: 1개의 pos sample당 10개의 neg sample이 있고, 총 11개 중 pos sample의 softmax output을 maximize함.
  - student model: 
    - teacher model이 embedding시킨 모든 pos sample과 user embedding간의 거리 r_i_hat을 구함. 
    - r_i_hat보다 거리가 더 가까운 neg sample은 false neg sample로 판단함.
    - pos sample의 softmax를 maximize하는 과정에서, false neg sample가 섞여있으면 loss값이 더 커질 것이니, 더 강하게 penalize됨. 
      - false neg sample의 softmax값은 더 커지게 되므로 전체에서 차지하는 비중이 높아짐.
    - 그냥 neg sample이 있으면 student objective로 구한 loss가 teacher objective로 구한 loss보다 더 작게 나옴. -> 별로 가중치를 update해줄 부분이 없음.
      - 그냥 neg sample의 softmax값은 더 작아지게 되므로 전체에서 차지하는 비중이 줄어듬.
- Contrastive Learning
  - idea: SimCLR와 똑같음. 각 pos sample를 두가지 다른 방법(erase, replace)으로 augmentation시키고, 두 sample들은 서로 가까워지게 하고, 다른 sample들끼리는 멀어지게 함.
  - erase: 한 outfit내 3가지 items들 중 하나를 지움
  - replace: 한 outfit내 3가지 items들 중 하나를 다른 하나로 교체함.
- 그외
  - model input: 각 item이 하나의 input 단위임. item을 patch로 쪼갠다거나 하지는 않음. outfit 내의 item은 순서 상관X
  - model architecture: AlexNet(backbone) + Set Transformer
    - AlexNet: AlexNet-Large (for teacher model), AlexNet-small (for student model)
    - Set Transformer: Encoder를 두 개 쌓아서 각 item별 representation(총 3개)를 만든 뒤, 
randomly initialized된 seed를 query로 둬서 하나의 encoder를 더 쌓아서 해당 query의 representation을 outfit의 representation으로 사용.

**Contributions**
- false negative distillation을 사용해서 모델을 경량화함.(정확히는 backbone을 경량화)
- contrastive learning을 이용해서 data sparsity문제를 해결하고 성능을 높임.

**Experiments**
- Dataset:
  - Polyvore-{630, 519, 53, 32}를 사용.
  - 630, 519를 가지고 대부분의 실험을 함. 각 user당 outfit수가 많음.
  - 53, 32는 cold user에 대한 실험. 각 user당 outfit수가 적음.
- Cold user에 대응하는 실험.
  - 각 user의 outfit이 1개, 5개만 주어진 극단적인 상황에 대해 실험함.
  - LPAE는 cold user profiling문제를 target한 baseline인데도, LPAE보다 잘함.
- Hard Negatives에 대한 실험을 함.
  - 모델을 training할 때 절반은 negative, 나머지 절반은 hard-negative를 이용.
- cold starter에 대한 실험.
- 여러가지 hyper-parameter적인 tuning을 함.
  - alpha를 바꿔봄. (false-negativeness의 정도를 조절하는 상수)
  - data augmentation기법을 다르게 적용해봄.
  - model size를 키워봄.(backbone의 model size를 말함.)
  - batch size를 다르게 해봄.
- Evaluation Metric: AUC, NDCG(Normalized Discounted Cumulative Gain)

**총평**
- minor domain이라 논문 내기 괜찮은 분야일지도??

## Study

**읽는데 걸린 시간**
- 읽는데 7:00 + 정리하는데 1:05분
- pages: 8 (without references&appendix)

**비판적 사고**
- SimCLR를 적용할 때, 3개 중 1개를 erase/replace하는데 semantic이 preserve가 되나? (1st augment는 erase, 2nd augment는 replace)
- abstract에서는 FND를 쓴 이유가 huge memory와 time cost때문이라고 했는데, Table 5.에 보면 required RAM이나 inference time이 딱히 안빨라졌다. backbone의 #params가 줄어들긴 했는데, 애초에 teacher model의 사이즈 자체가 크지 않아서 ..
- Key contribution이 무엇인지 잘 모르겠다.
- 보통 vision이나 NLP에서는 여러 dataset으로 평가를 하던데, Polyvore-{630, 519, 53, 32} 이렇게 4개를 사용했다지만, 사실상 같은 distribution에서 sampling된 것이니 하나의 dataset이나 다름없어보인다.
minor domain이라 괜찮은가? IQON-3000과 같은 dataset을 사용하면 안되나?
- FND는 false negative를 찾는거고, 이러한 점에서 결국 positive sample을 새로 만든거죠?
- Contrastive Learning에서 현재는 outfit끼리만 가까워지게 하고, outfit들과 user들을 가까워지게 하는건 안하는거죠?

**이해못한 점**
- data sparsity issue가 무엇인가? 단순히 user당 outfit개수가 부족한걸 data sparsity라고 하나?
- evaluation metric으로 사용된 NDCG가 뭐지?
- FND할 때는 data augmentation는 적용 안하는거죠?
- Eq(19)는 student model을 위한 것인지?
