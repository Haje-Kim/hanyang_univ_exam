# 컴퓨터 알고리즘 (임을규) — 강의 자료 추출본
> 출처: 강의자료 PDF 16개 + Notion Day 9개 | 추출일: 2026-06-02
> 교재: Cormen et al., *Introduction to Algorithms* (CLRS). PDF가 1차 자료, Notion Day 노트는 시험 범위·강조점 보강.

## 0. 과목 개요
- **푸는 문제**: "주어진 계산 문제를 어떤 절차로, 얼마나 효율적으로(시간·공간) 풀 수 있는가"를 점근적 복잡도로 분석하고, 정렬/그래프/최적화/암호 문제에 대한 표준 알고리즘과 그 정당성(증명)을 익힌다.
- **큰 줄기**: ① 점근 표기·복잡도 분석 → ② 분할정복·재귀 점화식 → ③ 정렬(비교/비비교) → ④ DP·그리디 → ⑤ 분할상환 → ⑥ 그래프(탐색·MST·최단경로) → ⑦ 정수론·공개키 암호(RSA).
- **평가 핵심(SKILL 도메인 힌트 + Notion + summary 자료 종합)**: 증명·복잡도 유도가 시험 핵심. summary 강의(PDF 16)는 교수가 직접 손글씨로 강조 표시한 **시험 리뷰 세션**으로, 출제 범위의 강력한 신호다.
- **주의(범위 신호)**: Master Method(마스터 정리)는 **다루지 않음**(Notion Day 2 + PDF 03 — 형태 `aT(n/b)+f(n)`만 제시, 3 case 미전개). 일부 절은 강의에서 건너뜀(아래 §1·§4 기록).

---

## 0-1. 선행 지식 키워드 (배경지식 — 본문에서 설명 없이 전제되는 것)

