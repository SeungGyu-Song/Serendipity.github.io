---
aliases: 
date: 2024-10-24
author: Jerry Hsiung
draft: true
URL:
---

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

## Experiments
- 어떤 데이터
- 어떤 정보를 분석했는지 (ate, rpe, NEES)


## ❓️Questions

### Future Articles to Read
Nonlinear factor recovery for long-term SLAM (IJRR)

## Link
[[📦️BASALT]]
[[📦️VINS-Fusion Review]]