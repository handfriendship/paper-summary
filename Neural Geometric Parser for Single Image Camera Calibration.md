# Neural Geometric Parser for Single Image Camera Calibration
- Author : Jinwoo Lee, Minhyuk Sung, Hyunjoon Lee, and Junho Kim
- 2020 ECCV

**Summary**


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


**Experiments**


**총평**
- 

**질문or향후 연구방향 제시(교수님 랩면접 대비)**
- 

## Study

**읽는데 걸린 시간**
- 읽는데 5:15시간 + 정리하는데 1시간

**알게 된 지식**
- Extrinsic Parameters:
  - pitch: 카메라를 아래위로 tilt 시키는 것(고개를 끄덕이는 것)(x축 중심 회전)
  - roll: 고개를 좌우로 갸웃 거리는 것(z축 중심 회전)
  - yaw: 고개를 도리도리 하는 것(x축 중심 회전)

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


