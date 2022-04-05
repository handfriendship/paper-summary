# CTRL-C: Camera calibration TRansformer with Line-Classification
- Author : Jinwoo Lee, Hyunsung Go, Hyunjoon Lee, Sunghyun Cho, Minhyuk Sung, Junho Kim*
- 2021 ICCV

**Summary**
- camera calibration을 transformer 구조를 사용하여 해결한 논문이다. 
- single image를 input으로 사용하고, image에서 semantic cue와 geometric cue를 둘 다 사용하는데, 이 둘을 잘 
collaborate시켜 사용하기 위해 loss function도 합쳤고, 모델도 end-to-end 형식으로 하나만 사용하였다. line classification, camera intrinsic prediction 둘 다 supervised 이다.
- camera intrinsics은 global value이므로, input이 여러 local image patch, local line segment로부터 출발하여 모델을 거치며 global feature가 될텐데, 이 과정에서 
long-term dependency를 유지하도록 하기 위해 Transformer를 사용하였다.
- geometric cue를 사용하기 위해서 line segment를 classification하는 task를 보조 task로 풀게 하여 loss function에 집어넣고 같이 학습시켰다.
- line classification task 없이 Transformer만 사용했을 때도 SOTA가 나왔고, line classification 문제를 추가하여 풀게 했더니, 더 좋은 성능이 나왔다.

**Main Idea**
- Intuition:
  - Transformer를 사용하게된 intuition: Contribution 1.에서 설명함.
  - line classification task를 같이 풀게 한 intuition: geometric cue를 함께 사용하게 하기 위함. 기존에도 geometric cue를 사용하게 하기 위해 line classification task를 풀게 했음.

- 모델 Structure:
  1. raw input image의 backbone으로 resnet50을 사용함.
  2. positional encoding을 하여 transformer encoder의 input으로 넣음.
  3. encoder를 6개 쌓음.
  4. decoder의 입력으로는 1)encoder의 출력, 2)line segment, 3)intrinsic query 이렇게 3개가 들어감.
    4-1) 3.의 출력을 입력으로 씀.
    4-2) line segment를 LSD 알고리즘을 이용해 추출한다. 거기에서 512개를 추출한다.
    4-3) intrinsic query 3개를 embedding한다.
  5. decoder를 6개 쌓음.
  6. loss function은 line classification task의 loss와 camera intrinsic prediction loss를 같이 쓴다. 
    6-1) VP(Vanishing Point)의 경우, GT(Ground Truth)-VP의 embedding과 predicted-VP의 embedding의 cosine distance를 loss로 쓴다. horizon line의 경우, image의 left boundary, right boundary와
각각 교점을 만든다. left 교점과 right 교점 중 각각의 GT와의 차이가 큰 것을 고른다. 그리고 GT-교점과 predicted-교점의 차이를 loss로 쓴다. FoV의 경우, GT-FoV와 predicted-FoV의 차이를 loss로 쓴다.
    6-2) line classification의 경우, 4-2)의 결과인 512개의 line segment들을 vertical convergence, horizonal convergence, none of them 중 하나로 분류할 수 있는데, 이 3가지 카테고리로 softmax한 후,
cross entropy loss를 쓴다.

- Estimating Pseudo Horizontal VPs
  - dataset에는 zenith VP는 있는데 Horizontal VP는 없다. 따라서 레이블을 적절히 만들어줘야 한다.
  - L_in^i는 ith VP candidate인 v_i를 포함하여 만들 수 있는 모든 line segment의 집합이다.
  - m_i는 L_in^i의 모든 원소의 길이를 다 합한 것이다.
  - m_i가 가장 클 때의 v_i를 고르고, L_in에서 L_in^i를 뺀 다음, 다시 가장 큰 m_i에 대한 v_i를 고른다. 
  - 이 두 v_i를 이어서 pseudo horizontal VP를 고른다.
  - horizontal line은 VP가 모여있는 것이고, 따라서 가장 먼 곳에 위치해야 하고, 그렇기에 다른 line segment들로부터 가장 멀리 떨어져있어야 되서 이런 방법을 쓰는건가?

