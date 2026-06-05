# 인공지능개론 (최용석 교수) — 강의 자료 추출본
> 출처: 강의자료 PDF 10개 + Notion Day 8개 | 추출일: 2026-06-02
> (PDF는 1.Uninformed / 2-1.Heuristic / 2-2.Local / 3.Adversarial / 4.CSP / 5.kNN / 5-Supp.DecisionTree / 6.Clustering / 7.Q-learning / 8.ANN — 과제 명세의 "9개"는 5.kNN과 5-Supp.DecisionTree를 한 묶음으로 센 듯하나 **물리 파일은 10개**)

---

## 0. 과목 개요

- **이 과목이 푸는 문제**: 딥러닝 이전의 **Classic AI / Classic Machine Learning**. 큰 줄기는 교수 명시대로 **"Problem Solving by Search Algorithm"**(탐색으로 문제 풀기)에서 시작해, 분류·군집·강화학습·신경망(고전 backprop)으로 확장. (출처: Notion Day1)
- **두 큰 흐름**:
  1. **탐색(Search)** — 상태공간을 노드/엣지 그래프로 보고 목표 상태(또는 최적 경로)를 찾는다. Uninformed(BFS/DFS/IDS/UCS) → Informed(Greedy/A*/Local) → Adversarial(Minimax/α-β) → CSP(backtracking + heuristics + local search).
  2. **학습(Learning)** — 지도(kNN·Decision Tree·Perceptron/BP-ANN), 비지도(k-means·계층군집), 강화(Q-learning).
- **역사적 맥락(Day1)**: 1980년대 NN은 연산량 한계로 학습 불가 → GPU 병렬연산 보편화로 AI 부흥.
- **평가 비중 (출처: Notion Day1)**: 출석 10% / 과제 15% / **기말고사 75%**. (중간고사 항목은 노트에 없음 → 사실상 기말 중심.)

---

## 0-1. 선행 지식 키워드 (배경지식 — 본문에서 설명 없이 전제되는 것)

> 본 과목(C1~C23)이 *설명 없이 전제하는* CS·수학 기초만 선별(과목 내부 노드와 중복 금지 — 토대만). 배경 없는 학습자를 본문 출발선에 세우는 글로서리.

### PK1. [수학] 확률 · 조건부확률 · 기댓값
- **쉬운 정의**: 사건이 일어날 가능성을 0~1로 나타낸 값. 조건부확률 P(A|B)는 "B가 일어난 상황에서 A가 일어날 확률". 기댓값은 "값 × 그 값이 나올 확률"을 모두 더한 평균.
- **왜 필요**: Decision Tree의 엔트로피 $E(p)=\sum_j -p_j\log_2 p_j$·Gini는 클래스 **확률 $p_j$**의 함수이고, kNN 다수결·k-means 군집도 확률/평균 개념을 깐다.
- **출처/연결**: C18(Entropy·Gini), C17(majority voting).

### PK2. [수학] 로그 ($\log_2$)와 지수
- **쉬운 정의**: $\log_2 x$ = "2를 몇 번 곱해야 x가 되나". 지수 $b^d$의 역연산. $\log_2 8=3$.
- **왜 필요**: 엔트로피의 $\log_2$(비트 단위 정보량), 그리고 탐색 복잡도 $O(b^d)$의 "지수 폭발"을 읽으려면 로그·지수 감각이 전제.
- **출처/연결**: C18(Entropy), C1~C5b(복잡도 $b^d$).

### PK3. [수학] 벡터 · 거리(유클리드/맨해튼)
- **쉬운 정의**: 데이터를 숫자 좌표 묶음 ⟨a₁,…,aₙ⟩으로 본 것이 벡터. 두 점의 가까운 정도가 거리 — 직선거리(유클리드 $\sqrt{\sum(x_i-y_i)^2}$), 격자거리(맨해튼 $\sum|x_i-y_i|$).
- **왜 필요**: kNN은 "가장 가까운 이웃"을 거리로 찾고, k-means centroid는 벡터 평균, A\* 휴리스틱 h₂도 맨해튼 거리.
- **출처/연결**: C17(kNN 거리식), C19(centroid), C6/C8(맨해튼 휴리스틱).