> 본문(§2 개념 카탈로그)이 **설명 없이 가정**하는 기초만 선별(과목 내부 노드 C#와 중복 금지 — 토대만). 판별: "이걸 모르면 어느 노드에서 막히는가?"

### PK1. [수학] 로그·지수와 lg(=log₂)
- **쉬운 정의**: 로그는 "몇 번 곱해야 그 수가 되나"의 역연산. $\lg n=\log_2 n$ = n을 2로 몇 번 나눠야 1이 되나. 지수 $2^h$는 그 반대로 h번 곱하기.
- **왜 필요**: 분할정복이 입력을 반으로 줄이는 횟수가 $\lg n$(트리 높이), 이진트리 잎 수 $\le 2^h$ 등 거의 모든 복잡도가 로그/지수로 표현된다.
- **출처/연결**: C1(점근표기)·C2(점화식 재귀트리 높이)·C6(하한 $n!\le2^h$).

### PK2. [수학] 재귀와 점화식
- **쉬운 정의**: 함수가 자기 자신을 더 작은 입력으로 부르는 것(재귀). 그 수행시간을 자기 참조로 쓴 식 $T(n)=2T(n/2)+n$이 점화식.
- **왜 필요**: 분할정복·DP의 비용을 점화식으로 세우고 푸는 것이 C2의 전부.
- **출처/연결**: C2(점화식 풀이)·C3·C5(분할정복).

### PK3. [수학] 수학적 귀납법
- **쉬운 정의**: "n에서 성립하면 n+1에서도 성립 + 출발점 성립" → 모든 n에 성립을 보이는 증명 틀.
- **왜 필요**: 치환법(C2)·알고리즘 정당성 증명이 모두 귀납으로 전개된다.
- **출처/연결**: C2(치환법 귀납단계·경계조건).

### PK4. [수학] 부등식·극한 직관 (충분히 큰 n)
- **쉬운 정의**: $f(n)\le cg(n)$ 같은 부등식을 다루고, "n이 커질수록 $1/n\to0$" 같은 극한 직관으로 항을 버리는 감각.
- **왜 필요**: 점근 정의가 "$\forall n\ge n_0$"의 부등식이라, 작은 항을 버리고 상수배를 잡는 데 필수.
- **출처/연결**: C1(c·n₀ 찾기), C9(등비급수 $\sum n/2^i=2n$).

### PK5. [수학] 등비급수 합
- **쉬운 정의**: $\sum_{i=0}^{\infty} r^i = \frac{1}{1-r}$ ($|r|&lt;1$). 예: $\sum 1/2^i=2$.
- **왜 필요**: BUILD-MAX-HEAP O(n)·binary counter amortized O(1) 유도가 등비합 수렴에 의존.
- **출처/연결**: C4(왜 O(n)), C9(총 flip $&lt;2n$).

### PK6. [수학] 순열·조합과 n!
- **쉬운 정의**: n개를 줄 세우는 경우의 수 = $n!=n(n-1)\cdots1$. Stirling 근사로 $\lg(n!)=\Theta(n\lg n)$.
- **왜 필요**: 비교정렬 하한 증명이 "정렬 결과는 n!가지"에서 출발한다.
- **출처/연결**: C6(decision-tree 하한 $n!\le2^h$).

### PK7. [수학] 모듈러 산술 (mod)
- **쉬운 정의**: $a \bmod n$ = a를 n으로 나눈 나머지. 시계 산술(12시 +3 = 3시)처럼 일정 수로 감싸는 연산.
- **왜 필요**: 정수론·RSA·Diffie-Hellman의 모든 계산이 mod n 위에서 이뤄진다.
- **출처/연결**: C17·C18·C19(암호 전부).

### PK8. [CS기초] 자료구조 기본 — 배열·연결리스트·스택·큐
- **쉬운 정의**: 배열(인덱스 즉시접근), 연결리스트(노드 사슬), 스택(LIFO, 마지막 넣은 것 먼저), 큐(FIFO, 먼저 넣은 것 먼저).
- **왜 필요**: BFS=큐, DFS=스택(재귀), 힙=배열, 그래프 표현=배열/리스트. 자료구조 선택이 곧 복잡도.
- **출처/연결**: C4(힙=배열), C10(그래프 표현), C11(BFS 큐), C12(DFS 스택).

### PK9. [CS기초] 이진 트리
- **쉬운 정의**: 각 노드가 자식 최대 2개인 트리. "거의 완전(nearly complete)"하면 배열로 빈틈없이 담긴다(부모 i, 자식 2i·2i+1).
- **왜 필요**: 힙·Huffman·decision-tree·DFS forest가 전부 이진/일반 트리 구조.
- **출처/연결**: C4(힙), C6(decision-tree), C8(Huffman).

### PK10. [CS기초] 그래프 용어 직관 (정점·간선·경로)
- **쉬운 정의**: 점(정점 V)과 그것을 잇는 선(간선 E), 점을 따라가는 길(경로). 방향이 있으면 방향그래프.
- **왜 필요**: 그래프 줄기(C10~C16) 전체가 V·E·경로 위에서 정의된다. (단 알고리즘 자체는 C10에서 정식 전개 — 여기선 직관만)
- **출처/연결**: C10~C16(그래프 줄기 전체의 토대 어휘).

### PK11. [CS기초] 의사코드 읽기
- **쉬운 정의**: 특정 언어가 아닌, 사람이 읽는 절차 표기(`for`, `if`, 배열 A[i], swap). CLRS 스타일.
- **왜 필요**: 모든 알고리즘이 의사코드로 제시되므로, 이를 읽고 한 줄씩 추적할 수 있어야 한다.
- **출처/연결**: 거의 모든 노드(C3 PARTITION, C5 QUICKSORT, C15 RELAX 등).

---

## 1. 시험 가이드 (Notion Day 1~9 + summary PDF 출처)

> ⚠️ Notion Day 노트는 대부분 **헤더(목차) 수준 아웃라인**으로, 개념 본체보다 "어디까지 다뤘는지/무엇을 건너뛰었는지"의 범위 신호를 제공한다. 개념 본체는 PDF에서 추출.

- **[Day 2] Master Method 미출제**: 분할정복에서 마스터 정리는 다루지 않음. 점화식 풀이는 **substitution(치환)·recursion-tree(재귀 트리)** 방법만. (출처: Notion Day 2 + PDF 03)
- **[Day 4] DP running-time 슬라이드 일부 건너뜀**: DP 도입부 일부(p.54 이전 일부) 생략. (출처: Notion Day 4)
- **[Day 7] MST는 Prim's부터**: cut property/safe edge 이후 Prim's·Kruskal's 중심. (출처: Notion Day 7)
- **[Day 9] RSA 중심, 일부 페이지 건너뜀**: 정수론→공개키→RSA→PKI. (출처: Notion Day 9)
- **[summary 손글씨 강조 ⭐高]** — PDF 16은 교수의 시험 리뷰 슬라이드(필기 주석 포함). 강조된 항목:
  - O-notation 정의/증명 (3n+1=O(n²) 유도, c≥4, n≥1)
  - substitution method (T(n)=2T(n/2)+n ≤ cn lg n, boundary n₀=2, c≥2)
  - heaps: BUILD-MAX-HEAP, EXTRACT-MAX, 전체 O(n log n)
  - quicksort: PARTITION, balanced Θ(n lg n) vs unbalanced O(n²)
  - **decision-tree 하한 Theorem 8.1: 비교정렬 Ω(n lg n)** (n!≤2^h ⇒ lg(n!)≤h ⇒ Ω(n lg n)) — 증명 통째로 강조
  - LCS DP 점화식 + c[i,j] 표 추적
  - Huffman codes 트리 구성 (224,000 bits 예제)
  - amortized: binary counter aggregate 분석 ⇒ amortized O(1)
  - topological sort (DFS finishing time 역순)
  - Prim's MST 단계 추적
  - **⚠️ summary에서 빠진 것**: BFS/DFS 본체, Kruskal, 최단경로(Dijkstra/Bellman-Ford), Floyd-Warshall, RSA 수식 — summary는 전·중반 핵심에 집중. (그래프 후반·암호는 PDF 본강의로 출제 가능, 단 summary 미포함은 상대적 가중치 신호)
- **[기출/예상 명시 가이드]**: PDF 각 장 끝 "Self-study" 슬라이드에 연습문제 번호 지정(예: Ex 22.2-4 BFS 인접행렬 복잡도, 22.2-6 불가능한 BFS 트리, 22.3-5 edge classification, Problem 22-2 articulation points, 22.4-2 simple path 개수, 22.4-3 무방향 사이클 검출, 22.4-5 다른 topological sort). ⇒ 자가학습 지정 문제는 출제 후보.

---

## 2. 개념 카탈로그

### C1. 점근 표기 (Asymptotic Notation: O, Ω, Θ, o, ω)   ⭐高
- **출처**: PDF 02 (growth of functions) 전체; PDF 16 p.2~3 (강조)
- **선행 개념**: 없음 (기초)
- **한 줄 직관**: 입력 크기 n이 커질 때 함수의 성장률을 상수배·하위항 무시하고 분류하는 도구.
- **정의**:
  - $O(g(n)) = \{f(n): \exists c,n_0>0,\ 0\le f(n)\le cg(n),\ \forall n\ge n_0\}$ — 상계(worst).
  - $\Omega(g(n))$: $0\le cg(n)\le f(n)$ — 하계(best/lower).
  - $\Theta(g(n))$: $0\le c_1 g(n)\le f(n)\le c_2 g(n)$ — 정확한 한계(tight). $f=\Theta(g) \iff f=O(g)\ \text{and}\ f=\Omega(g)$ (Theorem 3.1).
  - $o,\omega$: non-tight (등호 불성립, e.g. $2n=o(n^2)$).
- **수식 의미**: c=상수배 허용, $n_0$=충분히 큰 n 시작점. "$n\ge n_0$만 성립하면 됨" — 작은 n 무시.
- **핵심 메커니즘/유도(시험 단골)**: $3n+1=O(n^2)$ 증명 → $3n+1\le cn^2$ 양변 $n^2$로 나눔 → $3/n+1/n^2\le c$ → $n\ge1, c\ge4$에서 성립.
- **성질**: 추이성(transitivity, O/Ω/Θ/o/ω 모두), 반사성(reflexivity, O/Ω/Θ), 대칭성(symmetry, Θ만), 전치대칭(transpose symmetry: $f=O(g)\iff g=\Omega(f)$). **삼분법(trichotomy) 불성립** — 비교 불가 예: $n^{1+\sin n}$ vs $n$.
- **흔한 함정**: O는 상한이지 "정확히 그 정도"가 아님(느슨해도 참, e.g. $3n+1=O(n^2)$도 참). Θ만이 정확한 동급.
- **시험 포인트**: 정의로 c·n₀ 찾기 증명, 표기 간 관계, 함수 성장률 정렬.

### C2. 재귀 점화식 풀이: 치환법·재귀트리   ⭐高
- **출처**: PDF 03 (Divide-and-Conquer); PDF 16 p.4~10 (substitution 강조)
- **선행 개념**: C1
- **한 줄 직관**: 분할정복 알고리즘의 수행시간 점화식 $T(n)$을 닫힌형 복잡도로 변환.
- **핵심 메커니즘**:
  - **Substitution(치환)**: ① 답 추측(guess) ② 수학적 귀납법으로 증명(귀납가정 → 귀납단계 → 경계조건).
    - 예: $T(n)=2T(\lfloor n/2\rfloor)+n$, guess $O(n\lg n)$, 증명 $T(n)\le cn\lg n$.
    - 귀납단계: $T(n)\le 2(c\lfloor n/2\rfloor\lg(\lfloor n/2\rfloor))+n \le cn\lg(n/2)+n = cn\lg n - cn\lg 2 + n = cn\lg n - cn + n \le cn\lg n$ (단 $c\ge1$).
    - **경계조건 트릭**: $n=1$은 불가($c\cdot1\lg1=0$이나 $T(1)=1$). ⇒ $n_0=2$로 설정, $T(2)=4\le c\cdot2\lg2$ ⇒ $c\ge2$. (모든 n 증명 불요, $n\ge n_0$만)
  - **Recursion-tree(재귀 트리)**: 각 레벨 비용 합산 → 트리 높이 × 레벨당 비용으로 추정.
- **표/그래프 인사이트**: $T(n)=2T(n/2)+n$ 재귀트리 → 각 레벨 비용 n, 높이 lg n ⇒ $\Theta(n\lg n)$.
- **⚠️ 누락(범위 신호)**: **마스터 정리(Master Method)는 다루지 않음** (Notion Day 2). 형태 $aT(n/b)+f(n)$만 언급, 3 case 미전개. → 점화식은 치환·재귀트리로만 풀 것.
- **시험 포인트**: 주어진 점화식을 치환법으로 증명(귀납단계 + 경계조건 처리가 핵심).

### C3. 정렬 — 삽입/병합 정렬   ⭐中
- **출처**: PDF 01 (getting started), PDF 03 (merge)
- **선행 개념**: C1, C2
- **삽입 정렬(Insertion sort)**: 최악 $\Theta(n^2)$, 최선 $\Theta(n)$(정렬됨), **stable, in-place**. 거의 정렬된 작은 입력에 유리. 불변식: A[1..j-1]은 항상 정렬 상태.
- **병합 정렬(Merge sort)**: 분할정복, $T(n)=2T(n/2)+\Theta(n)=\Theta(n\lg n)$ (최선=최악), **stable**, 공간 $\Theta(n)$(in-place 아님).
- **표 인사이트(비교)**: 삽입=in-place·안정·최악 느림 ↔ 병합=항상 n lg n·안정이나 추가공간 n.
- **시험 포인트**: 안정성/in-place/최악·최선 매칭형.

### C4. 힙·힙 정렬 (Heaps, Heapsort)   ⭐高
- **출처**: PDF 04 (Heap sort); PDF 16 p.11~14 (강조)
- **선행 개념**: C1, C3
- **한 줄 직관**: 거의 완전 이진 트리(배열 표현)로 최대/최소를 O(log n)에 뽑아 정렬.
- **정의**: (binary) heap = nearly complete binary tree. Max-heap property: 부모 ≥ 자식. 배열로 표현(부모 i → 자식 2i, 2i+1).
- **핵심 메커니즘/복잡도**:
  - MAX-HEAPIFY: $O(\lg n)$ (한 노드 아래로 내리기).
  - **BUILD-MAX-HEAP**: $i=\lfloor n/2\rfloor$부터 1까지 MAX-HEAPIFY → **O(n)** (느슨한 분석은 O(n log n)이나 tight는 O(n)).
  - HEAPSORT/EXTRACT-MAX: 루트(최대) 제거 → 마지막 원소를 루트로 → MAX-HEAPIFY 복원. n번 반복: $n\times O(\lg n)=O(n\lg n)$.
  - **전체 = BUILD O(n) + n·EXTRACT O(log n) = O(n log n)**.
- **속성**: **in-place, not stable**.
- **시험 포인트**: BUILD-MAX-HEAP가 왜 O(n)인지, EXTRACT-MAX 단계 추적, in-place·불안정.

### C5. 퀵 정렬 (Quicksort)   ⭐高
- **출처**: PDF 05 (Quicksort); PDF 16 p.15~18 (강조)
- **선행 개념**: C2, C3
- **한 줄 직관**: pivot 기준 분할 후 재귀(분할정복). 평균 빠르나 최악 느림.
- **의사코드**: `QUICKSORT(A,p,r): if p<r: q=PARTITION(A,p,r); QUICKSORT(A,p,q-1); QUICKSORT(A,q+1,r)`
- **PARTITION**: pivot x 기준 ≤x | x | >x로 재배치. 예: [2,8,7,1,3,5,6,4(pivot)] → [2,1,3,4,7,5,6,8].
- **복잡도**:
  - **Balanced partitioning**(매번 절반): 재귀트리 높이 lg n, 레벨당 n ⇒ $\Theta(n\lg n)$.
  - **Unbalanced**(매번 1:n-1): 높이 n ⇒ $O(n^2)$ (최악, 이미 정렬된 입력 + 끝 pivot).
  - 평균 $O(n\lg n)$.
- **속성**: **in-place, not stable**.
- **시험 포인트**: balanced vs unbalanced 재귀트리, 최악 케이스가 언제인지.

### C6. 선형시간 정렬 — Counting/Radix + 비교정렬 하한   ⭐高
- **출처**: PDF 06 (Sorting in linear time); PDF 16 p.19~23 (decision-tree 하한 강조)
- **선행 개념**: C3, C4, C5
- **Counting sort**: 키 범위 [0,k]. $\Theta(k+n)$, **stable**, in-place 아님. k=O(n)일 때 선형.
- **Radix sort**: 자릿수 d개를 LSD부터 안정 정렬. $\Theta(d(n+k))$, **stable**(내부 정렬이 안정해야), not in-place.
- **decision-tree 모델(⭐핵심)**: 비교정렬 = full binary tree. 잎=입력 순열, 내부노드 i:j=비교 $a_i\le a_j$. 알고리즘 실행 = 루트→잎 경로. 최악 비교 횟수 = 트리 높이 h.
- **Theorem 8.1 (비교정렬 하한, 증명 통째로 시험)**: 임의 비교정렬은 최악 $\Omega(n\lg n)$ 비교 필요.
  - 증명: 잎 개수 = $n!$ (모든 순열). 높이 h 이진트리 잎 수 $\le 2^h$ ⇒ $n!\le 2^h$ ⇒ $\lg(n!)\le h$. $\lg(n!)=\Theta(n\lg n)$ (식 3.19, Stirling) ⇒ $h=\Omega(n\lg n)$.
- **시험 포인트**: counting/radix 안정성·복잡도, **decision-tree 하한 증명 재현**(잎 n!, n!≤2^h, lg(n!)=Θ(n lg n)).

### C7. 동적 계획법 (Dynamic Programming) — 기초 + LCS   ⭐高
- **출처**: PDF 07 (Dynamic Programming); PDF 16 p.24~31 (LCS 강조)
- **선행 개념**: C2
- **한 줄 직관**: 겹치는 부분문제를 한 번만 풀어 표에 저장(memoization/bottom-up)해 지수→다항.
- **핵심 조건**: ① **최적 부분구조**(optimal substructure: 최적해가 부분문제 최적해로 구성) ② **겹치는 부분문제**(overlapping subproblems). 둘 다 있어야 DP 적용.
- **방식**: top-down memoization vs bottom-up table.
- **대표 예제**: assembly-line scheduling, rod cutting, **matrix-chain multiplication** ($O(n^3)$, 괄호 위치 결정), **LCS**.
- **LCS (Longest Common Subsequence, ⭐시험 단골)**:
  - substring(연속) ≠ subsequence(순서 유지, 비연속). 예: X=ABCBDAB, Y=BDCABA → LCS=BCBA (길이 4).
  - 점화식 ($c[i,j]$=$X_i,Y_j$ prefix의 LCS 길이):
    $$c[i,j]=\begin{cases}0 & i=0\ \text{or}\ j=0\\ c[i-1,j-1]+1 & x_i=y_j\\ \max(c[i,j-1],c[i-1,j]) & x_i\ne y_j\end{cases}$$
  - 시간 $\Theta(mn)$, 공간 $\Theta(mn)$ (길이만 필요하면 $\min(m,n)+1$로 축소). brute force는 $2^m$ 부분수열 ⇒ infeasible.
  - 표 추적 + 화살표(↖ 대각=일치, ↑/← =max 방향)로 LCS 복원.
- **시험 포인트**: 최적부분구조·겹침 정의, LCS 점화식 작성·표 채우기·복원, matrix-chain $O(n^3)$.

### C8. 그리디 알고리즘 (Greedy) — 활동선택/배낭/Huffman   ⭐高
- **출처**: PDF 08 (Greedy Algorithms); PDF 16 p.32~38 (Huffman 강조)
- **선행 개념**: C7 (DP와 대비)
- **한 줄 직관**: 매 단계 국소 최적(greedy choice)을 선택 → 전역 최적이 보장되는 문제에만 적용.
- **핵심 조건**: ① **greedy-choice property**(국소 최적이 전역 최적으로 이어짐) ② 최적 부분구조.
- **Activity selection**: 가장 빨리 끝나는(earliest finish) 활동 선택 → 최대 개수. 그리디 성립.
- **0-1 knapsack vs fractional**: **0-1은 그리디 실패**(DP 필요), **fractional(분할 가능)은 그리디 성공**(가치/무게 비율 큰 것부터).
- **Huffman codes (⭐시험 단골)**:
  - 데이터 압축. 빈도 높은 문자에 짧은 코드(variable-length). prefix code(어떤 코드도 다른 코드의 접두사 아님) → full binary tree.
  - 구성: 빈도 최소 2개 노드를 합쳐 새 내부노드(빈도=합) 생성, 반복(min-priority queue). $O(n\lg n)$.
  - 예제(시험 단골): {a:45,b:13,c:12,d:16,e:9,f:5}(천 단위). 고정 3-bit = 300,000 bits. Huffman variable = (45·1+13·3+12·3+16·3+9·4+5·4)·1000 = **224,000 bits**. 트리: f,e 합→14, c,b→25, 14,d→30, 25,30→55, a,55→100.
- **흔한 함정**: 모든 최적화가 그리디로 풀리지 않음(0-1 knapsack 반례). greedy-choice property 증명 필요.
- **시험 포인트**: greedy vs DP 선택 기준, Huffman 트리 구성·비트 수 계산, fractional vs 0-1 knapsack.

### C9. 분할상환 분석 (Amortized Analysis)   ⭐高
- **출처**: PDF 09 (amortized analysis); PDF 16 p.39~48 (binary counter 강조)
- **선행 개념**: C1
- **한 줄 직관**: 연산 시퀀스 전체의 평균 비용으로 개별 연산을 평가(확률 무관, 최악보장).
- **정의**: n개 연산 시퀀스의 최악 총 시간 $T(n)$ → 연산당 평균(=amortized cost) $T(n)/n$. **확률 미개입**(평균케이스 분석과 다름), 각 연산의 worst-case 평균 성능 보장.
- **세 방법**:
  - **Aggregate(집계)**: 시퀀스 총비용 $T(n)$ 구해 $T(n)/n$.
  - **Accounting(회계/credit)**: 연산에 amortized 비용 부과, 차액을 credit으로 적립/사용.
  - **Potential(잠재)**: 퍼텐셜 함수 $\Phi$로 상태 에너지 정의.
- **대표 예제 — Binary counter INCREMENT (⭐시험 단골, aggregate)**:
  - k-bit 카운터, INCREMENT 비용 = 뒤집힌 비트 수. 단일 최악 $\Theta(k)$(전부 1일 때).
  - naive: n번 → $O(nk)$. **Aggregate 개선**: A[0]은 매번 flip(n회), A[1]은 n/2회, A[2]는 n/4회... → 총 flip $\sum_{i=0}^{k-1}\lfloor n/2^i\rfloor < \sum_{i=0}^\infty n/2^i = 2n$ ⇒ 총 $O(n)$.
  - **amortized cost = O(n)/n = O(1)**.
- **다른 예제**: stack MULTIPOP, dynamic table doubling(amortized O(1)).
- **시험 포인트**: binary counter aggregate 분석(2n 합 유도), amortized O(1) 도출, 세 방법 구분.

### C10. 그래프 기초 + 표현 (Graph basics & representation)   ⭐中
- **출처**: PDF 10 (Elementary Graph Algorithms) p.1~41
- **선행 개념**: 없음
- **정의·용어**: 방향/무방향 그래프, degree(in/out), path/simple path, cycle/simple cycle, acyclic, connected(무방향)/strongly connected(방향), connected components/SCC, bipartite(이분), forest, **tree(|E|=|V|-1, 연결·비순환·유일경로 등 동치)**, **DAG**.
- **핵심 수식**: **Handshaking lemma**: $\sum_{v}\deg(v)=2|E|$ (무방향). edge 상한: 방향 $|E|\le|V|^2$, 무방향 $|E|\le|V|(|V|-1)/2$.
- **표현(표 인사이트)**:
  - **인접 리스트(adjacency-list)**: 공간 $\Theta(V+E)$. sparse(희소)에 유리. edge 존재 검사 $O(V)$(degree만큼), 모든 edge 나열 $\Theta(V+E)$.
  - **인접 행렬(adjacency-matrix)**: 공간 $\Theta(V^2)$. dense(밀집)에 유리. edge 검사 $\Theta(1)$, 모든 edge 나열 $\Theta(V^2)$. 가중치 저장 용이.
  - 트레이드오프: 희소→리스트, 밀집/빠른 edge검사→행렬.
- **시험 포인트**: |E|=|V|-1 트리 특성, handshaking, 리스트 vs 행렬 공간·연산 복잡도 매칭(Ex 22.2-4 BFS 인접행렬은 $O(V^2)$).

### C11. 너비 우선 탐색 (BFS)   ⭐中
- **출처**: PDF 10 p.42~63
- **선행 개념**: C10
- **한 줄 직관**: source에서 거리 증가 순(1,2,3...)으로 정점 발견 → 무가중 최단경로(거리=간선 수).
- **메커니즘**: 큐 사용. 색: **WHITE**(미발견), **GRAY**(발견·큐 안), **BLACK**(완료·큐 밖). 각 정점 $v.d$(거리), $v.\pi$(predecessor).
- **의사코드 핵심**: source s를 GRAY·d=0로 큐에 넣고, DEQUEUE한 u의 인접 WHITE 정점 v를 GRAY·$v.d=u.d+1$·$v.\pi=u$로 ENQUEUE, u 처리 후 BLACK.
- **predecessor subgraph $G_\pi$ = breadth-first tree** ($|E_\pi|=|V_\pi|-1$, 연결). tree edges.
- **복잡도**: 초기화 $\Theta(V)$ + 탐색 $O(V+E)$(정점 1회·간선 ≤2회) = **$O(V+E)$**.
- **시험 포인트**: 큐 상태·색·d 단계별 추적, BFS=무가중 최단거리, $O(V+E)$.

### C12. 깊이 우선 탐색 (DFS) + 간선 분류   ⭐中
- **출처**: PDF 10 p.64~82
- **선행 개념**: C10, C11
- **한 줄 직관**: 가능한 깊이 먼저 탐색, 막히면 backtrack. 두 timestamp로 구조 파악.
- **메커니즘**: 색(WHITE/GRAY/BLACK) + **timestamp** $v.d$(발견=gray), $v.f$(완료=black). 재귀 DFS-VISIT. predecessor subgraph = **depth-first forest**.
- **Parenthesis theorem(괄호 정리)**: 임의 두 정점의 gray interval [d,f]는 **포함(inclusion, 조상-자손)** 또는 **분리(disjoint)** — 부분 겹침 불가.
- **간선 분류(Edge classification, Ex 22.3-5 시험)**:
  - **Tree edge**: DFS forest 간선(처음 도달 시 v가 WHITE).
  - **Back edge**: 자손→조상(v가 GRAY). self-loop 포함.
  - **Forward edge**: 조상→자손(non-tree, v가 BLACK·gray interval 포함).
  - **Cross edge**: 그 외(v가 BLACK·gray interval 분리).
  - 색 판별: 처음 도달한 v가 WHITE→tree, GRAY→back, BLACK→forward/cross.
- **중요 정리**: **무방향 그래프 DFS는 tree edge와 back edge만** 존재(forward/cross 없음). **방향 그래프 G가 acyclic ⟺ DFS에 back edge 없음**.
- **복잡도**: $\Theta(V+E)$.
- **시험 포인트**: 간선 분류·색 판별, parenthesis theorem, acyclic⟺no back edge, d/f timestamp 추적.

### C13. 위상 정렬 (Topological Sort)   ⭐高
- **출처**: PDF 10 p.83~88; PDF 16 p.49~52 (강조)
- **선행 개념**: C12 (DFS)
- **한 줄 직관**: DAG의 정점들을 모든 간선이 왼→오른쪽이 되도록 선형 순서로 나열(작업 순서).
- **메커니즘(세 가지 관점)**: ① 왼쪽부터 in-degree 0인 노드 배치 ② 오른쪽부터 out-degree 0 배치 ③ **DFS 실행 후 finishing time 감소 순으로 나열**(= 오른쪽에 increasing finishing time).
- **정당성**: 간선 (u,v) 있으면 $v.f < u.f$ (DFS에서 v는 white/black, gray 불가 → back edge 없음 ⇒ acyclic). Lemma: DAG ⟺ DFS no back edge.
- **복잡도**: $\Theta(V+E)$.
- **예제**: 옷 입기 DAG(under shorts→pants→belt..., socks→shoes) → finishing time로 선형 순서.
- **시험 포인트**: DFS finishing time 역순으로 위상순서 도출, DAG 조건, $\Theta(V+E)$.

### C14. 최소 신장 트리 (MST) — cut property, Prim, Kruskal   ⭐高
- **출처**: PDF 11 (minimum spanning trees); PDF 16 p.53~58 (Prim 강조)
- **선행 개념**: C10, C8(greedy), C4(heap), disjoint-set
- **한 줄 직관**: 가중 무방향 연결그래프에서 모든 정점을 잇는 최소 비용 트리(|E|=|V|-1, acyclic).
- **용어**: **cut** (S,V-S) = V의 분할. edge가 cut을 **cross**(양 끝이 S/V-S 분리). cut이 set A를 **respect**(A의 어떤 간선도 cut을 cross 안함). **light edge** = cut을 cross하는 최소 가중 간선. **safe edge** = A에 더해도 어떤 MST의 부분집합 유지.
- **GENERIC-MST**: A=∅에서 매번 safe edge를 찾아 추가, 트리 될 때까지.
- **Theorem 23.1 (cut property, 증명 시험 가능)**: A가 어떤 MST에 포함 + cut이 A를 respect + (u,v)가 그 cut의 light edge ⇒ (u,v)는 A에 safe.
  - 증명 골자: MST T가 (u,v) 미포함이라 가정 → (u,v)가 T의 경로 p와 사이클 형성 → p 위에 cut을 cross하는 간선 (x,y) 존재 → $T'=T-\{(x,y)\}\cup\{(u,v)\}$. $w(u,v)\le w(x,y)$(light)이므로 $w(T')\le w(T)$ ⇒ T'도 MST, (u,v) safe.
