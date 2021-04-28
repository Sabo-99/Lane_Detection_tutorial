# RONELD(Robust Neural Network Output Enhancement for Active Lane Detection)  
해당 기법은 확률 맵(probability map)으로부터 active lances들에 대해 식별, 추적, 최적화를 진행한다. (FP 확률을 떨어뜨림)  
먼저, 확률 맵으로부터 차선 points들을 추출한 다음, 직선 차선에서 가중치가 적용된 최소 제곱 선형 회귀(leaset squares linear regression)를 사용하기 전에 곡선 및 직선 차선을 감지하여
실제 이미지에서 에지 맵 깨짐현상(frgamentation)으로 인한 차선을 수정하였다. 마지막으로, 이전 프레임을 추적하여 실제 active lanes를 가정한다.  

--
## Introduction  
Lane Detection에서 FP 확률이 높을수록 자율주행 시 제어 불안정성을 초래한다. 해당 논문에서는 학습된 데이터가 아닌 데이터에 대해서 정확히 예측하기 위해 robust한 모델을 제안한다.  

## Keypoints  
##### 1) Adaptive Lane Point Extraction  
딥러닝 모델(SCNN, ENet-SAD)으로부터 얻은 확률 맵에서 차선 지점(feature)을 추출한다. Adaptive Threshold를 통해 두드러진(salient) 지점만 추출하여 robustness 확보와 
low-confidence noise 제거를 진행한다. 이 때 추출된 지점 주변 영역(검색 영역)에 집중하기 때문에 처리 시간이 단축된다.  
검색 영역 내 차선에서 발견된 가장 높은 신뢰 지점을 차선 지점으로 사용한다.
##### 2) Curved Lane Detection  
탐지된 차선은 곡선 차선과 직선 차선으로 나뉜다. 선형 모델을 이용하여 직선 차선에 대한 확률 맵 결과 내에 끊어진(또는 검출되지 않은) 차선 가장자리를 개선하기 위해 다음과 같이 수행한다.  