### PK4. [수학] 정규화(표준화) · 평균/표준편차
- **쉬운 정의**: 평균 $\bar x$를 빼고 표준편차 $\sigma$로 나눠($x'=(x-\bar x)/\sigma$) 축들의 스케일을 같게 맞추는 것. 표준편차는 값들이 평균에서 흩어진 정도.
- **왜 필요**: kNN에서 정규화를 안 하면 스케일 큰 축이 거리를 지배해 오분류 — "정규화가 왜 필요한가"가 시험 단골.
- **출처/연결**: C17(scaling).

### PK5. [수학] 미분 · 경사하강(gradient descent)
- **쉬운 정의**: 미분 = 함수가 어느 방향으로 얼마나 빨리 변하는지(기울기). 경사하강 = 기울기 반대 방향으로 조금씩 내려가 함수의 최솟값을 찾는 방법("산에서 가장 가파른 내리막으로 한 걸음씩").
- **왜 필요**: Backpropagation은 오차함수를 **gradient descent**로 줄이고, sigmoid 도함수 $y(1-y)$가 핵심. delta rule도 같은 원리.
- **출처/연결**: C23(역전파), C22(delta rule).

### PK6. [CS기초] 그래프 · 트리 자료구조
- **쉬운 정의**: 그래프 = 점(노드)과 그것을 잇는 선(엣지)의 모임. 트리 = 사이클 없이 위에서 아래로 갈라지는 그래프(루트·자식·잎).
- **왜 필요**: 탐색 전체가 "상태=노드, 연산자=엣지"인 그래프를 트리로 펼치는 것. 게임트리·CSP constraint graph·덴드로그램도 그래프/트리.
- **출처/연결**: C1(탐색 정식화), C11(게임트리), C14(constraint graph), C20(덴드로그램).

### PK7. [CS기초] 큐(FIFO) · 스택(LIFO) · 우선순위 큐(Min-Heap)
- **쉬운 정의**: 큐=먼저 들어온 게 먼저 나옴(FIFO, 줄서기). 스택=나중에 들어온 게 먼저 나옴(LIFO, 접시 쌓기). 우선순위 큐(Min-Heap)=가장 작은 값이 먼저 나옴.
- **왜 필요**: BFS=FIFO 큐, DFS=LIFO 스택, UCS/A\*=Min-Heap — 탐색 전략의 차이가 곧 **어떤 자료구조로 fringe를 관리하느냐**.
- **출처/연결**: C2(BFS), C3(DFS), C5(UCS), C8(A\*).

### PK8. [CS기초] 재귀(recursion) · 백트래킹
- **쉬운 정의**: 재귀=함수가 자기 자신을 더 작은 문제로 다시 부르는 것. 백트래킹=시도하다 막히면 **한 단계 되돌아가** 다른 선택을 해보는 것.
- **왜 필요**: DFS·backtracking search·minimax backup·HAC 병합이 모두 재귀 구조이고, CSP는 백트래킹 그 자체.
- **출처/연결**: C3(DFS), C15(backtracking), C11(minimax 재귀).

### PK9. [CS기초] 빅오 표기 · 점근 복잡도
- **쉬운 정의**: 입력이 커질 때 연산량이 **얼마나 빨리 늘어나는지**를 대략 나타내는 표기. $O(n)$=선형, $O(b^d)$=지수(폭발적).
- **왜 필요**: 탐색 4지표의 time/space($O(b^d)$ vs $O(bd)$), k-means $O(IKnm)$ 선형, α-β $O(b^{d/2})$ — 모든 알고리즘 비교의 공통 언어.
- **출처/연결**: C1~C5b, C12, C16, C19.

---

## 1. 시험 가이드 (Notion 출처)

> ⚠️ Notion 어디에도 "중간고사 기출"은 없음. Day7에 **기말 시험 형식**이 명시돼 있어 가장 강한 신호.

- **[Day 7] 기말 시험 형식 (교수 명시, 가장 중요)** (출처: Notion Day7):
  - **객관식 없음**
  - **증명/유도 문제 없음** ← 수식 암기보다 **개념·절차·"왜"**를 묻는다
  - **토픽마다 한 문제씩**, 각 토픽의 **대표 문제**로 출제 (예: "Decision Tree에서 가장 중요한 게 뭐냐!")
  - **총 8~10문제**
  - → ⭐ **함의**: 각 토픽(탐색 5종, kNN, DT, 군집, Q-learning, ANN)의 "핵심 한 방"을 서술형으로 답할 수 있어야 함. 복잡한 수식 손계산보다 **개념 정의·장단점·언제 쓰나·핵심 메커니즘**.

- **[Day 5] CSP Local Search → "시험 문제, 졸업 시험 단골 문제!"** (교수 강조, 출처: Notion Day5):
  - Hill-Climbing을 CSP에 적용 / **min-conflicts** / Hill-Climbing vs. Simulated Annealing 비교가 단골.
  - → ⭐⭐ 최우선 학습 대상.

- **[Day 4] 실습 숙제(과제 15%)**: `8puzzle.ipynb`에서 **A\* 알고리즘 + 휴리스틱 함수 구현** 및 작동 테스트. (출처: Notion Day4)

- **[Day 1] 교수 강조 질문**: UCS의 Min-Heap 정렬 비용은 평가에서 무시하는가? b 증가 시? → 복잡도 평가의 가정(노드 생성 수 기준)을 이해하라는 의도. (출처: Notion Day1)

- **[Day 4] 교수 강조 질문(빨강)**: α-β pruning의 **ordering 효율화**, 모든 child utility 동일 시 depth 확장 여부, alliance 상황의 α-β 적용 가능성. (출처: Notion Day4)

- **⚠️ Notion 노트의 오류/주의 (교차검증 결과, 학생 노트가 PDF와 불일치 → PDF가 정답)**:
  - Day1 표: BFS=Stack(FIFO), DFS=Queue(LIFO)로 **자료구조가 뒤바뀜**. **정답(PDF): BFS=FIFO Queue, DFS=LIFO Stack.**
  - Day6: "Gini 계수는 높을수록 더 좋다" → PDF의 Gini=Σpⱼ² 정의에서는 **순도(purity)가 높을수록 Gini가 1에 가까움**(한 클래스만 있으면 Σpⱼ²=1). 즉 split 후 **자식 노드의 Gini가 높아지는** 쪽이 좋은 split. (엔트로피와 방향 반대인 건 맞으나 "Gini 계수=불평등 지수"라는 표현은 경제학 Gini와 혼동 주의 — 본 강의 정의는 purity 측도.)
  - Day8: single-link="Best/Max", complete-link="Worst/Min" → PDF 기준 **single-link sim = max(가장 가까운 쌍), complete-link sim = min(가장 먼 쌍)**. 라벨은 맞으나 "Best/Worst"보다 max/min 정의로 외울 것.

---

## 2. 개념 카탈로그

### C1. 탐색 문제 정식화 & 평가 4지표   ⭐高
- **출처**: 1.UninformedSearch p.5,19 / Notion Day1
- **선행 개념**: 없음 (시작점)
- **한 줄 직관**: 상태=노드, 연산자=엣지인 그래프를 탐색트리로 펼쳐 목표를 찾는다.
- **정의**: search space = states + operators (그래프). search tree = 그 그래프의 한 탐색 전개. search strategy = **노드 확장 순서**의 선택.
- **평가 4지표**:
  - **Completeness**: 해가 존재하면 반드시 찾는가?
  - **Optimality**: 항상 최소비용(또는 최소깊이) 해를 찾는가?
  - **Time complexity**: 생성된 노드 수 (worst case)
  - **Space complexity**: 메모리에 동시에 올라가는 최대 노드 수
- **3대 파라미터**: **b**=최대 분기수(branching factor), **d**=최소비용해의 깊이, **m**=상태공간 최대 깊이(∞일 수 있음).
- **시험 포인트**: 4지표 정의 + b/d/m 의미를 정확히. 서술형 단골.

### C2. BFS (Breadth-First Search)   ⭐高
- **출처**: 1.UninformedSearch p.6,16,17 / Notion Day1
- **선행 개념**: C1
- **한 줄 직관**: 가장 얕은 미확장 노드 먼저 = 레벨 순서.
- **자료구조**: **FIFO Queue** (새 successor를 큐 끝에 추가). ⚠️ Notion Day1은 이를 Stack으로 잘못 표기.
- **표 인사이트 (완전성·최적성·복잡도)**:
  | | 값 | 왜 |
  |---|---|---|
  | Complete | Yes | 유한 b면 결국 모든 레벨 도달 |
  | Optimal | Yes | (단계비용 동일 가정) 얕은 해 먼저 |
  | Time | O(b^d) (정밀히 O(b^{d+1})) | 깊이 d까지 노드 누적 |
  | Space | O(b^d) | **지수 메모리 = 치명적 약점** |
- **정량 예 (b=10, 10⁴ node/s, 1KB/node)**: d=8 → ≈10⁹노드/≈31시간/1TB; d=12 → ≈10¹³노드/≈35년/**10PB**. → BFS의 실질 한계는 **시간보다 메모리(지수 공간)**.
- **흔한 함정**: "Time도 약점"이라 답하기 쉬우나 슬라이드 강조는 **exponential SPACE**.
- **시험 포인트**: 자료구조(FIFO), 4지표, "왜 실무에서 못 쓰나(메모리)".

### C3. DFS (Depth-First Search)   ⭐高
- **출처**: 1.UninformedSearch p.10,18,19,20 / Notion Day1
- **선행 개념**: C1
- **한 줄 직관**: 가장 깊은 미확장 노드 먼저, 막히면 backtrack.
- **자료구조**: **LIFO Stack** (새 successor를 스택 앞에 추가). ⚠️ Notion Day1 뒤바뀜.
- **표 인사이트**:
  | | 값 | 왜 |
  |---|---|---|
  | Complete | No (트리 깊이 무한이면) | 무한 경로에 빠질 수 있음 |
  | Optimal | No | 먼저 찾은 해가 최소가 아닐 수 있음 |
  | Time | O(b^m) | 최악 전체 깊이 탐색 |
  | Space | **O(bm) = Linear** | 깊이 깊어져도 메모리 일정 = **최대 장점** |
- **BFS vs DFS 비교 인사이트**: Time은 같지만 — BFS는 worst-case 일반적으로 우수, 무한경로/많은 loop/얕은 해/작은 공간에 유리; DFS는 **메모리 압도적 우위(선형)**, 해가 많고 loop 적고 무한경로 없을 때 유리.
- **시험 포인트**: 자료구조(LIFO), DFS의 핵심 장점=선형공간, 비완전성 조건(무한깊이).

### C4. Depth-Limited & IDS (Iterative Deepening Search)   ⭐高
- **출처**: 1.UninformedSearch p.20,21,22 / Notion Day1
- **선행 개념**: C2, C3
- **한 줄 직관**: DFS를 depth-limit L을 1씩 키우며 반복 → DFS의 선형 메모리 + BFS의 완전성·최적성.
- **Depth-Limited(L)**: 무한경로 문제 해결하지만 해가 L 아래면 불완전. L은 문제 지식(예: state-space 지름)으로 선택.
- **IDS 의사코드**: `L=1; while no solution: DFS(initial, cutoff=L); if found return; else L++`
- **복잡도/성질**: Space **O(bd)**, Time O(b^d), **Complete Yes**, **Optimal Yes** (단, 경로비용이 깊이의 비감소 함수일 때).
- **"낭비 아닌가?" 인사이트**: 반복 탐색이지만 대부분 시간은 가장 깊은 레벨 d에서 소모 → BFS 대비 약 11%만 더 듦 (b=10,d=5: N(IDS)≈123,450 vs N(BFS)≈111,110, 비율 ≈ b/(b-1)). **큰 공간 + 미지의 해 깊이일 때 선호되는 uninformed 표준 방법**.
- **시험 포인트**: "IDS는 왜 좋은가"(DFS 메모리 + BFS 완전성), 낭비가 적은 이유.

### C5. UCS (Uniform Cost Search)   ⭐高
- **출처**: 1.UninformedSearch p.23,24,25 / Notion Day1
- **선행 개념**: C2
- **한 줄 직관**: g(n)=시작~n 경로비용이 최소인 fringe 노드를 항상 확장 (BFS의 비용일반화).
- **자료구조**: **Min-Heap (priority queue)**. (Notion Day1 표는 여기만 맞게 Min Heap 표기.)
- **수식**: g(n)=시작 상태 S에서 노드 n까지의 누적 경로비용. 항상 min g(n) 확장.
- **최적성/완전성 증명 아이디어 (PDF에 명시)**:
  - 가정: 모든 step 비용 ≥ ε > 0.
  - **Completeness**: ε>0 + 유한 b ⇒ 목표 비용에 도달하기까지 유한 횟수 확장 ⇒ 유한 단계 내 도달.
  - **Optimality(모순법)**: UCS가 비최적이라 가정 → 더 작은 비용의 목표가 존재 → 그러나 UCS는 정의상 그 노드를 **먼저** 확장했을 것 → 모순.
- **복잡도**: 시간·공간 **O(b^{1+⌊C*/ε⌋})** (C*=최적해 비용). ⌊C*/ε⌋ ≈ 모든 비용이 거의 같을 때의 해 깊이.
- **흔한 함정**: 비용이 모두 같으면 BFS와 동일하게 동작.
- **시험 포인트**: Min-Heap 사용, 최적성 보장 조건(ε>0, 비음수 비용), 모순법 증명 골자.

