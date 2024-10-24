OpenVINS Study 5주차에는 IMU propagation에 관해 다룰 예정입니다.
4주차와는 다르게 실제 EKF를 이룬

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
- [Discrete Propagation](https://docs.openvins.com/propagation_discrete.html) : fast state propagate , IMU intrinsics 고려 X
-  [Indirect Klaman Filter for 3D Attitude Estimation](https://mars.cs.umn.edu/tr/reports/Trawny05b.pdf) : quaternion 배경 지식
- [Online Self-Calibration for Visual-Inertial Navigation: Models, Analysis, and Degeneracy](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=10145468) : IMU intrinsics를 고려한 VINS 
- **[Analytical Propagation](https://docs.openvins.com/propagation_analytical.html) : EKF propagation, IMU intrinsics 고려 O**
- **[On-Manifold Preintegration for Real-Time Visual-Inertial Odometry](https://rpg.ifi.uzh.ch/docs/TRO16_forster.pdf) : SO(3) 배경지식** 
---
# Code

## EKF Propagation
- [[🧩OpenVINS Code Analysis#do_feature_propagate_update|VioManager::do_feature_propagate_update]] 
	- [[🧩OpenVINS Code Analysis#propagate_and_clone|Propagator::propagate_and_clone]]
		- [[🧩OpenVINS Code Analysis#predict_and_compute|Propagator::predict_and_compute]]
			- **[[🧩OpenVINS Code Analysis#compute_Xi_sum|Propagator::compute_Xi_sum]] :  [Analytical Propagation](https://docs.openvins.com/propagation_analytical.html)에서 $\Xi$ 구하는 부분으로 예상** 
			- **[[🧩OpenVINS Code Analysis#predict_mean_analytic|predict_mean_analytic]]  or   [[🧩OpenVINS Code Analysis#predict_mean_rk4|predict_mean_rk4]]   or   [[🧩OpenVINS Code Analysis#predict_mean_discrete|predict_mean_discrete]]** 
			- **[[🧩OpenVINS Code Analysis#compute_F_and_G_analytic|Propagator::compute_F_and_G_analytic]]**
# Class
[[🧩OpenVINS Code Analysis#Propagator|Propagator]]

---
