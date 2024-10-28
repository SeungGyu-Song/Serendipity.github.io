---
aliases: 
date: ""
author: 
draft: false
URL:
---
## KeyIdea
- landmark 지우기 using null-space projection
- marginalization prior를 저장하기 위해 Hessian에 square root를 씌움.
- old pose들을 marginalization할 때, *specialised QR decomposition*을 이용해 square root prior를 update함.

→ baseline보다 36% 빠르고, pseudo inverse jacobian을 다룸.

## Critique
- 얻어갈 것
- 이 방법의 장점 / 부족한 점
- 어떻게 써먹을 수 있을지

## Introduction

- 어떤 문제를 풀 것인지

## Methodology
### 4. Square root marginalization
Hessian matrix의 landmark - landmark sparsity를 위해 marginalized될 landmark의 active window 안에 있는 observation을 그냥 drop함.

nullspace projection을 사용하는데 frame variable한테는 이게 optimal 하지 않다. 그래서 두 차례에 걸쳐서 사용할 예정임.

#### 4.3.2 Rank-deficient Jacobian
아마 Marginalization term에 대해서만 말하는 거 같은데, 아래와 같은 상황에서 rank-deficient jacobian 발생 가능
- system이 initial prior에 의해 잘 constrained되지 않았을 때
- operation 동안 prior가 numeric noise보다 작아질 때

## Experiments
- 어떤 데이터
- 어떤 정보를 분석했는지 (ate, rpe, NEES)


## ❓️Questions

### Future Articles to Read