### C5b. Uninformed 종합 비교표   ⭐高
- **출처**: 1.UninformedSearch p.24
- | Criterion | BFS | UCS | DFS | Depth-Limited | IDS |
  |---|---|---|---|---|---|
  | Complete? | Yes | Yes | No | No | **Yes** |
  | Time | O(b^{d+1}) | O(b^{⌈C*/ε⌉}) | O(b^m) | O(b^L) | O(b^d) |
  | Space | O(b^{d+1}) | O(b^{⌈C*/ε⌉}) | **O(bm)** | O(bL) | **O(bd)** |
  | Optimal? | Yes | Yes | No | No | **Yes** |
- **인사이트(시험 단골 매칭형)**: "선형공간 & 완전 & 최적" = **IDS**가 유일. DFS만 선형공간이나 불완전·비최적. BFS/UCS는 최적이나 지수공간.

### C6. Informed Search & Heuristic Function   ⭐高
- **출처**: 2-1.HeuristicSearch p.2,3 / Notion Day2
- **선행 개념**: C1
- **한 줄 직관**: 문제별 휴리스틱 h(n)으로 "목표에 가까워 보이는" 노드를 우선 → 평균 속도 대폭 향상(8-puzzle).
- **정의**: Best-first search = 평가함수 f(n)("desirability")이 낮은 노드 우선 확장. h(n) = n→goal 추정비용("rule of thumb"), h(goal)=0. 예: 직선거리(SLD).
- **8-puzzle 두 휴리스틱**: **h₁** = 잘못 놓인 타일 수(misplaced tiles), **h₂** = 타일들의 맨해튼 거리 합. 예시 상태: h₁=8, h₂=18.
- **시험 포인트**: heuristic 정의, best-first의 일반형 f(n), h₁/h₂ 정의·계산.

### C7. Greedy Best-First Search   ⭐中
- **출처**: 2-1.HeuristicSearch p.4~8 / Notion Day2
- **선행 개념**: C6
- **한 줄 직관**: f(n)=h(n)만 사용 — "목표에 가장 가까워 보이는" 노드만 확장.
- **성질**: **Complete? No**(상태 추적 안 하면 loop, DFS처럼 막힘) / **Optimal? No**(반례: 루마니아 Arad→Bucharest에서 Greedy는 Fagaras 경유 비최적 경로 선택) / Time·Space O(b^m).
- **흔한 함정**: g(n)을 무시하므로 "가까워 보여도" 실제 비용 큰 경로 선택.
- **시험 포인트**: Greedy가 비최적인 이유 = 누적비용 g 무시.

### C8. A* Search   ⭐高
- **출처**: 2-1.HeuristicSearch p.9~12,13 / Notion Day2
- **선행 개념**: C6, C7
- **한 줄 직관**: f(n)=g(n)+h(n) — 지나온 비용 + 남은 추정비용을 합쳐 균형.
- **수식**: **f(n)=g(n)+h(n)**. g(n)=시작~n 실제비용, h(n)=n~goal 추정비용. (Notion Day2: g=엣지비용 합, h=목표까지 최단(맨해튼)거리)
- **성질**: Complete Yes(f≤f(G)인 노드 유한 시) / **Optimal Yes**(admissible h이면 TREE-SEARCH에서) / 또한 **optimally efficient**(같은 h에서 다른 어떤 최적 알고리즘도 A*보다 적게 확장 못함) / Time·Space 최악 지수.
- **A 알고리즘 vs A\* (Notion Day2)**: 차이 = **admissible heuristic** 사용 여부. 최적성 보장에 **엣지 비용 ≥ 0 (음수 불가)** 필요.
- **시험 포인트**: f=g+h 의미, A*가 최적인 조건(admissible), optimally efficient.

### C9. Admissibility & Consistency, Dominance, Relaxed Problems   ⭐高
- **출처**: 2-1.HeuristicSearch p.12,14,15,16 / Notion Day2
- **선행 개념**: C8
- **Admissible(허용성)**: 모든 노드 n에서 **h(n) ≤ h\*(n)** (h\*=실제 최적비용). 즉 **절대 과대평가하지 않음 = optimistic**. 예: 직선거리 SLD는 실제 도로거리를 절대 초과 안 함 → admissible.
  - **정리**: h admissible ⇒ A*(TREE-SEARCH) optimal.
