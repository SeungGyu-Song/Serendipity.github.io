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
- 유튜브

### Under-determined System
information matrix의 rank가 full rank가 아닌 경우를 under-determined system이라 하며, 우리의 state에서 일부만 unique solution으로 해를 얻을 수 있따.

따라서 이러한 상황을 해결하기 위한 방법이 **Levenberg-Marquardt** 방법이다.
즉, *damping factor*를 더함으로써 full rank의 matrix를 만들어 unique한 해를 찾는다.


## ❓️Questions
왜 근데 damping factor를 추가해서 rank-deficient한 상황을 해결할 수 있었을까?

## Link
[[🔎Optimization vs Filter]]
[[📦️X-ICP]]
[[🧩VINS-Fusion Map.canvas|🧩VINS-Fusion Map]]
[[📦️BASALT]]
[[📦️Square Root Marginalization for Sliding-Window Bundle Adjustment]]

