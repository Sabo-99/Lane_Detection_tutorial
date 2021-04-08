# Key Points Estimation and Point Instance Segmentation

학부생 종업 설계 과제로 차선 검출 및 디스플레이를 주제로 정하였다. 열악한 환경 (ex.안개, 비, 야간, 빛번짐 등)에서 차를 운전할 때 차선이 보이지 않아 위험한 경우가 많다.
이를 해결하기 위해 헤드업 디스플레이(HUD) 기술에 사용된 방법을 이용하고자 한다.  
기존 차량 주행 보조 장치(ADAS)에 사용되는 HUD는 단순히 네비게이션 또는 속력 측정계 화면을 띄운다. 그러나 이를 운전자 시야가 아닌 차량 앞유리 전체에 맞춰 차선을 표시한다면 위의 열악한 환경에서도 
운전자들의 차선 시야 확보가 되지 않을까. 또한 딥러닝을 사용하여 단순 차선 인식이 아닌 차선이 지워진 부분에도 표시가 가능하다면 좋지 않을까.  
  
Paperswithcode 사이트에서 lane detection 네트워크 중 가장 가벼운 모델을 선정하였다. (더 가벼운 모델이 있을수도 있으나 우선 PINet을 살펴보고자 한다.)  

---
## Abstract  
##### 기존 sota 기술 문제점  
1. 탐지 가능한 차선 개수 제한
2. FP 비율 높음 (불안정한 자율 주행)
  
##### PINet에서 제안점  
1. (feature extraction 단계) Stacked Hourglass 방법 기반 -> 차선의 정확한 point 추정
2. (추정된 point들) instance별 clustering 문제는 point-cloud cluster instance segmentation으로 해결
3. 제안된 post-processing 방법을 통해 outlier 제거
---
## Introduction  
_RGB 카메라로 차선 탐지 방법을 많이 사용하므로 해당 센서 데이터 이용_  
  
##### Semantic Segmentation
각 lane을 구별하는 multi-class 접근 방식으로 고정된 lane 수로 구성된 장면에만 사용 가능  
∴) 입력 pixel 크기보다 lane에 대해 더 적은 양의 정확한 point 예측 & 각 point를 instance로 구별  

##### 3 values (feature extraction 출력값)  
_(차선의 정확한 위치와 point에 대한 instance 단위 특징들 예측)_  
  1. Confidence : Grid 단위 예측값(각 grid에서 다음 block 이동시 안정적 학습 가능하도록 함)
  2. Offset : 차선이 위치한 정확한 point 좌표
  3. Feature : 차선 instance 단위 정보

##### 제안하는 개선 사항  
  1. compact output 출력 (모델 경량화)
  2. post-processing으로 outlier 제거
  3. 수평, 수직, 임의의 lane 수에 따른 차선 탐지
  4. FP 수치 떨어뜨림 (안정적인 자율 주행 수행)  
---
## Method
<img src="https://img1.daumcdn.net/thumb/R720x0.q80/?scode=mtistory2&fname=http%3A%2F%2Fcfile8.uf.tistory.com%2Fimage%2F99D10E405EE5E17113AD3C" width="600" height="305">

##### 1. Resizing Layer  
- 입력 이미지 크기 ↓ (512x256 → 64x32)을 통해 연산량 ↓
- convolution 연산과 maxpooling 사용  
(conv → same bottleneck → maxpooling → same bottleneck → maxpooling → same bottleneck)  

##### 2. Feature Extraction Layer  
- Stacked Hourglass : 총 2개의 hourglass block(down-sample bottleneck + up-sample bottleneck)으로 구성
- bottleneck layer를 활용하였기에 적은 연산량으로도 높은 성능을 보임
- skip layer를 통해 다양한 크기의 디테일한 특징들을 유지
- 차선 특징을 압축해서 예측  

##### 3. Output Layer  
- 각 grid 단위 3 values(confidence/offset/feature) 예측  

<**Loss Function**>  
1. confidence (각 grid 안 신뢰도 score)  
![CodeCogsEqn](https://user-images.githubusercontent.com/54304718/113986391-2a71b000-9888-11eb-81fb-82457728c0d4.png)
  - Ne, Nn : grid 안의 point가 존재하는(e), 존재하지 않는(n) grid의 개수
  - Ge, Gn : grid 세트 수
  - gc* : ground truth value (1: key point 포함, 0: 미포함)
  - gc : predicted vlaue of each cell in the confidence output  

2. offset (각 point (x,y) 위치값 (0~1))