- **Dominance(우월)**: 두 admissible h₁,h₂에 대해 모든 n에서 **h₂(n) ≥ h₁(n)이면 h₂가 h₁을 dominate** → h₂가 탐색에 더 효율적(실제값에 가까울수록 좋음, Notion Day2 강조).
  - **정량 인사이트 (8-puzzle 평균 확장 노드)**: d=12 → IDS 3,644,035 / A\*(h₁) 227 / **A\*(h₂) 73**. d=24 → A\*(h₁) 39,135 / **A\*(h₂) 1,641**. → 더 강한(dominant) 휴리스틱이 수십~수백 배 적게 확장.
- **Relaxed Problem(완화 문제)로 휴리스틱 발명**: 제약을 푼 문제의 최적해비용 = 원문제의 admissible heuristic.
  - 8-puzzle: 타일이 **아무 데나** 이동 가능(완화) → h₁(misplaced) 가 최단해. 타일이 **인접칸**으로 이동 가능(완화) → h₂(맨해튼)가 최단해.
- **흔한 함정**: admissible은 "과소추정"이 아니라 "**과대추정 안 함(≤)**". consistency(일관성)는 PDF에서 명시적으로 깊게 다루지 않음(⚠️ 아래 누락 참조).
- **시험 포인트**: admissible 정의(부등식·optimistic), dominance가 효율에 주는 영향, relaxed problem으로 h 만드는 법.

### C10. Local Search & Hill-Climbing   ⭐高
- **출처**: 2-2.LocalSearch (전체) / Notion Day3
- **선행 개념**: C6
- **한 줄 직관**: 경로를 기억하지 않고 **현재 한 상태**만 들고 이웃으로 이동, 목적함수 개선만 추구.
- **언제 쓰나**: 경로가 무의미하고 최종 상태만 중요한 문제(예: 8-queens, n-queens). 장점: **메모리 거의 안 씀**, 크거나 무한(연속) 공간에서 합리적 해.
- **Pure optimization 문제 4성질**: 모든 상태에 목적/휴리스틱 함수, 목표=max/min 값 상태 찾기, path-cost/goal-state 틀에 안 맞음, local search가 잘 함.
- **Hill-Climbing**: 값이 증가하는 방향으로 계속 이동, peak에서 종료 ("a kind of greedy local best search", "안개 속 에베레스트 정상 찾기"). 즉각 이웃만 보고 lookahead 없음.
- **약점 & 해법**: **local maxima/minima**에서 suboptimal로 멈춤. 해법 = **Multiple Random Restarts**(여러 번 무작위 재시작). (Notion Day3: "but how many times?", Day5: Greedy local search로는 shortest path 못 찾고 가다 끝남, restart하면 state는 찾음.)
- **8-queens 사례 (정량)**: h=서로 공격하는 queen 쌍 수(최소화). 무작위 시작 시 **14%만 해결, 86%는 local minimum에 빠짐**. 성공 시 평균 4 step, 실패 시 평균 3 step(상태공간 ~17M). Random restart 분석: p=0.14, 99% 성공에 필요 재시작 n: (0.86)ⁿ<0.01 → n>30.5 → ≈31회, 최대 step ≈4×31=124.
- **시험 포인트**: local search의 특징(상태 1개·경로 무시·저메모리), hill-climbing이 suboptimal인 이유(local maxima)와 random restart 해법.

### C11. Adversarial Search: 게임 설정 & Minimax   ⭐高
- **출처**: 3.Adversarial p.1~7 / Notion Day4
- **선행 개념**: C1
- **한 줄 직관**: 적(상대)이 내 이익을 깎으려 둔다고 가정하고, **최악의 경우를 최대화(maximin)** 하는 수를 둔다.
- **게임 가정 4가지**: **Deterministic, Turn-taking, Zero-sum, Perfect information**. 두 플레이어 MAX·MIN, MAX가 먼저, 교대.
- **Search vs Games**: 일반 탐색의 해=목표/경로; 게임의 해=**모든 상대 응수에 대한 전략(strategy)**. 시간제한 때문에 보통 근사해.
- **게임의 4요소**: Initial state, Successor function(합법수 목록), Terminal test, Utility function(예: tic-tac-toe win +1 / lose −1 / draw 0).
- **트리 크기**: O(b^d), b=분기수, d=양 플레이어 합산 수. 체스 b~35, d~100 → ≈10¹⁵⁴ (탐색 불가).
- **Minimax 값 (수식)**:
  - MINIMAX(n) = UTILITY(n) [terminal]; max_{s∈succ} MINIMAX(s) [MAX 노드]; min_{s∈succ} MINIMAX(s) [MIN 노드].
  - 가정: **양쪽 모두 최적(infallible) 플레이** → minimax는 MAX의 worst-case 결과를 최대화.
  - **MIN이 비최적이면?** MAX는 오히려 더 좋아짐(손해 없음).
- **복잡도**: Time **O(b^d)** (☹), Space **O(bd)** (☺, DFS 방식).
- **Multiplayer**: minimax 값이 벡터(UtilityA,B,C)로. 비zero-sum이면 동맹(alliance)이 유효(2인이라도).
- **시험 포인트**: 4가정, minimax 재귀 정의, 복잡도, "MIN이 실수하면 MAX 손해 없음".

### C12. Alpha-Beta Pruning   ⭐高
- **출처**: 3.Adversarial p.9~15 / Notion Day4
- **선행 개념**: C11
- **한 줄 직관**: **최종 결정에 영향 없는 가지를 잘라** minimax와 **동일한 결과**를 더 빨리.
- **메커니즘**: DFS로 단일 경로만 보며 — **α** = MAX 경로에서 지금까지 찾은 최고값(하한), **β** = MIN 경로에서 지금까지 찾은 최저값(상한). 노드 값이 조상의 α(MAX)/β(MIN)보다 나빠지는 순간 나머지 가지 prune. (예시: MAX≥3 확정 후 다른 MIN 노드가 ≤2가 되면 그 MIN의 나머지 자식은 잘림.)
- **복잡도 (표 인사이트)**:
  - **Worst-case: O(b^d)** — 가지가 최악 순서로 정렬되어 prune 전혀 안 됨(=minimax와 동일).
  - **Best-case: O(b^{d/2})** — 각 플레이어의 최선수가 항상 leftmost(완벽한 ordering)일 때 최대 prune.
  - 실무 평균은 best-case에 가까움 → 유효 분기수 √b. 체스 b~35 → ~6 → **같은 시간에 약 2배 깊이 탐색 가능**.
- **핵심 함정**: prune해도 결과는 minimax와 **항상 동일**. 효과는 **move ordering**에 크게 좌우(Day4 교수 질문: ordering 효율화?).
- **시험 포인트**: α/β 정의, prune 조건, best O(b^{d/2})/worst O(b^d) 와 ordering 의존성.

### C13. Utility / Evaluation Function   ⭐中
- **출처**: 3.Adversarial p.16 / Notion Day4
- **선행 개념**: C11
- **정의**: 비terminal 보드의 좋음 정도를 추정. 보통 (내 점수)−(상대 점수). Othello: 흰돌 수−검은돌 수. 체스: **선형 가중합** Eval(s)=w₁f₁(s)+...+wₙfₙ(s) (예 w₁=9, f₁=흰퀸수−검은퀸수). 범위 [−∞,+∞] 또는 [−1,+1].
- **실세계 (PDF)**: Checkers Chinook(1994 인간챔피언 격파), Chess Deep Blue(1997 Kasparov 격파, α-β 기반), Go는 b>300·d>>150로 당시 탐색 불가.
- **시험 포인트**: evaluation function의 정의와 선형가중합 형태.

