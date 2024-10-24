# 통계 표현
1. X가 정규분포 를 따른다 : X \sim \mathcal{N}(\mu, \sigma^2) → $X \sim \mathcal{N}(\mu, \sigma^2)$ 
# 행렬 표현
\begin{matrix} \end{matrix} 
1. matrix : 괄호 없음
2. **p**matrix : parentheses → ( )
3. **b**matrix : brackets → [ ]
4. **B**matrix : curly brackets → { }
5. **v**matrix : vertical bar brackets → | |
6. **V**matrix : double vertical bar brackes → || ||
## 원소 간 구분 방식
& : 열 구분
\\ : 행 구분

## 큰 행렬 표현 : 점점점

-  **`\cdots`** : 중앙 **가로** 점 3개
- **`\vdots`** : 중앙 **세로** 점 3개
- **`\ddots`** : 중앙 **대각선** 점 3개

$\begin{bmatrix} \cdots \end{bmatrix}$
# 글씨체
bold체 : `\textbf{bold text}` → $\textbf{bold text}$


## 수식 안에서
- bold체 : `\mathbf{Mat}` →$\mathbf{Mat}$
- 
- 캘리그래피 : `\mathcal{ABCD}` → $\mathcal{ABCD}$ :  quaternion L, R 연산을 이걸로 표시.
- 
- Fraktur 글씨체 :  `\mathfrak{Fraktur}` →$\mathfrak{Fraktur}$
- 
- Blackboard : `\mathbb{ABCD}` → $\mathbb{ABCD}$ : 집합

- 아마 외적표시 대충 : `\mathsf{x}` →$\mathsf{x}$

그 외
![[Pasted image 20240903150116.png]]

# 띄어쓰기
1. `\,` : 한 칸
2. `\;` : 두 칸
3. `\quad` : 네 칸(=tab)
4. `\qquad` : 여덟 칸 (tab * 2).

# 분수
1. `\frac{\pi}{3}` : $\frac{\pi}{3}$


# 기호
| LaTeX                         |                                           |                                                              |
| ----------------------------- | ----------------------------------------- | ------------------------------------------------------------ |
| \cdot                         | ·                                         | 곱셈기호 / 내적 (inner product)                                    |
| \circ                         | ∘                                         | circle / 원 (*degree 등을 나타내기 위한 위 첨자로 사용하려면 ^\circ를 이용하면 된다.) |
| \times                        | ×                                         | 곱셈기호 / 외적 (cross product)                                    |
| \pm                           | ±                                         | 플러스 마이너스 (plus minus)                                        |
| \circledast                   | ⊛                                         | convolution / 원 안의 별표 (asterisk)                             |
| \odot                         | ⊙                                         | 원 안의 곱셈기호                                                    |
| \oplus                        | ⊕                                         | 원 안의 덧셈기호                                                    |
| \otimes                       | ⊗                                         | 원 안의 곱셈기호                                                    |
| **관계성 기호 (Relation symbols)** |                                           |                                                              |
| \neq                          | ≠                                         | not equal / 같지 않음                                            |
| \geq                          | ≥                                         | greater than or eual to / 이상                                 |
| \leq                          | ≤                                         | less than or equal to / 이하                                   |
| \sim                          | ~                                         | similarity / 유사, 비슷                                          |
| \simeq                        | ≃                                         | asymptotic equality / 유사, 비슷                                 |
| \approx                       | ≈                                         | approximately equal / 유사, 비슷                                 |
| \propto                       | ∝                                         | proportional / 비례                                            |
| **화살표 (Arrows)**              |                                           |                                                              |
| \rightarrow                   | →                                         | right arrow / 오른쪽 화살표                                        |
| \leftarrow                    | ←                                         | left arrow / 왼쪽 화살표                                          |
| \uparrow                      | ↑                                         | up arrow / 위쪽 화살표                                            |
| \downarrow                    | ↓                                         | down arrow / 아래쪽 화살표                                         |
| \leftrightarrow               | ↔                                         | bidirectional arrow / 양방향 화살표                                |
| \Leftrightarrow               | ⇔                                         | bidirectional arrow / 양방향 화살표                                |
| **집합 기호 (Set operations)**    |                                           |                                                              |
| \cup                          | ∪                                         | union / 합집합                                                  |
| \cap                          | ∩                                         | intersection / 교집합                                           |
| \in                           | ∈                                         | 원소 포함                                                        |
| \notin                        | ∉                                         | 원소 포함되지 않음                                                   |
| \ni                           | ∋                                         | 원소 포함                                                        |
| \subset                       | ⊂                                         | subset / 부분집합                                                |
| \subseteq                     | ⊆                                         | subset / 부분집합                                                |
| \supset                       | ⊃                                         | supset, subset / 부분집합                                        |
| \supseteq                     | ⊇                                         | supset, subset / 부분집합                                        |
| \uplus                        | ⊎                                         | union & plus disjoint union of sets / 합집합 +                  |
| **미적분 (Calculus)**            |                                           |                                                              |
| \partial                      | ∂                                         | partial derivative / 편미분 기호                                  |
| \nabla                        | ∇                                         | nabla / del / 미분 기호 / 역삼각형                                   |
| \Delta                        | Δ                                         | large delta / 미분 기호 / 삼각형                                    |
| \int                          | ∫                                         | integral / 적분                                                |
| **Others**                    |                                           |                                                              |
| \angle                        | ∠                                         | angle / 각                                                    |
| \therefore                    | ∴                                         | therefore                                                    |
| \because                      | ∵                                         | because                                                      |
| \lfloor x \rfloor             | $\lfloor \mathbf{a} \; \mathsf{x}\rfloor$ | 외적 표시할 때 쓰임.                                                 |


![Greek문자](https://cdn2.ppomppu.co.kr/zboard/data3/2021/1129/20211129082355_urgmdzxv.png)


