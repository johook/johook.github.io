---
published: true
title:  "Transfuser"

categories: 
  - SensorFusion
---



# [Transfuse] Multi-Modal Fusion Transformer for End-to-EndAutonomous Driving
  
  
## Abstract
Transformer의 self-attention을 이용해서 Image와 Lidar를 fusion하겠다

## Introduction
- Image only / Lidar only 방식의 end-to-end 구조가 object detecion에 좋은 성과를 보여준다. 

-  하지만 이러한 연구들은 동적인요소가 제한된 환경 or 다른 에이전트가 이상적인 행동 한다는 가정을 한다 

- 따라서 한가지 데이터 input 사용한 모델은 이상적인 움직임 보이는 객체가 많은 환경에서만 높은 성능을 보여준다
- 즉 **adversarial scenarios**에서는 성능이 만족스럽지 못하다(adversarial scenarios= 빨간불에 달리는 차 / 4교차로 / 갑자기 나타나서 길을 건너는 보행자 등 )
- Lidar는 넓은범위(카메라 보이지않는 범위)인식_아래사진에서는 좌측에있는 차량 Lidar는 인식하지만 카메라는 인식하지못함 하지만 lidar는 신호등 색을 검출못함 카메라는 신호등색 검출 가능
-  이렇게 각자 장단점이 있기때문에 Sensor Fusion을 해야한다
![enter image description here](https://velog.velcdn.com/images/minkyu4506/post/18537d03-8701-4bc7-90aa-f1cd4b26a51a/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202021-08-06%20%EC%98%A4%EC%A0%84%2012.14.18.png)
- Sensor Fusion할때 대부분 2D image space와 BEV, RV와 같은 3D projection space를 투영을 통해서 정보를 집계하는 geometric feature projections 
- 한가지 데이터 input보다는 좋지만 이러한 아키텍처 디자인은 복잡한도시장면에서는 성능이 저하된다. 
- 그 이유는 깊은 convolution 신경망은 하나의 modality에서 global context는 포착잘하지만 여러 modal에 대해 확장하거나 특징간 상호작용을 모델링하는게 어렵기 때문이다

따라서 **Transformer의 attention 메커니즘 사용**하겠다. 

transformer의 self-attention= 입력값의 각 원소가 전체적인 입력값의 어느부분을 더 주목해야하는지 반영해주기 때문에 이를 이용해서 이미지, 라이다 데이터를 전체적으로 고려해서 특성맵 추출하겠다 
이 저자는 single-view image 와 Lidar를 입력으로 한다 이 두 input들이 서로 상호보완성(서로가 서로에게 부족한점을 채워주는)이 있기 때문이다

## Method
1. Problem Setting 
2. Input and Output Parameterizations 
3. Model architecture 
	a. MMTF b. Waypooint prediction Network

	
### Method_1. Problem Setting
모델이 수행할 Task : point to point navigation(목표지점까지 waypoint를 따라 다른 객체충돌, 교통법규 어기는것 등과 같은 사고없이 완주하는 것) 

학습방법: Imitation learning(IL) 
Imitation learning은 전문가(expert)가 하는걸 따라하는 학습법(Behavior Cloning) 으로 지도학습이다.(*“We consider the Behavior Cloning (BC) approach of IL which is a supervised learning method.”*)


**Imitation learning**: 데이터셋 수집 -> 학습 

데이터셋 수집: CARLA에 있는 가상환경에서 수집 ![enter image description here](https://velog.velcdn.com/images/minkyu4506/post/bf940494-966f-4a2d-ab3e-15adb7e16afe/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202021-08-05%20%EC%98%A4%EC%A0%84%209.06.20.png)

X는 “high dimension observation of environment” 전면 카메라 이미지+라이다 PCD한장(이전 IL연구들에서 observation history들이 유의미한 이득을 얻어내지 못한다는 연구결과가 많기때문에 한장만 사용한다.) 

W는 “corresponding expert trajetory” T개의 waypoint가 모인 것이 w 

학습: Loss 함수-> expert가 주행한 경로와 우리가 만든 모델이 예측한 주행경로의 loss가 최소가 되게끔 policy를 학습시킨다 ![enter image description here](https://velog.velcdn.com/images/minkyu4506/post/42199532-a602-4a20-89e2-8a21126a3759/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202021-08-05%20%EC%98%A4%EC%A0%84%209.11.08.png)


이렇게 학습을 통해 모델이 예측한 경로를 inverse dynamics model에 넣은 action으로 주행한다고 한다. 이때 inverse dynamics model을 **PID controller**로 구현한다.

![enter image description here](https://upload.wikimedia.org/wikipedia/commons/thumb/4/40/Pid-feedback-nct-int-correct.png/400px-Pid-feedback-nct-int-correct.png)

### Method_2. Input/output parameterizations


***Input representation***
*Lidar pointcloud*: 얻은 Pointcloud 중 ego-vehicle의 전면 32m, 좌우 측면 16m씩해서 총 32x32m 영역만 사용 -> 2D 데이터로 변환(256x256 픽셀데이터)=한 셀당 0.125mx0.125m + 높이에 대한 채널 2개(지면위/지면 밑) 따라서 256x256x2(높이채널) 사이즈의 데이터를 얻는다 
*camera image*: 얻은 이미지 400x300 사이즈 중 256x256영역만 추출하여 사용 이유는 렌즈 구조상 외각 이미지가 왜곡되어있기 때문 따라서 256x256x3(RGB) 사이즈의 데이터 얻음
***Output representation*** 
출력값 즉 waypoint는 BEV space에서 (x,y) 의 양식을 지닙니다. ![enter image description here](https://velog.velcdn.com/images/minkyu4506/post/e1e69200-2870-4820-87fb-71022c6b7fa2/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202021-08-05%20%EC%98%A4%EC%A0%84%2010.05.40.png)
![Screenshot from 2023-05-20 20-52-23](https://github.com/johook/Codingtest/assets/116954375/cd32c2bc-9698-4f5e-8027-d5bc06cdecd1)

### Method_3. model architecture
**MMTF(Multi-Modal Fusion Transformer)**
key Idea: global context(image+PCL)를 self-attention 메커니즘을 통해 혼합하는것
[Transformer의 큰 3가지 구조 설명]
![Screenshot from 2023-05-20 20-55-15](https://github.com/johook/Codingtest/assets/116954375/f24fb523-c604-411d-a96c-987d2b8b579f)

하지만 Transformer 모델의 NLP(Natural Language Processing_자연어처리) 에서의 토큰입력구조와 달리 object detection은 그리드 구조의 feature map에서 작동을 한다.

이전 object detection에 대한 transformer 적용에 대한 연구들은 transformer를 이미지 분류 및 객체 감지와 같은 작업에 적용하는 데에 있어서, 이미지를 격자 형태로 분할하고 각 격자의 중앙 값을 토큰으로 변환하는 방법을 주로 사용했다.

대표적으로 2가지로 나뉜다. 
1. 이미지를 일정한 크기의 격자(patch)형태로 분할->각 격자의 중앙에 위치한 값을 토큰으로 변환___*"An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale"* 
2.  이미지를 일정한 크기의 patch로 분할하는 대신에 이미지 featrue map을 set로 처리하여 transformer에 입력으로 제공하는 방식 제안 __*"End-to-End Object Detection with Transformers"*

Model architecture
![Screenshot from 2023-05-20 20-57-44](https://github.com/johook/Codingtest/assets/116954375/70e23a97-98b4-4e53-9361-3d613b43ac82)

(과정 요약)
1. Conv + Pool 연산으로 특성맵 추출 
2. 추출한 특성맵의 사이즈를 8 X 8로 압축 후 Transformer에 입력값으로 보냄
3. 각 데이터에서 보내준 8 X 8 크기의 특성맵 2개를 합체 = 16 X 8 사이즈의 특성맵 생성 
4. 16X8 벡터를 Positional Embedding 후 Linear layer를 이용해 자동차의 현재 속도를 Embedding vector에 projection 
5. Transformer에 넣어 self-attention 연산 => Embedding vector내 원소별로 전체 데이터(이미지 + LiDAR)에 대한 Attention이 반영됨. 사이즈는 16 X 8로 같음
6.  Attention이 반영된 Embedding vector를 Image, LiDAR별로 나눔 -> 8 X 8 사이즈의 벡터가 2개 생성 
7.  8 X 8 사이즈의 벡터를 압축하기 전의 크기로 scale up 
8.  원래 특성맵과(1에서 추출한 특성맵) element-wise summation(원소끼리 더함)