- **Corollary 23.2**: forest $G_A$의 연결요소 C에서 다른 요소로 가는 light edge는 safe (Prim/Kruskal 근거).
- **Prim's algorithm**:
  - A가 항상 단일 트리. root r에서 시작, 매 단계 트리와 외부를 잇는 light edge 추가(min-priority queue, key[v]=트리로의 최소 간선 가중).
  - 의사코드: 모든 v의 key=∞·π=NIL, key[r]=0, Q=V. while Q≠∅: u=EXTRACT-MIN(Q); 인접 v∈Q에 대해 w(u,v)<key[v]면 π[v]=u, key[v]=w(u,v)(DECREASE-KEY).
  - 복잡도: BUILD-HEAP O(V) + V번 EXTRACT-MIN O(lg V) + E번 DECREASE-KEY ⇒ **O(E lg V)** (binary heap).
- **Kruskal's algorithm**:
  - 간선을 가중 오름차순 정렬, 사이클 안 생기면 추가(disjoint-set: MAKE-SET/FIND-SET/UNION).
  - 의사코드: A=∅; 각 v MAKE-SET(v); E를 가중 오름차순 정렬; 각 (u,v)에 대해 FIND-SET(u)≠FIND-SET(v)면 A에 추가·UNION(u,v).
  - 복잡도: 정렬 $O(E\lg E)$ + E번 FIND/UNION $O(E\cdot\alpha(V))$ ⇒ **$O(E\lg V)$** ($\lg E=O(\lg V)$).