### C14. CSP 정식화 & 종류   ⭐高
- **출처**: 4.CSP p.1~10 / Notion Day5
- **선행 개념**: C1
- **한 줄 직관**: 상태를 **변수 Xᵢ + 도메인 Dᵢ + 제약(constraints)**으로 정의 → 일반목적 알고리즘 사용 가능.
- **정의**: 상태=지금까지의 변수 할당, goal test=모든 제약을 만족하는 **complete & consistent** 할당. (일반 탐색의 "black box 상태"보다 구조화 → 더 강력.)
- **대표 예**: **Map-Coloring**(인접 지역 다른 색; 호주 WA,NT,Q,NSW,V,SA,T / D={red,green,blue}). **Constraint graph**: 노드=변수, 아크=제약(binary CSP).
- **변수 종류**: Discrete-finite(n변수, 도메인크기 d → **O(dⁿ)** 완전할당), Discrete-infinite(정수/문자열, 제약언어 필요 e.g. StartJob₁+5≤StartJob₃), Continuous(선형제약은 LP로 다항시간).
- **제약 종류**: **Unary**(SA≠green), **Binary**(SA≠WA), **Higher-order**(3변수↑, cryptarithmetic — TWO+TWO=FOUR, alldifferent(F,T,U,W,R,O) + 자리올림 제약).
- **표준 탐색 정식화**: 초기=빈 할당{}, successor=미할당 변수에 충돌 없는 값 부여, goal=complete&consistent. 모든 CSP 동일.
- **복잡도 인사이트**: 해는 깊이 n에 나타남 → DFS. 순진하게는 가지 b=(n−l)d → n!·dⁿ leaf지만, 할당이 **commutative**("WA=red then NT=green" = 그 반대)이므로 한 노드에서 **한 변수만** 고려 → b=d, **dⁿ leaf**.
- **시험 포인트**: CSP 3요소(변수/도메인/제약), complete&consistent, dⁿ 복잡도, constraint graph.

### C15. Backtracking Search + 효율 향상 휴리스틱   ⭐高
- **출처**: 4.CSP p.13~22 / Notion Day5
- **선행 개념**: C3, C14
- **한 줄 직관**: 한 노드에 한 변수만 할당하는 DFS(=backtracking). 막히면 되돌아감. n-queens n≈25까지 풀림.
- **의사코드 골자**: `RECURSIVE-BACKTRACKING(assignment, csp)`: 완전하면 반환 → 미할당 변수 선택 → 각 값에 대해 consistent하면 추가하고 재귀, 실패 시 제거 후 다음 값. 모두 실패하면 failure.
- **효율 향상 4종 (일반목적 휴리스틱, 큰 속도 향상)** — Notion Day5 강조 / PDF:
  - **MRV (Most Constrained Variable / Minimum Remaining Values)**: 남은 합법값이 가장 적은 변수를 먼저 → 바로 다음 child의 탐색공간 최소화.
  - **Most Constraining Variable (Degree heuristic)**: MRV 동점 타이브레이커 — 남은 변수들에 제약을 가장 많이 거는 변수 → grandchild의 분기수 최소화.
  - **LCV (Least Constraining Value)**: 변수 정했으면 남은 변수의 값을 가장 적게 배제하는 값을 선택. (MRV+LCV 조합 시 1000-queens 가능.)
  - **Forward Checking**: 미할당 변수의 남은 합법값을 추적, 어떤 변수의 합법값이 0이 되면 즉시 종료(=조기 실패 감지).
- **워크드 예제**: 4-Queens — X1..X4 도메인{1,2,3,4}, forward checking으로 도메인 좁히며 진행, 막히면 backtrack, 최종 해 **X1=2, X2=4, X3=1, X4=3**.
- **흔한 함정**: MRV(변수 선택)와 LCV(값 선택)는 **방향이 반대**(변수는 가장 제약 많은 것, 값은 가장 제약 적은 것).
- **시험 포인트**: backtracking=한 변수씩 DFS, MRV/degree/LCV/forward-checking 각각이 "무엇을 최소화/언제 쓰나".

### C16. Local Search for CSP (min-conflicts)   ⭐⭐高 (졸업시험 단골)
- **출처**: 4.CSP p.22,23 (Local Search for CSPs) / Notion Day5 ("졸업 시험 단골 문제!")
- **선행 개념**: C10, C15
- **한 줄 직관**: 모든 변수를 일단 할당(complete state, 제약 위반 허용)한 뒤 **충돌을 줄이는 방향**으로 값을 재배정(hill-climbing).
- **메커니즘**: 변수 선택=무작위로 **충돌하는(conflicted) 변수** 선택. 값 선택=**min-conflicts 휴리스틱**(위반 제약 수가 최소가 되는 값) = h(n)=총 위반 제약 수로 hill-climbing.
- **정량 인사이트**: 4-queens는 4⁴=256 상태, h=공격 수. 무작위 초기 상태에서 **임의 n에 대해 거의 상수 시간**에 해결(예 n=10,000,000도 높은 확률로).
- **Hill-Climbing vs Simulated Annealing (Day5 강조)**: hill-climbing은 local optimum에 갇힘 → SA는 확률적으로 나쁜 수도 허용해 탈출(상세 SA는 ⚠️ PDF 미수록).
- **시험 포인트**: ⭐⭐ min-conflicts 절차(complete state→conflicted 변수→위반 최소 값), 왜 빠른가, hill-climbing의 한계와 SA. **최우선 암기.**

### C17. kNN (k-Nearest Neighbor) Classifier   ⭐高
- **출처**: 5.Instance-Based Learning(kNN) (전체) / Notion Day6
- **선행 개념**: 없음(거리 개념)
- **한 줄 직관**: 새 인스턴스를 **가장 가까운 k개 이웃의 다수결(majority vote)**로 분류. 모델을 학습하지 않음.
- **정의**: instance-based / **lazy learning** — 분류함수를 국소적으로만 근사, **모든 계산을 질의 시점으로 미룸**(pre-training 없음, 데이터만 보유). k=1이면 최근접 1개 클래스로.
- **거리 수식**:
  - **Euclidean**: $d_E(x,y)=\sqrt{\sum_{i=1}^{N}(x_i-y_i)^2}$
  - **Absolute/Manhattan**: $d_A(x,y)=\sum_{i=1}^{N}|x_i-y_i|$
  - 인스턴스 = n차원 벡터 ⟨a₁,...,aₙ⟩.
- **전처리 절차 (시험 단골 워크드 예제 — PlayTennis)**:
  1. **범주형→수치 변환**: Outlook{Rain=0,Overcast=1,Sunny=2}, Temp{Cool=0,Mild=1,Hot=2}, Humidity{Normal=0,High=1}, Wind{Weak=0,Strong=1}. 예: ⟨Sunny,Hot,High,Weak⟩→⟨2,2,1,0⟩.
  2. **정규화(scaling)**: $x'=\frac{x-\bar{x}}{\sigma(x)}$ — 차원 크기 차이로 인한 왜곡 완화, 정확도 향상. (그림 인사이트: X차원이 크면 정규화 전엔 white로 오분류, 정규화 후 black로 정분류.)
  3. **k개 최근접 질의 → majority voting**. 워크드 예: query=⟨Sunny,Cool,High,Strong⟩→⟨1.18,−1.18,1,1⟩, k=3 → 최근접 Day2(No),Day7(Yes),Day11(Yes) → t_q=(0+1+1)/3=2/3 ≥ 1/2 → **PlayTennis=YES**.
