---
aliases: 
date: ""
author: 정진용
draft: true
URL: https://jinyongjeong.github.io/2017/02/26/lec14_Least_square_SLAM_landmark/
---
#Optimization #LM
ref : 
- [정진용님 블로그](https://www.youtube.com/watch?v=I8wCohCAS60)
- [김태영님 블로그](https://www.youtube.com/watch?v=I8wCohCAS60)
- [유튜브-Cyrill](https://www.youtube.com/watch?v=mZBdPgBtrCM)

### Under-determined System
information matrix의 rank가 full rank가 아닌 경우를 under-determined system이라 하며, 우리의 state에서 일부만 unique solution으로 해를 얻을 수 있따.

따라서 이러한 상황을 해결하기 위한 방법이 **Levenberg-Marquardt** 방법이다.
즉, *damping factor*를 더함으로써 full rank의 matrix를 만들어 unique한 해를 찾는다.

damping factor를 Hessian matrix $\mathbf{H}$에 추가해줌으로써 $\lambda \mathbf{I}$ 가 system을 positive-definite을 만들어줌. ($\mathbf{I}$가 full rank라서)


## $(\mathbf{H}+\lambda\mathbf{I})\Delta x = -b$의 의미

$-b = - \mathbf{J}^T \Omega r$이다. (r은 residual function)
이 의미가 
1. information matrix $\Omega$ : residual을 가중치만큼 scaling 
2. $\mathbf{J}^T$ : 가중치가 곱해진 residual을 new parameter가 존재할 새로운 공간으로 mapping하는 것.


- $\mathbf{J}$: **Jacobian 행렬**로, 모델의 상태 변수(예를 들어, 로봇의 위치나 지도에 대한 파라미터)들이 잔차에 어떻게 영향을 미치는지를 나타냅니다. 즉, 상태 변수에 대한 잔차의 변화율을 나타냅니다.

- $\mathbf{J}^T$: **Jacobian 행렬의 전치**는 **잔차가 상태 변수에 미치는 영향**을 각 상태 변수 방향으로 재조정하는 역할을 합니다. 이 값은 각 상태 변수가 잔차를 줄이기 위해 얼마나 업데이트되어야 하는지를 보여줍니다
- 따라서, $b = \mathbf{J}^T \Omega$는 상태 변수를 업데이트하기 위한 **gradient(경사)** 를 제공하며, Jacobian 행렬의 전치는 **잔차가 각 상태 변수에 어떻게 반영되어야 하는지**를 나타냅니다
- 
## ❓️Questions
왜 근데 damping factor를 추가해서 rank-deficient한 상황을 해결할 수 있었을까?

## Link
[[🔎Optimization vs Filter]]
[[📦️X-ICP]]
[[🧩VINS-Fusion Map.canvas|🧩VINS-Fusion Map]]
[[📦️BASALT]]
[[📦️Square Root Marginalization for Sliding-Window Bundle Adjustment]]
[[📦️Is  Levenberg-Marquardt the Most Efficient Optimization Algorithm for Implementing Bundle Adjustment?]]

$$
\mathrm{rank}(J^TJ) = \mathrm{rank}(J^T)=\mathrm{rank}(J)
$$