- **표 인사이트**: Prim=정점 성장(heap)·dense에 유리 ↔ Kruskal=간선 정렬(disjoint-set)·sparse에 유리. 둘 다 $O(E\lg V)$.
- **시험 포인트**: cut property 정의·증명, Prim/Kruskal 단계 추적·복잡도, safe/light/respect 정의.

### C15. 단일 출발점 최단경로 (SSSP) — Dijkstra, Bellman-Ford, DAG, PERT   ⭐高
- **출처**: PDF 13 (Single Source Shortest Paths)
- **선행 개념**: C11(BFS), C13(topo), C14
- **정의**: path weight=간선 가중 합. shortest-path weight $\delta(u,v)$=최소. d[v]=추정치, π[v]=predecessor.
- **음수 가중 처리**: **음수 간선 자체는 OK**. 문제는 **source에서 도달 가능한 음수 사이클** → $-\infty$로 최단경로 정의 불가. 최단경로는 사이클 미포함(길이 ≤ |V|-1).
- **Relaxation(완화)**: `if d[u]+w(u,v)<d[v]: d[v]=d[u]+w(u,v); π[v]=u`. 모든 SSSP의 공통 핵심 연산. d[v]는 $\delta(s,v)$로 수렴.
- **Dijkstra (⭐, 비음수 가중만)**:
  - greedy: S=∅에서 min d[u]인 정점을 EXTRACT-MIN으로 S에 추가(확정), 인접 RELAX.
  - 의사코드: INITIALIZE-SINGLE-SOURCE; S=∅; Q=V; while Q≠∅: u=EXTRACT-MIN(Q); S=S∪{u}; 각 v∈Adj[u] RELAX(u,v,w).
  - **복잡도**: array $O(V^2)$, binary heap $O((V+E)\lg V)=O(V\lg V+E\lg V)$, Fibonacci heap $O(V\lg V+E)$.
  - **정당성(증명 강조)**: u가 S에 추가될 때 $d[u]=\delta(s,u)$. 모든 간선 양수 가정, 최초로 $d[u]\ne\delta(s,u)$인 u 모순 유도(경로가 S 밖으로 나가는 첫 간선 (x,y), $\delta(s,y)<\delta(s,u)$ ⇒ y가 먼저 추가됐어야 함).