- **개선**: **Distance-Weighted NN** — 이웃 가중치를 거리에 반비례(보통 1/d 또는 1/d²). k=전체이면 모든 인스턴스 기여 = **Shepard's method**(분류시간 큰 부담). **Voronoi diagram** = 1-NN 결정경계 시각화.
- **장단점 (표)**: Pros = 단순하나 노이즈·복잡 타깃함수에 효과적, 학습 거의 없음. Cons = **분류 시간 비용 큼(k 클수록, 매 질의마다 전 데이터와 거리계산)** → k-d tree로 완화.
- **흔한 함정**: "학습"이 아니라 lazy(질의 시 계산). scaling 안 하면 큰 스케일 축이 거리 지배.
- **시험 포인트**: lazy learning 정의, majority voting, 거리식 2종, **정규화의 필요성**, k 선택의 영향, Voronoi.

### C18. Decision Tree   ⭐高
- **출처**: 5-Supplementary.DecisionTree (전체) / Notion Day6
- **선행 개념**: 없음
- **한 줄 직관**: 내부노드=속성 테스트, 가지=속성값, 잎=분류값 → 데이터를 **순도 높은(homogeneous) 그룹**으로 재귀 분할.
- **정의**: 큰 이질적 집단을 더 작고 동질적인 그룹으로 나누는 split rule 집합. (예: PlayTennis 트리 — Outlook→{Sunny:Humidity, Overcast:Yes, Rain:Wind}.)
- **종류**: Binary(2분기), N-way/ternary(3분기 이상).
- **Best Split = Purity**: 잎이 한 클래스만 가지면 pure, 여러 클래스면 impure. **순도를 가장 많이 증가**시키는 split이 best (+ 비슷한 크기 노드 선호).
- **불순도 측도 4종**: Information Gain(entropy 기반), **Gini**, Information Gain Ratio, Chi-square. (강의는 **Information Gain & Gini만** 다룸.)
- **Entropy (수식)**: $E(p)=\sum_j -p_j\log_2 p_j$. p_j=클래스 j 확률. **균등분포에서 최대**(가장 무질서/놀람 큼). Notion Day6: "−log를 취하는 이유 = 놀람의 정도", 압축에서 빈출 알파벳=짧은 코드(엔트로피 이론).
- **Information Gain (수식)**: $Gain(S,A)=E(S)-\sum_{v\in Values(A)}\frac{|S_v|}{|S|}E(S_v)$. = 분할 전후 불순도 차이. **Gain 최대 속성이 root**.
- **워크드 예제 (Transportation, PDF)**: 데이터셋 entropy 1.571. 속성별 Gain: Gender 0.125, Car Ownership 0.534, **Travel Cost 1.210(최대)→root**, Income 0.695. Travel Cost로 split → Expensive=Car, Standard=Train(pure), Cheap=추가분할. 2차반복 root=Gender(Gain 0.322), 3차반복 Car Ownership로 마무리. (재귀: pure 잎은 고정, 나머지 반복.)
- **Gini (수식)**: $Gini=\sum_j p_j^2$. PDF 예: root 0.5²+0.5²=0.5(균등=impure), 잎 0.1²+0.9²=0.82(거의 pure). **Entropy 대신 Gini만 바꿔 넣으면** 동일하게 best split 결정. ⚠️ Notion "Gini는 높을수록 좋다"=순도 측면(한 클래스면 1) — 분할 후 자식 Gini 높을수록 pure.
- **장점**: 이해 쉬움, business rule로 매핑, 선험적 가정 없음, 수치·범주 속성 모두 처리.
- **단점**: 타깃은 **범주형이어야**, **단일 타깃**, **때때로 불안정**(local search 유사 — Day6 "왜?"), 수치속성 데이터셋은 트리 복잡.
- **시험 포인트**: ⭐ "Decision Tree에서 가장 중요한 게?"(=best split/purity/information gain) — Day7 교수 예시 질문. entropy·gain·gini 수식과 root 선택 절차. 장단점.

### C19. Clustering 개요 & k-means   ⭐高
- **출처**: 6.Clustering p.1~20 / Notion Day7
- **선행 개념**: C17(거리)
- **한 줄 직관**: 라벨 없는 데이터를 유사한 것끼리 묶는 **비지도학습**. (cluster 내 유사·cluster 간 상이.)
- **정의**: unsupervised learning의 가장 흔한 형태. 응용: IR(문서 군집), Google News, 검색결과 분류.
- **유사도/거리**: cosine similarity 또는 Euclidean distance(주로 dissimilarity). 문서=term vector. "trivial 클러스터(너무 크거나 작음) 회피"가 기본 원칙.
- **알고리즘 분류**: **Flat**(k-means) vs **Hierarchical**(Bottom-up agglomerative / Top-down divisive).
- **k-means**:
  - **centroid 수식**: $\vec{\mu}(c)=\frac{1}{|c|}\sum_{\vec{x}\in c}\vec{x}$ (term vector들의 평균=무게중심).
  - **의사코드**: K개 random seed 선택 → {각 doc를 가장 가까운(Euclidean) centroid의 cluster에 배정 → 각 cluster의 centroid 재계산} 수렴(또는 종료조건)까지 반복.
  - **종료 조건 3종**: ① 고정 반복수 ② 모든 cluster 불변 ③ 모든 centroid 위치 불변. (Notion Day7: ②와 ③은 보통 같으나 특수 상황에서 다름.)
  - **수렴(Convergence)**: 항상 fixed point 존재. **k-means = EM 알고리즘의 특수경우** → EM은 안정상태 수렴 증명됨. 반복수는 클 수 있으나 실무에선 작음.
  - **시간복잡도**: 거리계산 O(m), 재배정 O(Knm), centroid계산 O(nm), I회 반복 → **O(IKnm)** = **선형**. (m=차원, n=문서수, K=클러스터, I=반복.)
  - **초기 seed 민감성**: 결과가 초기 random seed에 따라 변함(나쁜 수렴/local optimum 가능). 해법 = 휴리스틱 seed(기존 seed에서 가장 먼 벡터) / 다중 시도(hill-climbing 성질). 예: {A,B,C,D,E,F}에서 B,E 시작→{A,B,C}{D,E,F}, D,F 시작→{A,B,D,E}{C,F}.
- **흔한 함정**: k-means는 local optimum/seed 민감 → 다중 restart. 종료조건 ②≠③(특수 경우).
- **시험 포인트**: centroid 수식, 4단계 절차, 종료조건 3종, **O(IKnm) 선형복잡도**, seed 민감성·EM 수렴.

### C20. Hierarchical Clustering (single/complete/group-average link)   ⭐高
- **출처**: 6.Clustering p.21~41 / Notion Day7, Day8
- **선행 개념**: C19
- **한 줄 직관**: 데이터를 트리(덴드로그램)로 묶는다 — bottom-up(병합) 또는 top-down(분할).
- **두 방식**: **Divisive(Top-Down)**=전체를 하나로 보고 재귀 분할(예: k-means로). **Agglomerative(Bottom-Up)**=개별 인스턴스에서 시작해 유사한 것 병합.
- **HAC 절차 (Day8)**: ① 각 n개 문서를 자기 자신만의 cluster로 ② 모든 쌍 cluster-cluster 유사도 계산 ③ 가장 유사한 쌍 병합 ④ cluster 수>1이면 ②로.
- **cluster-cluster 유사도 3종 (시험 핵심, Day8)**:
  | 방식 | sim 정의 | Notion Day8 라벨 | 성향 |
  |---|---|---|---|
  | **Single-link** | **max**(두 cluster 멤버 쌍 중 가장 가까운/유사한 쌍) | "가장 가까운 거리, Best/Max" | 적은 수의 크고 느슨한 cluster, **chaining 효과** |
  | **Complete-link** | **min**(가장 먼/덜 유사한 쌍) | "가장 먼 거리, Worst/Min" | 많은 수의 작고 단단한 cluster, IR에 더 적합하나 비용 큼 |
  | **Group-average** | 모든 멤버 쌍 평균 | "평균/가중평균" | 위 둘의 중간(intermediately bound) |
