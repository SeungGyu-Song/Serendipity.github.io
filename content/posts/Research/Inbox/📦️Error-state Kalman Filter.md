---
aliases:
  - eskf
date: 2024-10-14
title: Error-state Kalman Filter
URL: https://gsk1m.github.io/slam/2022/09/23/kf-tutorial10.html
tags: 
draft: true
---
`## Objective
- OpenVINS, FAST-LIO 등 최근 filter 기반 논문에서 eskf를 베이스로 하고있다.
- Error state Kalman filter를 왜 사용해야하는지, 장점에 대해 알아보자
- **결론적으로 일반 EKF보다 Manifold 특성을 고려한 연산이 된다는 장점이 있다.**

1. 근데 그냥 EKF하고 어차피 선형화되는 지점만 다른 건데, 왜 Lie group / Lie algebra를 활용하는 데 용이하다는 걸까? 
	1. 그럼 실제 optimization 방식에서도 error state처럼 해야 이득 아닌가? #점검 
2. position, velocity, bias도 error state에서 하는 거 같은데,  landmark는 어떻게 다뤄줘야할까? #점검



## Introduction
#### Kalman Filter 
- prior와 measurement 사이의 weighted sum을 해주는 장치
- 그냥 고정된 상수로 융합해주는 게 아니라, a posteriori의 covariance를 최소화하는 것을 cost로 해서 최적 gain을 결정하는 방법.
- Kalman filter는 Bayesian Filter를 기반으로 하고있고, Bayesian filter의 철학은 현재의 최적값이 다음 턴의 사전정보가 된다는 것이다. 따라서 다음 턴의 prior는 현재의 최적 정보가 되겠다.

### EKF와 ESKF
1. state가 nonlinear하거나, 
2. measurement model이 non-linear 
하다면 시스템에 non-linearity가 발생하고, 이를 Taylor expansion을 통해 근사를 할 수 있따.

한편, KF도 covariance라는 *squared loss*를 cost로하는 estimator인 것이다. 
이 점에서 least-square를 최적화하는 방식인 Gauss-Newton과 비슷하다고 할 수 있다.


### GN에 대해서
![[Pasted image 20241014153928.png]]
결국에는 $J^{\top} J \Delta x = -J^{\top} b$ 를 푸는 것. (우변에 대해서는 실제로 코드에서는 어떻게 되는 걸까? )

블로그 포스트에 있는 [Grisetti 교수님의 ICRA tutorial 자료](https://www.diag.uniroma1.it//~labrococo/tutorial_icra_2016/icra16_slam_tutorial_grisetti.pdf#page=3.00)에서 최종적으로 하고자 하는 말은 :
**state에 nonlinear한 element가 있다면, 그것의 tangent space에서 update해주어야한다.** 

예로, rotation의 경우, 3차원 공간에서 rotation은 보통 SO(3)로 표현이 되고, 이는 3x3 matrix이다. 이를 최적화 하기 위해 9-dimension  vector로 만들고 이 9개의 값들을 조금씩 조절해가면서 cost를 줄여나간다는 것은 안된다는 것이다.
이런 non-linear entity들은 vector space가 아니므로, 기존의 기계적인 GN pipeline에 바로 적용될 수 없다.

아래 그림은 [Grisetti 교수님의 ICRA tutorial 자료](https://www.diag.uniroma1.it//~labrococo/tutorial_icra_2016/icra16_slam_tutorial_grisetti.pdf#page=3.00)에서 가져온 건데, 아직 잘 이해가 안 됐다. 
여기서 회전을 $[-\pi, \pi]$ 로 정의를 했는데, 아래와 같이 바꿔서 해야한다는 게 잘 이해가 안된다. #점검 
![[Pasted image 20241014164408.png]]
### ESKF 알아보기

ESKF도 GN방법과 같이, 
1. non linear한 rotation이 state vector에 있으니, 
2. minimal reparameterization(3차원)하고, 이 공간에서 KF update를 수행.
3. 즉, error-state란, GN과 비슷하게, tangent space에서 살고있는 변수로 이해하면 된다.


#### EKF vs ESKF
[A micro Lie theory for state estimation in robotics](https://arxiv.org/pdf/1812.01537)에서 언급했듯,
1. manifold와 tangent space 사이의 연산을 할 수 있는 oplus 사용
2. Jacobian을 tangent space에서 계산

이 외로, 
1. prediction step : error state에 대해 process model을 세워주어야한다. 이를 이용해서 error state의 covariance를 propagate해준다.
2. correction step : Jacobian을 nominal state에 대해 구하는 것이 아닌, error state에 대해 구하는 것이 EKF와 다르다. 결과적으로, Kalman gain $K$ 가 tangent space에서 weight blending을 잘 할 수 있도록 만들어진다.


Tangent space에서의 Jacobian 계산법
![[Pasted image 20241014165658.png]]
1. 기존처럼 x에 관한 미분에 추가적으로 $\delta x$에 관한 미분까지 늘어나 2개의 미분식이 된 것을 볼 수 있다.
2. 흥미롭게도, 279번 식을 보면, Rotation term 외에는 모두 identity 임을 알 수 있다. 그 이유는 translation은 vector space이기 때문에 원래 공간과 tangent space가 동일해서 이와 같은 결과를 보여준다.



## Future Work
- [Kalman Filters on Differentiable Manifolds](https://arxiv.org/pdf/2102.03804) Fu Zhang
- [A micro Lie theory for state estimation in robotics](https://arxiv.org/pdf/1812.01537) : Joan Sola,  교수님도 추천해줬던 논문


### ❓️Questions