- **Bellman-Ford (음수 가중 허용)**:
  - 의사코드: INITIALIZE; **|V|-1번** 모든 간선 RELAX; 마지막으로 모든 간선 검사해 $d[u]+w(u,v)<d[v]$면 음수사이클 → return FALSE, 아니면 TRUE.
  - **복잡도 $O(VE)$**. 음수사이클 검출(반환값 = source 도달 음수사이클 없음).
  - 음수사이클 정당성: TRUE인데 음수사이클 $\langle v_0..v_k\rangle$ 가정 → $\sum d[v_i]\le\sum(d[v_{i-1}]+w)$ ⇒ $0\le\sum w(v_{i-1},v_i)$ 모순.
- **DAG shortest paths**: 위상정렬 후 순서대로 RELAX. **$\Theta(V+E)$** (음수 가중 OK, 사이클 없으니 안전).
- **PERT chart**: 간선=작업, 가중=소요시간. **critical path(임계경로)=DAG의 최장 경로**. 구하기: 가중 음수화 후 DAG-SHORTEST-PATHS, 또는 ∞→-∞·>→< 바꿔 최장경로.
- **시험 포인트**: Dijkstra 단계 추적(Q/S/d 표)·비음수 제약·정당성, Bellman-Ford |V|-1 패스·음수사이클 검출·$O(VE)$, DAG $\Theta(V+E)$, PERT 임계경로=최장경로.

