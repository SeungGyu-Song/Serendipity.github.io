---
aliases: 
date: 2024-10-24
author: Jerry Hsiung
draft: true
URL:
---
이거 [[📦️BASALT]]의 contribution과 상당히 유사하다.  mapping에 관련해서 그냥 sparsity를 사용한 듯. 
Marginalization을 하게 되면, prior term이 굉장히 dense해지는 것을 알고있자.
## Critique
- 얻어갈 것
- 이 방법의 장점 / 부족한 점
- 어떻게 써먹을 수 있을지

## Introduction

- SLAM에서 효율적인 계산을 위해 Sparsity가 중요하다.
- Marginalization은 linear prior term matrix를 dense하게 만들고, 이거는 accuracy와 efficiency를 떨어뜨린다.
- 따라서 sparsity를 가짐과 동시에 원래의 특성을 가지고 있는 matrix로 recovery시키자.

## Methodology
Marginalization 전략은 VINS와 비슷함.

KL Divergence를 이용해서 원래의 dense한 prior를 근사함. nonlinear factor들을 이용해서.


### IV - B. Information Sparsification
$$
\Lambda_{(\mathrm{MB)}} = \begin{bmatrix} \Lambda_{{\chi_{R}}{\chi_{R}}} \qquad \Lambda_{{\chi_{M}}{\chi_{R}}} \\ \Lambda_{{\chi_{R}}{\chi_{M}}} \qquad  \Lambda_{{\chi_{M}}{\chi_{M}}}   \end{bmatrix} 
$$

$\Lambda_t = \Lambda_{{\chi_R}{\chi_R}}- \Lambda_{{\chi_R}{\chi_M}} {\Lambda_{{\chi_M}{\chi_M}}}^{-1} \Lambda_{{\chi_M}{\chi_R}}$

$\chi_w = {K_w, F_w, L_w}$ 

dense prior information $\Lambda_t$은 다음과 같이 multivariate Gaussian을 따름 $p(\chi_t) \sim \mathcal{N}(\mu_t, \Lambda_t)$
1. NFR 논문처럼 처음에 Markov blanket에 대해 factor graph topology를 만들건데 아래처럼 $p_s(\chi_t)$를 $p(\chi_t)$와 (원래 것과) 유사해지게 만드는 거임 ($\Sigma_t = \Lambda_t^{-1}$)
![[Pasted image 20241024203318.png]]  

## Experiments
- Open dataset (EuRoC)으로 테스트 진행했고, 
- 연산량이 많이 걸린다는 단점이 있다. → parallelization으로 극복  가능
	- 주로 KLD optimization에서 소요.
- 다른 알고리즘보다 성능이 낮은 거는 blurred image이거나 dynamic lighting change 때문임.

## ❓️Questions

### Future Articles to Read
Nonlinear factor recovery for long-term SLAM (IJRR)

## Link
[[📦️BASALT]]
[[📦️VINS-Fusion Review]]