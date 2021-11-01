# Neural Geometric Parser for Single Image Camera Calibration
- Author : Jinwoo Lee, Minhyuk Sung, Hyunjoon Lee, and Junho Kim
- 2020 ECCV

**Summary**
- single image의 horizon line, focal length를 가르쳐주고 학습시키면, horizon line, focal length, camera rotation, up vector(pitch, roll, yaw)를 predict하는 모델을 제시한다. 
- ZSNet(zenith scoring net), FSNet(frame scoring net)으로 이루어져 있고, 각각은 따로 학습된다.
- semantic cue(FSNet에서 raw image를 ResNet에 넣어서 씀), geometric cue(line segment)를 둘 다 leverage함.
- ZSNet은 line segment을 input으로 받아서 zenith VP candidate를 scoring해서 좋은 것만 뽑음. FSNet은 ZSNet의 결과, raw image, line segment를 input으로 받아서 focal length, camera rotation, horizon line을 얻어냄.
- angle, pitch, roll, FoV, AUC를 계산해 봤을 때 다른 모델보다 성능이 좋았고, test set을 unseen dataset으로 해보았을 때 generalizability도 높았다. Ablation study를 해보니 ZSNet, FSNet의 각 구성요소들은 모두 큰 역할을 하고 있었다.

**Main Idea**
- ZSNet: 
  - input: L_z, Z
    - L_z: line segments를 Eq. (4)를 이용해서 sampling함.
    - Z: L_z로부터 한 쌍의 line segments를 random하게 sampling해서 교점을 구함.
  - output: camera rotation, focal length
  - intuition: line segment로부터 zenith VP candidate를 얻어낼 수 있다.
  - 목적: zenith VP candidate를 scoring해서 GT와 가까운 zenith VP를 얻어내기 위함.
  - loss: 
    1) 각 candidate마다 score(=feature)를 매김. GT zenith를 이용해서 zenith VP candidate의 label을 구함. label=1인 candidate의 score을 전부 더해 loss로 사용함.
    2) zenith VP candidates의 평균과 GT zenith VP가 같아지도록 한다. 평균을 구할 때, 각 candidate는 그것의 score(=feature)의 크기만큼을 가중치로 주어 weighted avg를 구한다.
  - 과정:
    1) Z를 PointNet feature extractor를 이용해서 embedding함.
    2) L_z를 PointNet feature extractor를 이용해서 embedding함. 이후, global max-pooling해서 global feature를 뽑아냄.
    3) 1)결과와 2)결과를 concatenate함.
    4) MLP layer에 집어넣고, sigmoid에 넣어서 0~1 사이의 score를 산출한다.

- FSNet:
  - input: ZSNet output(zenith VP candidate), line segment
  - output: horizontal line, focal length / (manhattan world assumption) camera rotation, focal length
  - intuition: prior knowledge가 사용됨.
  - 목적:
  - loss: cross entropy loss. label은 (각 candidate의) s_vh_i를 step function을 이용해서 label로 바꿔서 사용함. value는 (각 candidate의) manhattan directions를 FSNet에 넣어서 나온 결과를 사용함.
    - s_vh_i: s_h_i, s_v_i를 평균냄
      - 1) s_h_i: horizon candidate이 GT와 비슷한 정도
      - 2) s_v_i: zenith VP candidate이 GT와 비슷한 정도
  - 과정:


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


**Experiments**
- Datasets: Google Street View, HLW
  - Google Street View: GT Horizon Line, GT focal length
  - HLW: GT Horizon Line


**총평**
- 

**질문or향후 연구방향 제시(교수님 랩면접 대비)**
- 

## Study

**읽는데 걸린 시간**
- 읽는데 11:30 + 정리하는데 1시간

**알게 된 지식**
- Extrinsic Parameters:
  - pitch: 카메라를 아래위로 tilt 시키는 것(고개를 끄덕이는 것)(x축 중심 회전)
  - roll: 고개를 좌우로 갸웃 거리는 것(z축 중심 회전)
  - yaw: 고개를 도리도리 하는 것(y축 중심 회전)

- pitch 추정 방법:
  - 직접 측정
  - vanishing line으로 측정
  - world coords와 camera coords 간의 변환행렬(translation+rotation)으로부터 추정

- 소실점(Vanishing Point)
  - 정의: 물리공간에서 평행한 직선들이 2D image에 담기면 한 점에서 만나는 것처럼 보이는데, 그 점이 Vanishing Point
  - 특징: 같은 평면 위에 있지 않아도 평행하기만 하면 같은 VP를 가짐
  - 소실점을 이용해 알아낼 수 있는 것:
    - 카메라의 내부 파라미터(초점거리, principal point)
    - ...
  - zenith Vanishing Point: 천정 방향에 있는 VP

- Vanishing Line(소실선):
  - 정의: 동일 평면에 속한 직선들과, 그 평면에 평행한 평면들에 속한 직선들의 Vanishing Points들은 모두 일직선상에 존재하게 되는데 이 선을 우리는 vanishing line이라고 함.
  - 기하학적 정의: 어떤 평면(사진 촬영에 대한 대상이 놓여있는 평면)에 대한 소실선(vanishing line)은 이 평면과 수평이고 카메라 원점을 지나는 평면이 이미지 평면과 만나서 생기는 교선이다.
  - 특징:
    - 한 image는 여러 개의 vanishing line을 가질 수 있음.
    - 카메라 광학축(optical axis)에 대한 소실점은 이미지의 주점(principal point)
  - 소실선을 이용해 알아낼 수 있는 것:
    - pitch 추정:
      - 방법: principal point과 vanishing line 사이의 거리 d를 구함. 또, focal length f를 구함. atan2(d, f)가 pitch
    - roll 추정:
      - 방법: 소실선 상의 두 점을 잡고 그 직선의 기울기를 구해 atan2()를 통해 각도를 구한다.
      - vanishing line과 roll과의 관계: 카메라 roll이 일어나면 vanishing line도 그 각도만큼 기운다.
      - 어떤 image에서 pitch, yaw를 구할 때 camera가 roll되어있든 아니든, vanishing line과 principal point 사이의 거리를 구하면 됨.
    - yaw 추정:
      - 방법: principal point에서 vanishing line에 내린 수선의 발과 카메라로 만들 수 있는 직선과, 소실점과 카메라로 만들 수 있는 직선 사이의 각을 yaw라고 두고 구함.
      - 절대적인 원점 기준을 잡기가 애매함.
      - 이미지상의 물체의 line segment의 방향과 optical axis의 사잇각을 yaw라고 가정함.
- 기타:
  - atan2(y, x): tan^-1(y/x)

**아직까지 모르는 부분**
- Q1. (p9) s_vh_i가 뭔지 모르겠다.
- Q2. (p9) s_vh_i를 구할 때 i가 무엇인가? i가 각 candidate의 index라면, s_h_i와 s_v_i에서 각 i를 어떻게 sampling하나?
- Q3. (p9) manhattan directions를 의미하는 R_i에서 i가 무엇을 의미하는지 모르겠다.