**Contributions**
1. camera intrinsics value는 global한 characteristics를 찾는 과정인데, CNN based model은 local feature를 추출하는 것을 시작으로 점점 global feature를 추출해 나간다. 이 때long-term dependency를 잘 유지하는게 중요하고, Transformer를 adopt하여 그것을 이루었다. 이렇게 하여 SOTA를 찍었다. 
2. end-to-end model을 만들었고, semantic cue와 geometric cue를 같이 leverage할 수 있다. (기존 related works에는 모델을 두개 만들어 두 정보를 각각 이용하게 하거나, loss function을 따로 뒀다.)

**Experiments**
- Dataset: Google Street View(GSV), SUN360을 사용. (Lee et.al.)가 사용한 curated dataset을 사용하는데, original image의 FoV, pitch, roll을 rectify시켜서 사용하였다.
- Up Direction, Pitch, Roll, FoV 와 같은 camera intrinsics value는 GT와 predicted value사이의 각도를 비교한다.
- 여러 기존 모델들과 Up Direction, Pitch, Roll, FoV, AUC를 비교해보았다. SOTA가 나왔다.
- Horizontal Convergence line, Vertical Convergence line classification task 문제를 
GSV로 training시키고 SUN360으로 test해보거나, 반대로도 해보았다. accuracy가 괜찮았다. 따라서 robust하다.
- Line Classification은 qualitative하게 직접 image에 그려서 나타내보았다. make sense하더라.
- ablation study로 Transformer, vertical convergence line classification task, horizontal convergence line classication task를 빼보고 넣어보았다. vertical task를 넣고 안넣고가 차이가 많이나더라.
- Decoding 부분의 출력은 전부 같으니, decoder를 1개~6개로 바꿔보았다. decoder를 많이 쌓을수록 성능이 올라갔는데, 성능 향상이 converge하더라.

**총평**
- 

## Study

**읽는데 걸린 시간**
- 읽는데 5:15시간 + 정리하는데 1시간

**비판적 사고(개선점 찾기 / 비판 / 제안 / 향후연구 등)**
- Vision분야의 Transformemr에서 input image를 나눈 patch의 size는 성능에 영향을 많이 줄 수 있는 요소이다. ViT논문에서도 patch size에 대한 ablation이 있었고, ViT를 포함한 여러 논문에서도 patch size를 중요한 요소로 다룬다. 이 논문에서는 patch size의 ablation은 커녕 patch size를 얼마를 사용했는지에 대한 언급조차 없었는데, 이런 것에 대한 설명이 있었으면 더 좋았을 것 같다.

**알게 된 지식**
- camera intrinsics
  - roll: 카메라의 세로축(카메라 렌즈를 뚫는 방향)을 기준으로 회전, pitch: 카메라의 가로축을 기준으로 회전
  - FoV(Field of View): 카메라의 시야각(카메라가 볼 수 있는 세상의 범위)
  - zenith VP: 내가 아는 Vanishing Point랑 같은 것
  - Vertical Convergence line: 세로방향 vanishing line
  - Horizontal Convergence line: 가로방향 vanishing line
- camera calibration을 하는 방향에는 traditional approach, deep learning based approach가 있음.
  - traditional approach: line segment를 찾음 -> 가로방향, 세로방향 inliers를 각각 찾음(RANSAC등을 이용하여) -> 그것을 VP, horizon line으로 놓음.
  - deep learning based approach: semantic cue(image patch를 이용) 방법, semantic cue + geometric cue(line segment를 이용) 방법, etc...
- image를 rectify시키다: image를 transformation시키다.
- ablation study: 모델에서 어느 한 부분씩 빼보면서(ablate) 성능에 어떤 변화가 일어나는지 test해보는 experiment.

**아직까지 모르는 부분**
- positional encoding이 왜 필요하지? (local patch들도 서로 비슷한 지역의 것들끼리 모여야 하기 때문에, 거기에서 순서가 필요한가?) 
- line segment 중 512개를 어떻게 추려내는지 그부분 알고리즘은 이해 못했음.
- Estimating Pseudo Horizontal VPs에서 왜 m_i가 가장 큰 v_i를 두 개 골라서 pseudo horizontal VP를 만드는지 그 이유가 아직 와닿지 않음.

