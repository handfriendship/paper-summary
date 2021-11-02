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
  - input: 1)raw image(backboned with ResNet) 2)binary line segment map 3)3 activation maps 4)Mahnattan directions 5)estimated focal length
    - (이 모든 정보는 ZSNet output(zenith VP candidate), line segment를 통해 얻어짐)
  - output: horizontal line, focal length / (manhattan world assumption) camera rotation, focal length
  - intuition: 
    - 우리는 zenith VP, horizon line을 찾아야 하는데, horizon line을 찾기가 어렵다. 왜냐면 horizontal line segment는 horizon line방향이 아니더라도, 결국 horizon line에 있는 VP로 vanishing하기 때문.
    - 그래서 우리는 manhattan world assumption 하에 man-made scene의 특징을 이용할 것이다.
    - man-made scene은 rectangle을 많이 포함하고 있고, 이 rectangle을 둘러싸고 있는 line segment는 각각 vertical line, horizon line 방향일 확률이 높다.
    - 우리가 ZSNet으로부터 zenith VP는 scoring을 했으니깐, 현재 rectangle중에서 zenith VP와 방향이 비슷한 line segment를 갖고 있는 rectangle의 horizontal line segment를 구한다.
    - 이것을 이후에 어케어케 잘 sampling해서 horizontal VP candidate으로 쓰고, FSNet의 input으로 넣어서 scoring한다.
  - 목적: manhattan directions(zenith VP candidate, horizontal line candidate)를 scoring하기 위함.
    - scoring한 결과를 통해 최종 focal length, zenith VP를 산출함.
  - loss: cross entropy loss. FSNet이 predict한 manhattan directions가 GT와 같아지도록 학습시킴.
    - zenith VP는 점으로 비교하고, horizontal VPs는 그것을 line으로 만들어서 GT horizon line과 비교함.
    - GT horizon line과 비교할 때는 left, right border와의 교점을 만들어서 그것이 얼마나 차이나는지를 비교함.(CTRL-C와 같은 방법)
    - label은 (각 candidate의) s_vh_i를 step function을 이용해서 label로 바꿔서 사용함. value는 (각 candidate의) manhattan directions를 FSNet에 넣어서 나온 결과를 사용함.
    - s_vh_i: s_h_i, s_v_i를 평균냄
      - 1) s_h_i: horizon candidate이 GT와 비슷한 정도
      - 2) s_v_i: zenith VP candidate이 GT와 비슷한 정도
  - 과정:
    1) horizontal VP candidates를 찾음
      - horizontal line segments와 pseudo-horizon과의 intersections를 구함.
        - pseudo-horizon은 zenith representative를 통해서 구함.
        - zenith representative는 모든 zenith VP candidates의 weighted avg해서 structure tensor로 바꾼 다음, largest eigenvector를 구해서 구함.
      - zenith representative과 image center를 잇는 선을 pivot으로, intersections를 두 group으로 분류함.
      - 두 group에서 random하게 하나씩 뽑아서 horizontal direction line을 만듦.
    2) binary line segment map을 구함. (line segment가 있는 pixel은 1, 아니면 0)
    3) 3 activation maps를 구함. (line segment 중 vanishing line과 가까운 것에 가중치를 둠. 가깝다는 말은 각도가 가깝다는 말임.)
    4) Manhattan directions를 sampling함. (잘 모르겠다.)
    5) estimated focal length를 넣음. (zenith VP 1개, horizontal VP를 1개 sampling해서, 두 점을 Eq. (1)에 넣고 풀어서 구한다고 함.)
    6) 1)~5)를 전부 concatenate(channel에 추가하는 식으로)시켜 FSNet의 input으로 넣고, 위에서 설명한 loss로 학습시킴.
    7) 6)이 끝나면 모델은 zenith VP candidate, horizontal line candidate를 scoring함. line segments들이 manhattan world assumption과 얼마나 일치하는지에 대한 정도를 weight로 반영하여 최종 focal length, zenith VP를 구함.
    8) 최종 focal length를 Eq. (1)를 사용해서 pitch, roll, yaw를 유도할 수 있음.
    9) pitch, roll, yaw를 통해서 camera rotation을 유도할 수 있는 듯?

**Contributions**
- SOTA가 나왔다.
- manhattan world assumption 상의 특징을 잘 이용해서 horizontal line segment를 추려내고, horizon line을 구했다.

**Experiments**
- Datasets: Google Street View, HLW
  - Google Street View: GT Horizon Line, GT focal length
  - HLW: GT Horizon Line
- 6개의 모델들과 비교했더니 SOTA가 나왔다. 공정한 비교를 위해 모델에 따라, backbone으로 ResNet을 넣어주거나, 몇몇 parameter를 predict하지 않을 경우, GT를 넣어주거나 했다.
- Google Street View로 학습시키고 HLW로 test하거나, HLW로 학습시키고 Google Street View로 test하며 generalizability를 test했을 때도, 우리 모델이 젤 잘했다.
- 정성적 비교(직접 sample image에 plot)를 해봤을 때도 make sense했다.
- ZSNet의 성능만 비교해서 zenith VP를 예측하는 싸움을 했을 때도, 다른 모델보다 본 모델이 더 잘했다.
- Ablation Study로 모델의 구성요소들을 한 부분씩 차례대로 빼봤는데, 모든 구성요소들이 전부 큰 의미를 갖고 기여하고 있었다.


**총평**
- 

**질문or향후 연구방향 제시(교수님 랩면접 대비)**
- 

## Study

**읽는데 걸린 시간**
- 읽는데 11:30 + 정리하는데 3시간
- pages: 14 (without references, appendix)

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
- Q4. (p8) manhattan directions를 어떻게 2개의 VPs를 sampling해서 구한다는 것인지 모르겠다.
- Q5. (p8) 구한 manhattan directions는 image당 3개(zenith, both horizontal VPs)를 말하는거겟지? 모든 pixel의 자리에 같은 값이 들어가겠지? 그래서 channel(각 directions당 3개, 직선을 표현하려면 3개의 값이 필요하므로)이 총 9개가 필요한 거겠지?