### C16. 모든 쌍 최단경로 (All-Pairs) — Floyd-Warshall   ⭐高
- **출처**: PDF 14 (All-pairs shortest paths)
- **선행 개념**: C7(DP), C15
- **SSSP 반복 접근**: 각 정점을 source로 SSSP |V|회. 비음수→Dijkstra: array $O(V^3)$, heap $O(V^2\lg V+VE\lg V)$. 음수→Bellman-Ford: $O(V^2E)=O(V^4)$ dense.
- **Floyd-Warshall (⭐, DP)**:
  - 인접행렬 W($w_{ij}=w(i,j)$, 자기 0, 무간선 ∞). 결과 거리행렬 D($d_{ij}=\delta(i,j)$), predecessor 행렬 Π.
  - **intermediate vertex(중간 정점)** 개념: 경로 내부 정점. $d_{ij}^{(k)}$ = 중간 정점이 {1..k}만인 i→j 최단 가중.
  - **점화식 (DP 핵심)**:
    $$d_{ij}^{(k)}=\begin{cases}w_{ij} & k=0\\ \min(d_{ij}^{(k-1)},\ d_{ik}^{(k-1)}+d_{kj}^{(k-1)}) & k\ge1\end{cases}$$
    (k를 중간점으로 안 씀 vs 씀: $i\rightsquigarrow k\rightsquigarrow j$로 분해). 최종 $D^{(n)}=\delta$.
  - 의사코드: `for k=1..n: for i=1..n: for j=1..n: d_ij^(k)=min(d_ij^(k-1), d_ik^(k-1)+d_kj^(k-1))`. **$\Theta(V^3)$**.
  - Π 복원: $\pi_{ij}^{(k)}$ = i→j 경로에서 j의 predecessor.