- **워크드 예제 (A~F, 6개, 15쌍 유사도)**:
  - **Single-link**: sim(AF,X)=max(sim(A,X),sim(F,X)). 병합 순서 AF(0.9)→AEF(0.8)→ABEF(0.8)→ABEFD(0.6)→ABCDEF(0.5). 임계값별 클러스터: 0.85→{AF,E,B,D,C}, 0.65→{AFEB,D,C}, 0.55→{AFEBD,C}.
  - **Complete-link**: sim(AF,X)=min(...). "checked list" 필요(병합하려면 모든 쌍이 이미 체크돼야). 임계값 0.85→{AF,B,E,C,D}, 0.65→{AF,BE,C,D}, 0.35→{AF,BCE,D}.
- **흔한 함정**: single-link=**max** sim(가장 가까운 쌍 기준), complete-link=**min** sim(가장 먼 쌍 기준) — 라벨이 헷갈리기 쉬움. single-link는 chaining으로 불공정 성장.
- **시험 포인트**: ⭐ single vs complete vs group-average의 **sim 정의(max/min/avg)와 성향(chaining vs tight)**, agglomerative 절차, 덴드로그램을 임계값으로 자르기.

### C21. Reinforcement Learning & Q-Learning   ⭐高
- **출처**: 7.Reinforcement_Learning-Q-learning (전체) / Notion (Day별 노트엔 없음 — 강의자료 기준)
- **선행 개념**: C10(탐색/greedy)
- **한 줄 직관**: 에이전트가 **시행착오로 reward를 받아** 점차 최적 행동을 강화. Q값을 통해 "각 상태에서 어떤 행동이 유리한지"를 테이블로 학습.
- **ML 3분류**: Supervised(라벨 O) / Unsupervised(라벨 X) / **Reinforcement**(에이전트가 행동에 reward 받아 강화). RL 특징: **delayed reward**(지금 행동의 적절성은 당장 모름), 결과 즉시 몰라도 무방, life-long learning 가능. 구조: Agent ↔ Environment (state, reward / action).
- **Q-Learning 정의**: RL 중 가장 널리 쓰임. 현재 상태에서 **임의의 action** 선택·실행 후 reward 받음(시행착오). 학습 미완 시점의 "최적" 평가 action이 실제 최적이 아닐 수 있음. **reward를 전파(propagate)** 하는 형태.
- **Q(s,a) 의미**: 상태 s에서 action a를 택하는 것의 유리한 정도(estimated utility) = a로 얻는 **즉각 reward + a로 바뀐 s'에서 얻을 잠재 reward의 최대값**의 합.
- **갱신 수식 (핵심)**:
  $$Q(s,a) = r(s,a) + \gamma \max_{a'} Q(s',a')$$
  - r(s,a)=즉각 reward(immediate), **γ**=reward 전파 계수[0,1](delayed reward 할인), s'=a 후 새 상태, a'=s'에서 가능한 action.
  - 최적 정책: $\pi(s)=\arg\max_a Q(s,a)$.
- **알고리즘**: 모든 (s,a)에 대해 Q=0 초기화 → {임의 action 선택·실행 → immediate reward 수신 → 새 s' 감지 → 위 식으로 Q 갱신 → s=s'} 를 **Q가 수렴할 때까지** 반복.
- **워크드 예제 (3×2 그리드, S1~S6, S6=END, γ=0.5, 목표 reward=100)**:
  - 모든 Q=0 시작. S3→S6(a36) 도달 시 **Q(S3,a36)=r=100**. 다음 게임에서 Q(S2,a23)=0+0.5·max(Q(S3,*))=0.5·100=**50**. Q(S1,a12)=0+0.5·max(Q(S2,*))=0.5·50=**25**. → reward가 목표에서 **뒤로 전파**.
  - 수렴 후 Q-table 예: S3,a36=100 / S2,a23=50, S5,a56=100 / S1,a12=25,S1,a14=25, S5,a54=25,S5,a52=25 / S2,a21=12.5, S4,a41=12.5 등.
- **그래프/도식 인사이트**: 목표(S6)에 가까운 상태일수록 Q값이 큼(100→50→25→12.5, γ=0.5씩 감쇠) → **Q값의 공간적 그래디언트가 최적 경로**를 가리킴.
- **흔한 함정**: Q는 "즉각 reward"가 아니라 "**즉각 + 미래 최대 Q의 할인합**". 학습 중에는 임의(탐험) action도 선택.
- **시험 포인트**: ⭐ Q(s,a) 갱신식과 각 기호(r, γ, max Q(s',a')), reward **역전파(propagation)** 개념, 수렴 시 정책 argmax, γ의 역할.

### C22. ANN: Perceptron & XOR 한계   ⭐高
- **출처**: 8.Artificial Neural Network p.1~10 / Notion (Day 노트 없음)
- **선행 개념**: C18(지도학습)
- **한 줄 직관**: 뉴런(입력·처리·출력=Dendrites·Nucleus·Axon)을 모방한 단위 **perceptron**. 입력·가중치 곱의 합이 임계치를 넘으면 발화.
- **뇌 vs 컴퓨터**: 뇌=10¹⁰뉴런, 분산·비선형·병렬 처리; 컴퓨터=빠르나(10⁻⁹s) 중앙·선형·순차.
- **Perceptron 구조**: 입력 pᵢ × 가중치 wᵢ의 합 Σ → **activation function** fₙ → 출력 y. activation 종류: **hard limiter, linear, sigmoid**.
- **학습 = Widrow-Hoff (delta) rule**: 지도학습. 출력값 y(t)와 목표값 d(t)의 차이를 최소화. 차이 없으면 가중치 불변, 있으면 줄이는 방향으로:
  $$W_i(t+1) = W_i(t) + \alpha[d(t)-y(t)]\,X_i(t), \quad 0\le i\le N-1$$
  (α=학습률). 절차: 가중치·임계치 초기화 → 입력/목표 제시 → activation으로 실제출력 계산 → d−y로 가중치 갱신.
- **AND 가능, XOR 불가**: AND는 단일 perceptron 가능(W0,W1=0.3 or 0.4, θ=0.5). **XOR은 만족하는 W0,W1 없음 = linearly non-separable** → 단일 perceptron으로 해결 불가. 해결: **2~3개 층(multi-layer)** → **3층 perceptron으로 어떤 문제도 근사 가능**. Perceptron이 MLP·backprop의 기반.
- **시험 포인트**: ⭐ perceptron 구조, **XOR이 안 되는 이유(linearly non-separable)** 와 multi-layer 해결, delta rule.

### C23. Backpropagation Neural Network (역전파 유도)   ⭐高
- **출처**: 8.Artificial Neural Network p.11~22 / Notion (Day 노트 없음)
- **선행 개념**: C22
- **한 줄 직관**: input–hidden–output 다층망에서 출력 오차를 **gradient descent로 뒤로 전파**해 가중치를 갱신, 단층의 선형분리 한계를 극복하고 연속함수 근사.
- **정의**: input과 output layer 사이 1개 이상 hidden layer를 갖는 단방향 신경망. 80년대 중반 **Error Backpropagation = generalized delta rule**. 학습 = 목표 d와 출력 o의 **오차제곱합 Error Function 최소화**.
- **Gradient Descent**: 오차함수의 기울기 반대방향으로 가중치 이동. step coefficient(η, 학습률) 너무 작으면 정체, 너무 크면 진동(PDF 곡선: 0.005 vs 0.05 비교).
- **Forward pass 수식**:
  - hidden 입력 $Input_{pj}=\sum_{i=0}^{N-1}X_{pi}W_{ij}-\theta_j$, 출력 $O_{pj}=f(Input_{pj})$ (f=sigmoid).
  - output 입력 $Input_{pk}=\sum_{j=0}^{M-1}O_{pj}W_{jk}-\theta_k$, 출력 $O_{pk}=f(Input_{pk})$.
- **Backward pass (핵심 유도)**:
  - Error: $E_p=\frac{1}{2}\sum_k (d_{pk}-O_{pk})^2$.
  - Sigmoid 도함수: $y=f(x)=\frac{1}{1+e^{-x}}$ → $\frac{\partial y}{\partial x}=y(1-y)$.
  - **Output layer delta**: $\delta_{pk}=(d_{pk}-O_{pk})\,O_{pk}(1-O_{pk})$ (= $-\partial E_p/\partial W_{jk}$의 일부).
  - **Hidden layer delta**: $\delta_{pj}=\left(\sum_k \delta_{pk}W_{jk}\right)O_{pj}(1-O_{pj})$ — 출력층 δ를 가중합해 뒤로 전파.
  - **가중치 갱신**: Hidden→Output $W_{jk}(t+1)=W_{jk}(t)+\eta\,\delta_{pk}\,O_{pj}$ ; Input→Hidden $W_{ij}(t+1)=W_{ij}(t)+\eta\,\delta_{pj}\,X_{pi}$.
- **전체 절차(PDF 12단계 요약)**: ①가중치·θ 초기화 ②입력 x·목표 d 제시 ③④hidden 입력·출력 계산 ⑤⑥output 입력·출력 계산 ⑦가중치 gradient 계산 ⑧Hidden→Output 갱신 ⑨Input→Hidden 갱신 ⑩모든 학습쌍 반복 ⑫출력층 오차합 E가 허용치 이하 또는 최대반복 초과면 종료.
- **문제점**: 상위층 오차를 하위로 역전파하는 것은 생물학적 현상과 불일치(실제 뉴런은 상위 목표를 모름). 그래도 가장 널리 쓰임.
- **응용**: 함수근사 — y=0.5(cos8x+sin4x−x+0.8)를 1-6-1 망으로 1,000→10,000→20,000회 학습 시 점점 정확히 근사.
- **흔한 함정**: hidden δ는 output δ의 **가중합 × sigmoid'**. Day7 시험형식상 **유도 손계산은 안 나옴** → "오차를 gradient descent로 뒤로 전파한다"는 **개념·절차**를 서술할 수 있으면 됨.
- **시험 포인트**: ⭐ 역전파의 개념(출력오차→gradient descent→가중치 갱신, 뒤로 전파), sigmoid 도함수 y(1−y), output/hidden δ의 형태와 가중치 갱신식, 단층 한계 극복. (증명 X, 개념 O.)

---

## 3. 개념 의존성 맵

```
탐색 줄기:
C1(탐색정식화·평가) → C2(BFS) → C5(UCS)
C1 → C3(DFS) → C4(IDS)
{C2,C3,C4,C5} → C5b(비교표)
C1 → C6(휴리스틱) → C7(Greedy) → C8(A*) → C9(admissible/dominance/relaxed)
C6 → C10(Local/Hill-Climbing)
C1 → C11(게임·Minimax) → C12(α-β) ; C11 → C13(evaluation fn)
C1 → C14(CSP정식화) → C15(Backtracking+휴리스틱) → C16(min-conflicts ⭐졸업단골)
        (C16은 C10 hill-climbing도 선행)
C3 → C14, C15 (backtracking=DFS)

학습 줄기:
(거리개념) → C17(kNN)
C18(Decision Tree) — 독립
C17 → C19(k-means) → C20(계층군집 single/complete/avg)
C10 → C21(Q-learning)  (greedy/탐험 개념 연결)
C18 → C22(Perceptron/XOR) → C23(Backpropagation)
```

- **학습 권장 순서**: 탐색 5종(C1→C5b) → 휴리스틱·A*(C6→C9) → Local(C10) → 게임(C11→C13) → CSP(C14→C16, ⭐C16 집중) → kNN(C17) → DT(C18) → 군집(C19→C20) → Q-learning(C21) → ANN(C22→C23).

---

## 4. 누락·확인 필요 목록

- ⚠️ **중간고사 기출 부재**: Notion 어디에도 중간고사 기출/문제는 없음. 평가 비중(Day1)은 출석10/과제15/**기말75**로 명시 — 사실상 중간 정보 없음. **기말 출제 경향 추정의 유일한 근거는 Day7 형식(객관식X·증명X·토픽당 1문제·8~10문제)과 Day5 "졸업시험 단골=CSP local search".** 이를 넘는 출제 예측은 추측이므로 하지 않음.
- ⚠️ **Consistency(일관성) 휴리스틱**: A* 슬라이드는 admissibility 중심이고 consistency(monotonicity, GRAPH-SEARCH 최적성 조건)는 명시적으로 깊게 다루지 않음. 시험형식상 admissible 위주로 충분할 가능성 높으나, consistency 정의는 PDF 미수록.
- ⚠️ **Simulated Annealing 상세**: Day5에서 "Hill-Climbing vs Simulated Annealing"을 강조하나, SA의 온도 스케줄·수용확률 수식 등 상세는 제공된 PDF(2-2 Local Search, 4 CSP) 어디에도 없음. SA는 "확률적으로 나쁜 수도 허용해 local optimum 탈출" 수준의 개념만 확인됨.
- ⚠️ **Q-learning / ANN의 Notion 노트 부재**: Day별 노트는 Day8(계층군집)까지만 존재. **Q-learning(7번)·ANN(8번)에 대한 Notion 강조점/시험 가이드 노트가 없음** → 해당 토픽은 PDF 단독 근거. (단, Day7 시험형식은 전 토픽에 공통 적용된다고 보는 것이 합리적이나, Q-learning·ANN이 기말 범위인지 명시 확인 불가 — 강의 진도상 포함 추정이나 ⚠️.)
- ⚠️ **Notion 학생 노트의 표기 오류 3건**(§1에 정리): Day1 BFS/DFS 자료구조 뒤바뀜, Day6 Gini 해석 혼동, Day8 single/complete-link "Best/Worst" 표현 — 모두 PDF를 진실기준으로 정정함. QA 시 학생 노트가 아니라 본 추출본 §1 정정을 따를 것.
- ⚠️ **5번 PDF 2개**: 과제 명세 "PDF 9개"와 달리 실제 `5.Instance-Based Learning(kNN)`과 `5-Supplementary.DecisionTree`가 별도 파일(총 10개). 둘 다 정독·수록 완료.
