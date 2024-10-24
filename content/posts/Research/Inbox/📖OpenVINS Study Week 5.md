OpenVINS Study 5주차에는 IMU propagation에 관해 다룰 예정입니다.
4주차와는 다르게 실제 EKF를 이루는 두 단계 (Propagation과 Update) 중 Propagation에 해당하는 부분이 되겠습니다.

<목차> 
- Introduction / Heads up for reading the codes
- Feature Extraction / Tracking (Opticalflow / Descriptor)
- initialization (Static / Dynamic)
- IMU fast propagation / EKF propagation part 1
- **IMU fast propagation / EKF propagation part 2**
- EKF Update part 1
- EKF Update part2
---
## Reference
-  [Indirect Klaman Filter for 3D Attitude Estimation](https://mars.cs.umn.edu/tr/reports/Trawny05b.pdf) : quaternion 배경 지식
- [Online Self-Calibration for Visual-Inertial Navigation: Models, Analysis, and Degeneracy](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=10145468) : IMU intrinsics를 고려한 VINS 
- **[Analytical Propagation](https://docs.openvins.com/propagation_analytical.html) : EKF propagation, IMU intrinsics 고려 O**
- **[On-Manifold Preintegration for Real-Time Visual-Inertial Odometry](https://rpg.ifi.uzh.ch/docs/TRO16_forster.pdf) : SO(3) 배경지식** 

#### EKF 형태 훑어보기에 좋은 자료
- [ALIDA님 블로그](https://alida.tistory.com/54) 
	- https://arxiv.org/pdf/2406.06427 : 위 블로그 내용을 pdf로 변환해서 올린 자료
- [김기섭님 블로그](https://gsk1m.github.io/slam/2022/09/23/kf-tutorial10.html) : error state kalman filter에 대한 내용
	- https://gsk1m.github.io/slam/2022/09/23/kf-tutorial10.html
	- [[📦️Error-state Kalman Filter]]

---
# Code

## EKF Propagation
- [[🧩OpenVINS Code Analysis#do_feature_propagate_update|VioManager::do_feature_propagate_update]] 
	- [[🧩OpenVINS Code Analysis#propagate_and_clone|Propagator::propagate_and_clone]]
		- [[🧩OpenVINS Code Analysis#predict_and_compute|Propagator::predict_and_compute]]
			- **[[🧩OpenVINS Code Analysis#compute_Xi_sum|Propagator::compute_Xi_sum]] :  [Analytical Propagation](https://docs.openvins.com/propagation_analytical.html)에서 $\Xi$ 구하는 부분으로 예상** 
			- **[[🧩OpenVINS Code Analysis#predict_mean_analytic|predict_mean_analytic]]  or   [[🧩OpenVINS Code Analysis#predict_mean_rk4|predict_mean_rk4]]   or   [[🧩OpenVINS Code Analysis#predict_mean_discrete|predict_mean_discrete]]** 
			- **[[🧩OpenVINS Code Analysis#compute_F_and_G_analytic|Propagator::compute_F_and_G_analytic]]**

`VioManager::do_feature_propagate_update` 함수에서는 342번 째 줄인 `propagator->propagate_and_clone` 함수 호출부분까지만 보시면 됩니다.
# Class
[[🧩OpenVINS Code Analysis#Propagator|Propagator]]

---
Kalman filter에 대해서 이해를 하시고 오신다면 다음 주부터 시작될 update 부분에 더 잘 이해가 될 것으로 예상됩니다.