- **시험 포인트**: Floyd-Warshall 점화식·intermediate vertex 의미·$\Theta(V^3)$·행렬 $D^{(k)}$ 단계 계산, SSSP 반복과 복잡도 비교.

### C17. 정수론 기초 (Number Theory) — 소수, Fermat, Euler φ, 소수판정   ⭐高
- **출처**: PDF 15 (Number-Theoretic Algorithms) p.1~19
- **선행 개념**: 없음 (모듈러 산술)
- **소수·인수분해**: 소수=1·자신만 약수. 소인수분해 $a=\prod_{p\in P}p^{a_p}$. **인수분해는 곱셈보다 어렵다**(RSA 안전성 근거).
- **relatively prime(서로소)**: 공약수 1뿐. GCD = 소인수분해 최소 지수 곱 (예: gcd(18,300)=2¹·3¹·5⁰=6).
- **Fermat's little theorem**: $a^{p-1}\bmod p=1$ (p 소수, gcd(a,p)=1). 공개키·소수판정 활용.
- **Euler totient φ(n)**: n과 서로소인 [1,n-1] 개수(reduced set of residues). **φ(p)=p-1** (p 소수), **φ(pq)=(p-1)(q-1)** (p,q 소수). 예: φ(37)=36, φ(21)=2·6=12.
- **Euler's theorem(Fermat 일반화)**: $a^{\phi(n)}\bmod n=1$ (gcd(a,n)=1). 예: $3^4=81\equiv1\bmod10$.
- **소수 판정(Primality testing)**: trial division(√n까지, 작은 수만). 통계적 판정(소수는 항상 성질 만족, 일부 합성수=pseudo-prime도 만족).
- **Miller-Rabin**: Fermat 기반. n-1=2^k·q(q 홀수), 랜덤 a로 $a^q$ 및 $a^{2^j q}\bmod n$ 검사. "composite" 확정 / "maybe prime". pseudo-prime 검출 확률 <1/4, t회 반복 시 소수 확률 $1-4^{-t}$ (t=10이면 >0.99999).
- **소수 분포(Prime number theorem)**: 소수는 대략 ln n 간격. 짝수·5배수 제외하면 실제 0.4·ln(n)개 테스트면 소수 발견.
- **primitive root(원시근)**: $a^m\bmod n=1$의 최소 m=φ(n)이면 a는 원시근. p 소수면 a의 거듭제곱이 mod p 군을 생성.
- **discrete logarithm(이산로그)**: $a^x\equiv b\bmod p$의 x. 지수화는 쉬우나 이산로그는 어려움(Diffie-Hellman 안전성).
- **시험 포인트**: φ(pq)=(p-1)(q-1) 계산, Fermat/Euler 정리 적용, Miller-Rabin 절차·확률, 이산로그 난해성.

### C18. Diffie-Hellman 키 교환   ⭐中
- **출처**: PDF 15 p.16~19
- **선행 개념**: C17
- **한 줄 직관**: 안전하지 않은 채널에서 공유 비밀키를 합의(이산로그 난해성 기반).
- **메커니즘**: 공개 q(소수), α(q의 원시근). A: 비밀 $X_A$, 공개 $Y_A=\alpha^{X_A}\bmod q$. B: 비밀 $X_B$, 공개 $Y_B=\alpha^{X_B}\bmod q$. 공유키 $K=(Y_B)^{X_A}=(Y_A)^{X_B}=\alpha^{X_A X_B}\bmod q$.
- **예제**: q=353, α=3, $X_A=97→Y_A=40$, $X_B=233→Y_B=248$, K=160. 공격자는 $3^x\bmod353=40$(이산로그) 풀어야 함 → 어려움.
- **시험 포인트**: 키 합의 수식·공유키 일치 증명, 안전성=이산로그.

