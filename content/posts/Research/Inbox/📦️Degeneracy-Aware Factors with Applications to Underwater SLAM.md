---
aliases: 
date: 2024-10-24
author: 
draft: true
URL:
---
#Degeneracy 
## Critique
#### 전 과의 차이점 
Methodology 참고
#### 보완할 점
그냥 well-constraint한 방향만 최적화하면 나머지 방향으로는 값이 안 변하는 거 아닌가? 이 부분을 좀 더 생각해보면 좋을 것 같다.

## Methodology
- degeneracy한 환경에서 degenerate한 방향을 제외하고 optimization을 하자
- 
- [[📦️On degeneracy of optimization-based state estimation problems|저장대학교 논문]]의 후속으로, 심하게 degenrate 환경에서 얘네의 가정이 깨질 수 있다.
- 그래서 매 iteration마다 eigen value의 Max / min의 비율로 thresholding해서 degenrate 방향을 정함.
- 이후 degenrate한 방향은 최적화를 안 하고  well-constraint인 부분만 최적화.



### Future Articles to Read
ICP에 대해서 읽어보자. 여기 논문에서 ICP의 Ax=b에서 A를 linearized system이라고도 하고, $A^TA$를 covariance라고 하는 게, jacobian을 써서 얘도 linearization을 한 건가?
