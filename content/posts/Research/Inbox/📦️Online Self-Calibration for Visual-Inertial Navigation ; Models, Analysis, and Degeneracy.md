---
aliases: 
date: ""
title: 
author: 
tags:
  - Review
draft: true
---
[[🧩OpenVins.canvas|🧩OpenVins]]의 Analytical propagation 내용. 


zotero : 
source : https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=10145468

---
# Contribution
- Rolling shutter 카메라를 비롯한 IMU/camera intrinsics, extrinsics 포함 VINS observability analysis
- global xyz, yaw를 제외하고 모든 sensor calibration parameter는 observable하다, fully excited motion에서.
- imu camera intrinsic calibration에 있어 degenerate motion을 알려줌. 
### Critique
- IMU intrinsics를 왜 고려해야하지?

# Introduction
- mechanical 진동에 의해 IMU-cam sensor calibration이 달라질 수 있따. (extrinsics)
- IMU bias와 intrinsics는 온도, 습도에 영향을 받는다. 


- 어떤 문제를 풀 것인지

### Methodology
### Sensing Models
![[Pasted image 20241008173206.png]]

![[Pasted image 20241008173223.png]]
$D_w = T_w^{-1}, D_a = T_a^{-1}$

### Experiments
- 어떤 데이터
- 어떤 정보를 분석했는지 (ate, rpe, NEES)


### ❓️Questions

### Future Articles to Read