### C19. 공개키 암호 + RSA   ⭐高
- **출처**: PDF 15 p.20~47
- **선행 개념**: C17 (Euler 정리·φ가 핵심)
- **공개키 개념**: public/private 키 쌍. 공개키로 private 추론 infeasible. 보통 공개키 암호화·private 복호화. **RSA는 양방향 가능**.
- **요건**: **trap-door one-way function** 필요(한 방향 easy, 역방향 infeasible — 단 trap-door 정보 알면 easy). 6 요건(키 생성·암호·복호 easy, private 추론·평문 복구 infeasible, (선택)순서 무관). 응용: encryption/decryption(secrecy), digital signature(authentication), key exchange.
- **RSA (⭐핵심, Rivest-Shamir-Adleman 1977)**:
  - block cipher, 평문/암호문 = [0,n-1] 정수. 보통 n=1024 bits.
  - **키 생성**: 소수 p,q 선택 → **n=pq** → **φ(n)=(p-1)(q-1)** → e 선택(**gcd(e,φ(n))=1**, 1<e<φ(n)) → **d≡e⁻¹ mod φ(n)** (확장 유클리드). **공개키 {e,n}, 개인키 {d,n}**.
  - **암호화** $C=M^e\bmod n$, **복호화** $M=C^d\bmod n=M^{ed}\bmod n$.
  - **정당성(Euler 따름정리)**: $ed=k\phi(n)+1$ ⇒ $M^{ed}=M^{k\phi(n)+1}\equiv M\bmod n$ (Euler 정리 $m^{k\phi(n)+1}\equiv m\bmod n$, n=pq). $ed\equiv1\bmod\phi(n)$.
  - **예제(시험 단골)**: p=17, q=11 → n=187, φ(n)=16·10=160, e=7(gcd(7,160)=1), d=23(7·23=161≡1 mod 160). 평문 88 → $88^7\bmod187=11$ → $11^{23}\bmod187=88$.
  - **보안(3 공격)**: ① brute force(키 공간 크게) ② **mathematical(n 인수분해→φ→d, 또는 직접 φ/d — 모두 인수분해 난도)** ③ timing attack(지수화 시간 누출 → 상수시간·랜덤지연·blinding). p,q 제약: 길이 비슷·(p-1)(q-1)에 큰 소인수·gcd(p-1,q-1) 작게·d>n^(1/4).
- **시험 포인트**: RSA 키 생성 전 과정(n,φ,e,d), 암복호 수식, **Euler 정리로 정당성 증명**, 예제 계산, 인수분해=보안 근거, trap-door one-way function.

### C20. 디지털 서명 + PKI (X.509, CA)   ⭐低
- **출처**: PDF 15 p.48~56
- **선행 개념**: C19
- **인증(authentication)**: private key로 암호화 → 누구나 public key로 복호 → 송신자 검증.
- **디지털 서명**: 해시코드를 private key로 암호화. 출처·무결성 인증. **기밀성 미제공**(변조는 막으나 도청은 못 막음).
- **public key certificate**: 공개키 + 소유자 ID를 신뢰 3자(CA)가 서명. CA 해시 암호화로 서명, CA 공개키로 검증.
- **Certificate Authority(CA)**: 공개키+ID를 서명하는 신뢰기관(정부·금융). 사용자는 인증서 발급·공개.
- **X.509**: 공개키 인증서 표준(IPsec/SSL/SET/S-MIME). CCITT X.500의 일부. RSA 권장. 필드: version, serial #, issuer, validity, subject, public key, signature, CRL(revocation).
- **PKIX**: PKI 아키텍처(end entity, CA, RA, CRL issuer, repository).
- **시험 포인트**: 서명=무결성/인증(기밀성 아님), 인증서 구조, CA 역할. (출제빈도 상대적 낮음)

---

## 3. 개념 의존성 맵

```
C1(점근표기) → C2(점화식) → C3(삽입/병합) → C4(힙) → C5(퀵) → C6(선형정렬+하한)
C1 → C9(분할상환)
C2 → C7(DP) ─┬→ C8(그리디)  [대비]
             └→ C16(Floyd-Warshall, DP)

C10(그래프기초) → C11(BFS) → C12(DFS) → C13(위상정렬)
C10 → C14(MST: cut→Prim/Kruskal)   [C4 힙·C8 그리디 활용]
C11,C13,C14 → C15(SSSP: Dijkstra/Bellman-Ford/DAG/PERT) → C16(All-pairs)

C17(정수론: Fermat/Euler φ) ─┬→ C18(Diffie-Hellman)
                            └→ C19(RSA) → C20(서명/PKI)
```
핵심 선수관계: **점근표기(C1)→점화식(C2)→정렬/DP/그리디 전부**. **Euler φ·정리(C17)→RSA 정당성(C19)**. **DFS(C12)→위상정렬(C13)·acyclic 판정**. **cut property(C14)→Prim/Kruskal**. **relaxation→모든 최단경로(C15,C16)**.

---

## 4. 누락·확인 필요 목록

- ⚠️ **Master Method(마스터 정리)**: PDF 03·Notion Day 2 모두 "다루지 않음" 확인. 형태만 언급, 3 case 미전개 → **출제 범위 외**(추측으로 보강 금지).
- ⚠️ **PDF 번호 12 없음**: 강의자료 폴더에 11→13 점프(파일 11개=00~11, 13~16). 누락 아닌 의도적 번호 건너뜀(16개 파일 모두 정독 완료).
- ⚠️ **Notion Day 노트 깊이**: Day 1~9 노트는 헤더/범위 아웃라인 위주로, 구체적 기출 문제·점수 배분은 명시되지 않음. "명시적 기출 가이드"는 각 PDF의 Self-study 연습문제 번호로 대체 기록(§1).
- ⚠️ **summary(PDF 16) 미포함 후반 주제**: 그래프 후반(BFS/DFS 본체·Kruskal)·최단경로·Floyd-Warshall·RSA는 summary 리뷰 슬라이드에 없음. 이는 출제 가중치가 상대적으로 낮을 신호일 수 있으나, **본강의(PDF 10·11·13·14·15)에서 정식 다룸 → 출제 가능**. summary는 전·중반(복잡도/정렬/DP/그리디/분할상환/위상정렬/Prim) 집중 확인.
- ✅ MAX-HEAPIFY·PARTITION 등 일부 의사코드 세부 라인은 PDF 본문에 있으나 본 추출에서는 시험 핵심(복잡도·불변식·동작)으로 압축. 필요 시 PDF 04 p.12, PDF 05 PARTITION 슬라이드 재확인.
