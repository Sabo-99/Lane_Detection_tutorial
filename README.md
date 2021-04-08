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
2. FP 비율 높음 : 불안정한 자율 주행
  
##### PINet의 개선 사항  
1. (feature extraction 단계) Stacked Hourglass 방법 기반 -> 차선의 정확한 point 추정
2. (추정된 point들) instance별 clustering 문제는 point-cloud cluster instance segmentation으로 해결
3. 제안된 post-processing 방법을 통해 outlier 제거
---
## Introduction  
_RGB 카메라로 차선 탐지 방법을 많이 사용하므로 해당 센서 데이터 이용_  
  
##### Semantic Segmentation
각 lane을 구별하는 multi-class 접근 방식으로 고정된 lane 수로 구성된 장면에만 사용 가능  
∴) 입력 pixel 크기보다 lane에 대해 더 적은 양의 정확한 point 예측 & 각 point를 instance로 구별  

* 3 values (feature extraction 출력값)
_(차선의 정확한 위치와 point에 대한 instance 단위 특징들 예측)_  
  1. Confidence : Grid 단위 예측값(각 grid에서 다음 block 이동시 안정적 학습 가능하도록 함)
  2. Offset : 차선이 위치한 정확한 point 좌표
  3. Feature : 차선 instance 단위 정보

