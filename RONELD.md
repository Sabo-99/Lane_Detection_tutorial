# RONELD(Robust Neural Network Output Enhancement for Active Lane Detection)  
해당 기법은 확률 맵(probability map)으로부터 active lances들에 대해 식별, 추적, 최적화를 진행한다. (FP 확률을 떨어뜨림)  
먼저, 확률 맵으로부터 차선 points들을 추출한 다음, 직선 차선에서 가중치가 적용된 최소 제곱 선형 회귀(leaset squares linear regression)를 사용하기 전에 곡선 및 직선 차선을 감지하여
실제 이미지에서 에지 맵 깨짐현상(frgamentation)으로 인한 차선을 수정하였다. 마지막으로, 이전 프레임을 추적하여 실제 active lanes를 가정한다.  

---
## Introduction  
Lane Detection에서 FP 확률이 높을수록 자율주행 시 제어 불안정성을 초래한다. 해당 논문에서는 학습된 데이터가 아닌 데이터에 대해서 정확히 예측하기 위해 robust한 모델을 제안한다.  

## Keypoints  
##### 1) Adaptive Lane Point Extraction  
딥러닝 모델(SCNN, ENet-SAD)으로부터 얻은 확률 맵에서 차선 지점(feature)을 추출한다. Adaptive Threshold를 통해 두드러진(salient) 지점만 추출하여 robustness 확보와 
low-confidence noise 제거를 진행한다. 이 때 추출된 지점 주변 영역(검색 영역)에 집중하기 때문에 처리 시간이 단축된다.  
검색 영역 내 차선에서 발견된 가장 높은 신뢰 지점을 차선 지점으로 사용한다.
##### 2) Curved Lane Detection  
탐지된 차선은 곡선 차선과 직선 차선으로 나뉜다. 선형 모델을 이용하여 직선 차선에 대한 확률 맵 결과 내에 끊어진(또는 검출되지 않은) 차선 가장자리를 개선하기 위해 다음과 같이 수행한다.  
직선 차선의 경우, noise 영향을 줄이기 위해 최소 n=3개 차선 지점을 필요로 한다. 곡선 차선의 경우, 복잡성이 크기 때문에 최소값 3n을 정의한다. 두 범주를 구별하기 위해 결정 계수 r^2을 사용하여 해당 지점이 선형 모델에 얼마나 잘 맞는지 평가한다. 전체(whole) 차선 표시와 상위 n개 지점이 없는(끊어진) 차선 표시 사이의 결정 계수 r^2을 비교한다.  

![CodeCogsEqn](https://user-images.githubusercontent.com/54304718/116353518-e132d180-a831-11eb-88ef-9c28ffbeaec8.gif)
(X,Y는 탐지된 차선 지점의 x 및 y 성분을 나타내는 랜덤변수)  

전체 차선 표시 r^2이 끊어진 차선 표시 r^2보다 작으면 해당 차선 표시가 선형 모델에 적합하지 않다는 뜻이다. 상위 n개 차선 표시가 회귀선(regression line)을 벗어난다는 것을 의미하기도 한다. 따라서 곡선 차선으로 표시한다. 반대의 경우 직선 차선으로 표시한다.  
탐지된 차선 표시에서 잘못된 곡선 예측을 줄이기 위해 현재 frame 곡선에 이전 frame 곡선을 확증(뒷받침)한다.   
##### 3) Lane Construction  
예측된 곡선 차선에는 최종 차선 표시 출력을 형성하기 위해 2차 spline으로 연결된 점이 있다. 직선 차선의 경우, 확률 맵 출력에서 끊어진 차선 edge를 수정하고 아래 형식의 선형 모델을 기반으로 감지된 차선 표시를 고려하여 이상값(outlier)을 제거하고자 한다.  

![CodeCogsEqn (1)](https://user-images.githubusercontent.com/54304718/116360559-0415b380-a83b-11eb-8887-3956a885d34d.gif)  
![CodeCogsEqn (2)](https://user-images.githubusercontent.com/54304718/116360614-12fc6600-a83b-11eb-97ff-2bb8fe553da6.gif)  
![CodeCogsEqn (3)](https://user-images.githubusercontent.com/54304718/116360673-1db6fb00-a83b-11eb-82ae-d4101c71dbcd.gif)  

(x_i,y_i)는 i번째 탐지된 차선 포인트의 x와 y 좌표를 나타내고, (beta_0, beta_1)는 y-intercept와 line의 gradient를 나타니며, m은 탐지된 차선 points의 수이다.  
이러한 차선 매개 변수를 사용하여 딥러닝 모델 확률 맵 output에서 탐지되지 않은 차선 edge로 인해 깨진(gragmentation) 차선 표시를 수정할 수 있다.  
##### 4) Tracking Preceding Frames  
날씨, 조명과 같은 도로 상태가 다양한 주행 환경에서는 정확한 차선 탐지를 위해 현재 프레임의 정보가 부족할 수 있다. 확률 맵 출력에서 잘못 식별된 lane으로 인해 발생하는 왜곡을 최소화하기 위해 (이전 frame의 lane을 tracking) + (현재 frame의 lane을 mapping) 하여 안정적이고 강력한 active lane을 가정한다. 현재 frame과 이전 frame 간의 RMS x-distance L(L_1,L_2)를 계산한다.  

![CodeCogsEqn (4)](https://user-images.githubusercontent.com/54304718/116361794-53a8af00-a83c-11eb-9d6f-0d6b8454c016.gif)
, 같은 조건이면 같은 차선으로 고려한다.  

딥러닝 모델 출력을 기반으로 잠재적인 데이터 active lane 표시를 식별하고, 우선 순위를 통해 높은 값을 나타내는 lane 표시들을 정렬한다. 또한, lane 표시가 active lane 표시로 잘못 식별될 수 있으므로, 식별된 lane 표시 외에도 현재 frame에 non-active lane 표시를 저장한다. (FP lane 표시로 인해 실제 active lane 표시가 non-active lane 표시로 잘못 분류되는 것을 방지한다.)  


##### Reference  
[1] https://arxiv.org/pdf/2010.09548.pdf  
[2] https://github.com/czming/RONELD-Lane-Detection  
[3] https://go-hard.tistory.com/81
