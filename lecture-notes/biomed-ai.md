# 의생명과학을 위한 AI (노미나 교수) — 강의 자료 추출본 (재생성)
> 출처: 슬라이드 3개(1.Introduction · 2.gLM · 3.graphNN) + 핵심 논문 3개((Paper) Deep Learning · DNABERT · Graph Attention Networks) + 실습 2개(범위 밖) | 추출일: 2026-06-04
> ⚠️ **Notion 보강 실패**: 과목 Notion 페이지(ID 365fd22d-…) fetch 시 토큰 만료로 접근 불가 → 본 추출본은 **PDF 1차 자료만**으로 작성. 시험 가이드는 사용자 제공 시험범위(레지스트리)를 1순위 진실기준으로 사용.

## 0. 과목 개요

- **이 과목이 결국 푸는 문제**: 의생명 데이터(genomic 서열·분자 그래프·생물 네트워크)를 딥러닝으로 표현·예측하는 법. **의생명 ↔ AI의 overlap**만 다룬다(의생명 자체는 깊게 X). 3주 진도:
  - **1주 (Introduction + Deep Learning 논문)**: 생물학적 서열의 정보 흐름(복제·전사·번역), NGS·변이탐지 파이프라인, 딥러닝 근본(표현학습·**역전파 BP**·CNN·RNN). 변이탐지 AI(DeepVariant·NeuSomatic, pileup→CNN).
  - **2주 (gLM + DNABERT 논문)**: 서열을 언어로(gLM). **genomic 시퀀스 토큰화**(single-nucleotide / k-mer / BPE), 입력→토큰→임베딩 형상. 사전학습 MLM/CLM. **DNABERT**(k-mer + BERT + MLM).
  - **3주 (graphNN + GAT 논문)**: 그래프 신경망. **노드 업데이트(message passing: AGGREGATE+UPDATE)** 와 그 **수식**. GCN·GraphSAGE·**GAT(attention 가중 집계)**. 노드/엣지/그래프 레벨 예측.
- **출제 형태 (시험범위, 최우선)**: **큰 문제 6개 = 주차별 2개**. "입력을 어떻게 넣고 출력이 어떻게 나오나"를 식으로 서술. ① genomic 시퀀스 토큰화 ② BP(역전파) ③ **GNN 노드 업데이트 수식(식 문제 1개 확정)**. 논문 3편 → 포스터.
- **평가**: 4주차(6/9) in-class Review & Test (출처: 1.Introduction p.3 Course Schedule).

---

## 0-1. 선행 지식 키워드 (배경지식 — 본문에서 설명 없이 전제되는 것)

> 판별 기준: "이 용어를 모르면 본문 어느 노드에서 막히는가?" 막히는 토대만 선별(과목 내부 개념 C#는 제외). 카테고리: 수학 / CS기초 / 도메인(생물).

### PK1. [도메인] DNA·염기·서열
- **쉬운 정의**: DNA는 A·C·G·T 네 글자(염기)로 쓰인 생물의 정보 문자열. "서열(sequence)"은 이 글자들의 순서(예 ACTACG). RNA에선 T가 U로 바뀜.
- **왜 필요**: gLM(C7)·토큰화(C8~C10)가 "DNA=문장, 염기=글자"로 보는 전제. 이걸 모르면 토큰화가 무엇을 자르는지 안 잡힘.
- **연결**: C1, C7, C8, C9.

### PK2. [도메인] 유전자 발현 · 중심원리(Central Dogma)
- **쉬운 정의**: 유전자(DNA)가 RNA를 거쳐 단백질로 "발현"되는 정보 흐름(복제→전사→번역). DNA가 청사진이면 단백질은 실제 기능 부품.
- **왜 필요**: C1·C2가 "왜 DNA를 읽나"의 동기. 변이(글자 한 개 변화)가 단백질·질병을 바꾸는 이유의 배경.
- **연결**: C1, C2.

### PK3. [도메인] 변이(SNV) · 민감도 개념
- **쉬운 정의**: 한 위치의 염기가 바뀐 것(예 T→C)이 단일염기변이(SNV). 변이탐지는 "참조서열과 다른 글자"를 찾는 일.
- **왜 필요**: C2(NGS 파이프라인)·C6(CNN 변이탐지)의 목표 자체. pileup·VAF가 "변이인가"를 판정.
- **연결**: C2, C6.

### PK4. [CS기초] 원-핫 인코딩 · 임베딩
- **쉬운 정의**: 원-핫=범주를 "한 자리만 1, 나머지 0"인 벡터로(A=[1,0,0,0]). 임베딩=범주를 학습되는 짧은 실수 벡터로 바꿔 의미가 가까우면 벡터도 가깝게.
- **왜 필요**: C8(토큰화 형상)이 토큰ID→벡터로 바꾸는 두 방식. 원-핫(sparse·고정) vs 임베딩(dense·학습)을 모르면 "입력 형상" 서술 불가.
- **연결**: C8, C9.

### PK5. [CS기초] 신경망 · 퍼셉트론 · 활성함수
- **쉬운 정의**: 입력에 가중치를 곱해 더하고(가중합 z=Σwx) 비선형 함수 f를 씌워(y=f(z)) 출력하는 단위(뉴런)를 층층이 쌓은 것이 신경망. 활성함수=비선형(ReLU=max(0,z), sigmoid).
- **왜 필요**: C3·C4(BP)·C6·C11·C13 모든 학습의 기본 단위. z=Wx·y=f(z)를 모르면 forward/backward 식이 안 읽힘.
- **연결**: C3, C4, C5.

### PK6. [수학] 미분 · 연쇄법칙(chain rule) · 경사하강
- **쉬운 정의**: 미분=입력을 조금 바꿀 때 출력이 얼마나 변하나(기울기). 연쇄법칙=합성함수 미분은 각 단계 미분의 곱(∂z/∂x=∂z/∂y·∂y/∂x). 경사하강=기울기 반대로 조금씩 움직여 에러 최소화(w←w−η·∂E/∂w).
- **왜 필요**: C4(역전파)의 뼈대. "거꾸로 미분을 곱해 전파"가 곧 연쇄법칙. 시험 식 문제의 토대.
- **연결**: C4.

### PK7. [수학] 벡터 · 행렬 · 행렬곱
- **쉬운 정의**: 벡터=수의 1차원 묶음, 행렬=2차원 격자. 행렬곱은 (a×b)·(b×c)=(a×c) 형상으로 변환. 신경망 한 층 = 행렬곱 + 비선형.
- **왜 필요**: 토큰화 출력 형상 (L,d)·(512,768), GNN의 W·h, GCN의 인접행렬 곱을 다루려면 형상 감각 필수.
- **연결**: C8, C13, C15.

### PK8. [수학] 확률 · 소프트맥스 · 교차엔트로피
- **쉬운 정의**: 소프트맥스=점수 벡터를 합이 1인 확률분포로(exp 후 정규화). 교차엔트로피=예측확률이 정답에서 얼마나 빗나갔나의 손실(−Σ y' log y).
- **왜 필요**: MLM 손실(C11), GAT의 α 정규화(C16 softmax), 분류 손실(C18)이 전부 이 둘. softmax를 모르면 attention·손실식이 안 잡힘.
- **연결**: C11, C16, C18.

### PK9. [CS기초] 그래프 자료구조 (정점·간선·인접행렬·이웃)
- **쉬운 정의**: 그래프=점(정점/노드)과 점을 잇는 선(간선/엣지)의 모음 G=(V,E). 인접행렬 A: 연결되면 1, 아니면 0. 이웃 N(v)=v와 연결된 점들.
- **왜 필요**: 3주 전체(C12~C18)의 입력 자료구조. N(v)·A를 모르면 message passing·GCN 식이 무의미.
- **연결**: C12, C13, C15, C16.

### PK10. [CS기초] 사전학습→파인튜닝 (전이학습)
- **쉬운 정의**: 큰 데이터로 일반 능력을 먼저 배우고(사전학습), 작은 과제 데이터로 미세조정(파인튜닝)하는 2단계. 적은 라벨로도 강한 성능.
- **왜 필요**: gLM(C7)·DNABERT(C11)·실습(C19)의 학습 구조. "왜 사전학습이 이득인가"(DNABERT Fig.6)의 전제.
- **연결**: C7, C11.

---

## 1. 시험 가이드 (사용자 제공 시험범위 — 1순위 진실기준)

- ⭐⭐⭐ **시험 = 큰 문제 6개 = 주차별 2개**. (1주 Intro+DeepLearning / 2주 gLM+DNABERT / 3주 graphNN+GAT)
- ⭐⭐⭐ **GNN 식(수식) 문제 1개 확정 출제** — 특히 **3주차 노드 업데이트(message passing/aggregation)**. 기호 하나하나(h, m, N(v), W_neigh, W_self, σ, α_vu, e_vu) 풀이 필수.
- ⭐⭐ **출제형태 = "입력→출력 서술형"**. 구체적 입력 예시와 출력/형상(shape) 변화를 보일 것:
  - **genomic 토큰화**: DNA 서열 → 토큰화(single-nt/k-mer/BPE) → 토큰ID 시퀀스 → 임베딩 텐서 (batch, L, hidden).
  - **BP(역전파)**: forward(z=Wx, y=f(z)) → loss → backward(chain rule로 ∂E/∂z, ∂E/∂w 전파) → 가중치 갱신.
- ⭐⭐ **논문 3편 중심**, 슬라이드 보조. 논문 3편 → 별도 "논문 포스터"(terminology/concept/breakthrough).
- **실습 2개는 범위 제외** (추출하되 `범위 밖(참고)` 라벨).
- ⚠️ **기출 원문·Notion 강조점 없음**: Notion 접근 불가. 구체 문항·정답은 자료에 없으므로 추측 금지.

---

## 2. 개념 카탈로그

> 노드 ID는 주차로 묶음: 1주(C1~C6), 2주(C7~C11), 3주(C12~C18). 시험 직결도 ⭐로 표시.

### C1. 생물학적 서열과 정보 흐름 (Central Dogma)   ⭐低 (1주 배경)
- **출처**: 1.Introduction p.7 / Deep Learning 논문 (배경)
- **선행**: 없음
- **한 줄 직관**: "DNA(저장) → RNA(전달) → 단백질(기능)"의 정보 흐름. AI 입력이 되는 "생물학적 텍스트"의 출처.
- **정의/메커니즘**: ① 복제(Replication): DNA가 스스로 복제. ② 전사(Transcription): DNA → mRNA (염기 pairing A-U, T-A, G-C, C-G; 코돈 단위). ③ 번역(Translation): mRNA 코돈 → tRNA → 아미노산 서열(단백질). 예: ATG GCT TTT GGA CCT TAA → AUG GCU UUU GGA CCU UAA → Met-Ala-Phe-Gly-Pro-Stop.
- **시험 포인트**: 동기 배경. gLM이 "DNA=문장, 염기=글자"로 보는 근거. 깊은 출제보다 서론(의생명↔AI overlap 최소).

### C2. 변이와 질병 · NGS 변이 탐지 파이프라인   ⭐中 (1주)
- **출처**: 1.Introduction p.10–14, 19 / DeepVariant·NeuSomatic 슬라이드
- **선행**: C1
- **한 줄 직관**: "환자 vs 건강인 DNA를 읽어 한 글자(SNV) 차이를 찾아 질병 원인을 잡는다."
- **메커니즘 (입력→출력)**: 샘플 DNA → ① 시퀀싱(Reads, FASTQ) → ② Read Mapping(레퍼런스에 정렬, BWA-MEM/Seed-and-Extend, BWT·FM-index) → ③ Variant Calling(pileup으로 각 위치 염기 비교, Variant Allele Frequency 예: 2/10=20%) → ④ 변이 결과(VCF: Chr/Position/Ref/Alt/Type SNV/Gene/Effect Missense/AF/Quality).
- **예시**: 건강인 …GTGC**T**GGCCCAT… vs 환자 …GTGC**C**GGCCCAT… (T→C SNV).
- **시험 포인트 ⭐**: "입력(reads)→정렬→pileup→변이호출"의 흐름. BRCA1/2 변이→유방암 위험(정밀의료). 1주차 출제 후보.

### C3. 딥러닝 기초: 표현학습 (Representation Learning)   ⭐高 (1주, Deep Learning 논문)
- **출처**: (Paper) Deep Learning p.436 / 1.Introduction
- **선행**: 없음
- **한 줄 직관**: "사람이 feature를 손으로 설계하지 않고, 여러 층이 raw 데이터에서 점점 추상적인 표현을 스스로 배운다."
- **정의**: 딥러닝 = 여러 처리층(비선형 모듈)을 합성해 raw 입력을 점점 더 추상적인 표현으로 변환하는 representation-learning. 이미지 예: 1층=edge, 2층=motif, 3층=part, 4층=object. **"이 층들은 사람이 설계하지 않고 데이터로 학습"**이 핵심.
- **지도학습 절차**: 입력→점수벡터 출력 → 목적함수(에러)로 desired와 비교 → 가중치(수백만 knobs)를 에러 감소 방향으로 조정. **gradient vector** = 각 가중치를 조금 키우면 에러가 얼마나 변하는지 → 반대 방향으로 갱신.
- **시험 포인트 ⭐**: LeCun-Bengio-Hinton "근본 논문". "왜 딥러닝이 feature engineering보다 강한가"(표현학습). BP의 전제.

### C4. 역전파 (Backpropagation, BP) + SGD   ⭐⭐⭐高 (1주, 시험 확정 토픽)
- **출처**: (Paper) Deep Learning p.437 Fig.1 (b·c·d)
- **선행**: C3
- **한 줄 직관**: "출력에서 입력 방향으로 chain rule을 반복 적용해, 각 가중치가 에러에 얼마나 기여하는지(gradient)를 효율적으로 계산하는 절차."
- **정의**: BP = 다층 모듈의 가중치에 대한 목적함수 gradient를 구하는 절차 = **chain rule의 실용적 적용**. 핵심 통찰: 한 모듈의 입력에 대한 gradient는 그 출력에 대한 gradient(=다음 모듈 입력의 gradient)로부터 **거꾸로** 계산 가능. 출력층(top, 예측)에서 입력층(bottom)까지 반복 전파.
- **🧮 수식 (Fig.1c forward / 1d backward) ⭐⭐⭐**:
  - **Forward (순전파)**: 각 층에서 총입력 $z_l = \sum_k w_{kl}\, y_k$ (아래 층 출력의 가중합), 출력 $y_l = f(z_l)$. $f$=비선형(ReLU $f(z)=\max(0,z)$, sigmoid $1/(1+e^{-z})$, tanh).
  - **Chain rule (Fig.1b)**: $\Delta z = \frac{\partial z}{\partial y}\Delta y$, $\Delta y = \frac{\partial y}{\partial x}\Delta x$ → $\frac{\partial z}{\partial x} = \frac{\partial z}{\partial y}\frac{\partial y}{\partial x}$.
  - **Backward (역전파)**: 출력층 $\frac{\partial E}{\partial y_l} = y_l - t_l$ (cost $\frac{1}{2}(y_l-t_l)^2$일 때, $t_l$=정답). 그 다음 $\frac{\partial E}{\partial z_l} = \frac{\partial E}{\partial y_l}\frac{\partial y_l}{\partial z_l}$. 은닉층: $\frac{\partial E}{\partial y_k} = \sum_{l} w_{kl}\frac{\partial E}{\partial z_l}$, $\frac{\partial E}{\partial z_k} = \frac{\partial E}{\partial y_k}\frac{\partial y_k}{\partial z_k}$. **가중치 gradient**: $\frac{\partial E}{\partial w_{jk}} = y_j\,\frac{\partial E}{\partial z_k}$.
  - **갱신**: $w \leftarrow w - \eta\,\frac{\partial E}{\partial w}$ (gradient 반대 방향).
- **SGD**: 몇 개 예제로 평균 gradient를 추정해 가중치 갱신, 반복(목적함수가 더 안 줄 때까지). "stochastic"=소수 예제가 noisy 추정.
- **흔한 함정**: BP는 "마법"이 아니라 chain rule. 출력층 미분(y−t)에서 시작해 거꾸로. ReLU가 sigmoid보다 깊은 망에서 빠르게 학습(gradient 소실 완화).
- **시험 포인트 ⭐⭐⭐**: **"입력→forward→loss→backward→갱신"을 식으로 서술**. ∂E/∂z, ∂E/∂w 유도. 1주차 2문제 중 하나로 거의 확정. CNN/RNN도 같은 BP로 학습됨을 연결.

### C5. CNN vs Fully-Connected: 파라미터 공유   ⭐中 (1주)
- **출처**: 1.Introduction p.16–17 / Deep Learning p.439
- **선행**: C3
- **한 줄 직관**: "FC는 모든 픽셀↔모든 뉴런(파라미터 폭발), CNN은 작은 필터를 슬라이딩(국소 연결+가중치 공유)으로 파라미터를 극적으로 줄인다."
- **메커니즘/표 인사이트 ⭐**: 1000×1000 이미지, hidden 1M 유닛 →
  | | 연결 | 파라미터 |
  |---|---|---|
  | **Fully Connected** | 모든 입력↔모든 hidden, $z=\sum_d w_d x_d$ | **$10^{12}$ (폭발)** |
  | **CNN (10×10 필터)** | locally-connected, $z=\sum_{i,j} w_{ij}x_{ij}$, $i,j\ll d$ | **100M** |
- CNN 4대 아이디어: 국소 연결(local connection), 가중치 공유(shared filter bank), 풀링(max pooling으로 위치 불변성), 다층. 합성곱층=local motif 탐지, 풀링층=의미 유사 feature 병합.
- **시험 포인트 ⭐**: "왜 이미지에 CNN인가"(파라미터 공유·국소성). 3주차 GNN의 "이웃 가중치 공유"와 대조됨(C14).

### C6. CNN 기반 변이/biomarker 탐지: DeepVariant · NeuSomatic   ⭐中 (1주)
- **출처**: 1.Introduction p.18, 20–22 (DeepVariant: Nat Biotech 2018; NeuSomatic: Nat Comm 2019)
- **선행**: C2, C5
- **한 줄 직관**: "정렬된 read 더미(pileup)를 이미지(텐서)로 만들어 CNN이 '변이인가'를 이미지 분류하듯 본다."
- **DeepVariant 메커니즘 (입력→출력) ⭐**: aligned reads + reference → candidate site → **pileup image = 텐서 (100, 221, 6)** (6채널: read base / base quality / mapping quality / strand / read-supports-variant / base-differs-from-ref) → CNN → genotype likelihoods [Hom-ref, Het, Hom-alt] (예 [0.01, 0.95, 0.04] → heterozygous).
- **NeuSomatic 메커니즘**: Tumor/Normal reads 정렬 → Reference/Tumor count/Normal count matrix (채널: ref, tumor·normal freq·coverage, position, base/map quality, strand, clip) → Conv 블록(1×3,3×3,5×5, BN+ReLU+Pool, residual) → 멀티헤드: Mutation type 분류 / Length 분류 / Position 회귀.
- **CNN 변이탐지 학습**: 환자=라벨1/건강인=라벨0, 손실 Binary Cross Entropy, **역전파(C4)로 가중치 갱신**.
- **시험 포인트 ⭐**: "서열을 이미지로 → CNN" 입력 변환 + 6채널 pileup 해석. 1주차 논문 해석. (한계: local pattern 집중 → 전역 문맥 약함 → 2주 Transformer 동기).

### C7. 서열을 언어로 — gLM 패러다임   ⭐高 (2주)
- **출처**: 2.gLM.pdf p.6 / 1.Introduction p.26 (MLM 도식)
- **선행**: C1, C3
- **한 줄 직관**: "DNA=문장, 뉴클레오타이드=글자로 보고 NLP의 pretrain→finetune을 그대로 적용."
- **정의/대응**: Genome/Gene/Protein ↔ Document/Sentence, Nucleotides/Amino acids ↔ Words, Motifs ↔ idioms/word order(국소 관계), Co-evolution ↔ context(원거리 관계). gLM = ① genomic grammar·distant interaction 식별, ② 아키텍처·토크나이징·사전학습·평가를 태스크에 최적화.
- **학습 2단계**: Random gLM → (대량 unlabeled genomic data) Pretraining → Pretrained gLM → (task data) Fine-tuning → Task-adapted.
- **시험 포인트 ⭐**: 2주 전체의 뼈대. DNABERT가 이 틀의 구체 사례(k-mer×encoder×MLM).

### C8. genomic 시퀀스 토큰화 ① Single-nucleotide & One-hot   ⭐⭐高 (2주, 시험 확정 토픽)
- **출처**: 2.gLM.pdf p.7–8 / 1.Introduction p.27 (아미노산 임베딩)
- **선행**: C7
- **한 줄 직관**: "글자(A/C/G/T) 하나하나를 토큰으로 — SNP 효과를 직접 모델링하나 시퀀스가 길어진다."
- **정의·입력→출력 형상 ⭐⭐**:
  - **One-hot**: 각 염기를 distinct binary vector (A=[1,0,0,0], C=[0,1,0,0], G=[0,0,1,0], T=[0,0,0,1]). 서열 "ACTACGACT"(L=9) → **4×9 행렬**. bias·정보손실 없으나 입력이 매우 길어짐(L↑→4×L).
  - **Single-nucleotide embedding**: 각 염기를 vocabulary 토큰ID로 → **학습가능 dense vector**로 임베딩. 서열 길이 L → 토큰ID 시퀀스 길이 L → 임베딩 (L, d_model). base-level 보존, SNP 효과 직접 모델링.
- **시험 포인트 ⭐⭐**: **"DNA 서열 입력 → 토큰화 → 토큰ID → 임베딩 형상"** 서술. single-nt vs k-mer vs BPE trade-off(C9·C10과 묶음). One-hot(sparse·고정) ≠ embedding(dense·학습).

### C9. genomic 토큰화 ② k-mer tokenization (DNABERT)   ⭐⭐高 (2주, DNABERT 채택)
- **출처**: 2.gLM.pdf p.9 / (Paper) DNABERT p.2113–2114 §2.2.1
- **선행**: C8
- **한 줄 직관**: "연속 k개 염기를 한 단어로 묶기 — non-overlapping은 꼬리가 잘리고, overlapping은 중복이 심하다."
- **정의·입력→출력 ⭐⭐**: 연속 k 뉴클레오타이드를 한 토큰으로.
  - **Non-overlapping**: 겹치지 않게 자름 → trailing base 손실 가능. 3-mer로 ACTACGACT → [ACT, ACG, ACT] (토큰 3개).
  - **Overlapping**: 한 칸씩 밀며 자름 → highly redundant. 6-mer로 ACTACG / CTACGA / TACGAC / ACGACT (토큰 ≈ L−k+1, 5/6글자 공유).
- **DNABERT의 k-mer (논문 ⭐)**: k∈{3,4,5,6}로 4개 모델(DNABERT-3/4/5/6). **vocabulary 크기 = $4^k + 5$** (4^k개 k-mer + 5개 특수토큰 [CLS],[PAD],[UNK],[SEP],[MASK]). 예: DNABERT-6 → $4^6+5 = 4101$. 논문은 k=6이 최고 성능.
- **입력 구성 (DNABERT)**: 서열을 k-mer로 토큰화 → [CLS] + k-mer 토큰들 + [SEP] → 각 토큰을 임베딩 행렬 M으로 변환(Token Embedding + Positional Embedding). max length 512. 12 Transformer 층, 768 hidden, 12 attention head.
- **시험 포인트 ⭐⭐**: "왜 overlapping이 중복인가 / non-overlapping은 무엇을 잃는가" + DNABERT vocab=$4^k+5$ 계산 + 입력 형상([CLS]…[SEP], 512).

### C10. genomic 토큰화 ③ Byte Pair Encoding (BPE)   ⭐中 (2주)
- **출처**: 2.gLM.pdf p.10
- **선행**: C8, C9
- **한 줄 직관**: "자주 같이 나오는 글자쌍을 반복 병합해 새 토큰을 만든다 — 데이터가 vocabulary를 정한다."
- **메커니즘**: base vocab {A,C,T,G}에서 시작 → 가장 빈번한 인접쌍을 새 토큰으로 반복 병합 → 목표 vocab 크기까지. 예: iter0 {A,C,T,G} → iter1 +{AC} → iter2 +{ACT} → iter3 +{ACTAC}. DNABERT-2가 BPE 채택(현대 추세).
- **시험 포인트 ⭐**: 3토큰화(C8–C10) trade-off 비교. 빈도 기반 → 도메인(genome)별 토큰 상이.

### C11. 사전학습 MLM / CLM (genomic)   ⭐高 (2주, DNABERT=MLM)
- **출처**: 2.gLM.pdf p.13–15 / 1.Introduction p.26 / (Paper) DNABERT p.2113 §2.2.2
- **선행**: C7
- **한 줄 직관**: "MLM=중간 글자 가리고 양방향 문맥으로 맞히기(이해), CLM=이전 토큰으로 다음 토큰 맞히기(생성)."
- **MLM (DNABERT 채택) ⭐**: 입력 토큰 일부(DNABERT: contiguous k-mer span ~15%)를 [MASK] → 주변 문맥으로 예측. **Encoder-based(Bidirectional Self-Attention)**. 양방향 → 이해(understanding). 예 ACG [MASK] ATA … [MASK] GCA. 손실 cross-entropy $L = -\sum_i y_i' \log(y_i)$ ($y_i'$=정답, $y_i$=예측확률).
- **DNABERT MultiHead (논문 eq 1–2)**: $\text{MultiHead}(M)=\text{Concat}(\text{head}_1,…,\text{head}_h)W^O$, $\text{head}_i=\text{softmax}\big(\frac{MW_i^Q\,M W_i^{K\,T}}{\sqrt{d_k}}\big)MW_i^V$ — $M$=토큰 임베딩 행렬, $d_k$=key 차원. self-attention으로 전역 문맥 통합.
- **CLM**: autoregressive, 이전 토큰으로 다음 예측. Decoder-based(Causal Self-Attention), 단방향 → 생성·likelihood.
- **시험 포인트 ⭐**: MLM(양방향/이해) ↔ CLM(단방향/생성) 대비. MLM 손실식 + "self-supervised(라벨을 가린 토큰에서 만듦)". 사전학습이 random init보다 우월(DNABERT Fig.6: pre-trained가 낮은 loss로 수렴, generalizes to other organisms).

### C12. 그래프와 GNN 동기 (Why a special model for graph)   ⭐高 (3주)
- **출처**: 3.graphNN.pdf p.2–6
- **선행**: C5 (CNN과 대조)
- **한 줄 직관**: "관계(edge)가 핵심 정보인 데이터(소셜·유전자·분자)는 격자(이미지)·시퀀스(텍스트)에 안 맞아 별도 모델이 필요하다."
- **정의 ⭐**: 그래프 $G=(V,E)$. 노드집합 $V=\{v_1,…\}$, 엣지집합 $E$. 노드 feature $x_v\in\mathbb{R}^d$. 인접행렬 $A\in\mathbb{R}^{n\times n}$, $A_{uv}=1$ if $(u,v)\in E$ else 0. 이웃 $\mathcal{N}(v)=\{u\in V:(u,v)\in E\}$.
- **왜 특별한 모델? ⭐**: ① 그래프는 Euclidean 공간에 없음(좌표 없음). ② 노드마다 이웃 수가 가변 → **다양한 크기 이웃 처리** 필요. ③ 이웃에 정해진 순서 없음 → **permutation-invariant**(순서 불변) 필요. CNN(고정 격자)·RNN/Transformer(고정 순서)는 부적합.
- **3가지 예측 레벨**: Node-level(노드 라벨, 예 소셜 노드분류), Edge-level(엣지 라벨/확률, 예 protein-function 연관), Graph-level(그래프 1라벨, 예 분자 독성).
- **의생명 그래프 예**: Gene Network(BRCA1·TP53·EGFR…), Metabolic Pathway, Molecules(원자=노드). 소셜·subway·citation도.
- **시험 포인트 ⭐**: "왜 GNN이 필요한가"(가변 이웃·순서 불변). 3주차 출제. $G=(V,E)$, $A$, $\mathcal{N}(v)$ 정의.

### C13. GNN 노드 업데이트 — Message Passing (AGGREGATE + UPDATE)   ⭐⭐⭐高 (3주, 식 문제 확정)
- **출처**: 3.graphNN.pdf p.7–11 (Basic GNN: message passing / updating nodes)
- **선행**: C12
- **한 줄 직관**: "각 노드는 이웃으로부터 메시지를 모아(AGGREGATE) 자기 표현을 갱신(UPDATE)한다. 층을 쌓을수록 더 먼 이웃 정보가 도달."
- **🧮 수식 (일반형) ⭐⭐⭐**:
  - **초기화**: $h_v^{(0)} = x_v$ ($h_v^{(k)}$ = 층 $k$에서 노드 $v$의 은닉표현).
  - **Message Passing (한 층)**:
    $$m_v^{(k)} = \text{AGGREGATE}^{(k)}\big(\{h_u^{(k)} : u\in\mathcal{N}(v)\}\big)$$
    $$h_v^{(k+1)} = \text{UPDATE}^{(k)}\big(h_v^{(k)},\, m_v^{(k)}\big)$$
  - **구체 (mean aggregate + linear update)**:
    $$m_v^{(k)} = \frac{1}{|\mathcal{N}(v)|}\sum_{u\in\mathcal{N}(v)} h_u^{(k)}$$
    $$h_v^{(k+1)} = \sigma\big(W_{\text{neigh}}^{(k)} m_v^{(k)} + W_{\text{self}}^{(k)} h_v^{(k)} + b^{(k)}\big)$$
    또는 등가로 $h_v^{(k+1)} = \sigma\big(W_{\text{self}} h_v^{(k)} + W_{\text{neigh}}\cdot \text{MEAN}\{h_u^{(k)}:u\in\mathcal{N}(v)\}\big)$.
  - **기호 의미 ⭐**: $m_v^{(k)}$=이웃에서 모은 메시지, $\mathcal{N}(v)$=이웃집합, $|\mathcal{N}(v)|$=이웃 수(정규화), $W_{\text{neigh}}$=**모든 이웃에 공유되는** 이웃 가중치행렬(이웃 순서 없으므로 공유), $W_{\text{self}}$=자기 가중치, $b$=bias, $\sigma$=비선형(ReLU 등).
- **✏️ 단계별 풀이 예제 (슬라이드 p.11 — 시험 식 문제의 원형) ⭐⭐⭐**:
  - 그래프: $v_1{=}1.0, v_2{=}2.0, v_3{=}0.0, v_4{=}4.0$ (스칼라 feature, $X=[1,2,0,4]^T\in\mathbb{R}^{4\times1}$). 1-layer GNN(K=1), AGGREGATE=mean, **self-included 이웃** $\tilde{\mathcal{N}}(v_1)=\{v_1,v_2,v_3\}$. $W^{(0)}=[a^{(0)}\ b^{(0)}]=[0.6\ 0.4]$ (UPDATE: $h_v^{(1)}=W^{(0)}[h_v^{(0)};m_v^{(0)}]=a\,h_v^{(0)}+b\,m_v^{(0)}$).
  - **target $v_1$**: aggregate $m_{v_1}^{(0)}=\frac{1}{3}(h_{v_1}^{(0)}+h_{v_2}^{(0)}+h_{v_3}^{(0)})=\frac{1+2+0}{3}=1.000$. update $h_{v_1}^{(1)}=0.6\cdot1.0+0.4\cdot1.000=\mathbf{1.000}$.
  - 나머지: $\tilde{\mathcal{N}}(v_2)=\{v_1,v_2,v_4\}$: $m=\frac{1+2+4}{3}=2.333$, $h_{v_2}^{(1)}=0.6\cdot2.0+0.4\cdot2.333=\mathbf{2.133}$. $\tilde{\mathcal{N}}(v_3)=\{v_1,v_3,v_4\}$: $m=\frac{1+0+4}{3}=1.667$, $h=0.6\cdot0+0.4\cdot1.667=\mathbf{0.667}$. $\tilde{\mathcal{N}}(v_4)=\{v_2,v_3,v_4\}$: $m=\frac{2+0+4}{3}=2.000$, $h=0.6\cdot4+0.4\cdot2.0=\mathbf{3.200}$.
  - 결과 $H^{(1)}=[1.000, 2.133, 0.667, 3.200]^T\in\mathbb{R}^{4\times1}$.
- **K-layer 효과**: 1층=1-hop 이웃, 2층=2-hop, K층=K-hop receptive field(경로 있으면). 너무 깊으면 **oversmoothing**(임베딩이 너무 유사)·**oversquashing**(먼 정보가 한 벡터에 압축).
- **시험 포인트 ⭐⭐⭐**: **이 수식과 worked example이 "GNN 식 문제 1개"의 본체**. AGGREGATE/UPDATE 분리, mean 정규화 $1/|\mathcal{N}(v)|$, $W_{\text{neigh}}$ 공유 이유(순서 없음), 손계산(위 예제 숫자 그대로 나올 수 있음).

### C14. CNN vs GNN (공통점·차이)   ⭐高 (3주)
- **출처**: 3.graphNN.pdf p.12–15
- **선행**: C5, C13
- **한 줄 직관**: "둘 다 '이웃 정보로 한 위치를 갱신'하지만, CNN은 고정 격자·위치별 가중치, GNN은 가변 이웃·공유 가중치."
- **비교 ⭐ (슬라이드 식)**:
  - **CNN local aggregation**: $h_v^{(l+1)}=\sigma\big(\sum_{u\in\mathcal{N}(v)} W_l^{\rho(u,v)} h_u^{(l)} + B_l h_v^{(l)}\big)$ — $\rho(u,v)$=상대위치(예 3×3→9개), **위치마다 다른 $W$**. 파라미터: 9위치×($C_{out}{\times}C_{in}$=4×3=12)+bias 4 = **112**.
  - **GNN aggregation**: $h_v^{(l+1)}=\sigma\big(W_l \sum_{u\in\mathcal{N}(v)}\frac{h_u^{(l)}}{|\mathcal{N}(v)|}+B_l h_v^{(l)}\big)$ — **모든 이웃에 같은 $W_l$ 공유**, 정규화 $1/|\mathcal{N}(v)|$. 파라미터: $W_{neigh}$(12)+$W_{self}$(12)+bias(4) = **28**.
- **핵심 메시지**: CNN = 고정 이웃 크기·순서를 가진 GNN의 특수경우. CNN은 위치별 가중치 → 커널↑면 파라미터↑, **permutation-equivariant 아님**(픽셀 순서 바꾸면 출력 달라짐). GNN = 공유 가중치 → 이웃 크기 무관·파라미터 적음·순서 불변.
- **시험 포인트 ⭐**: "CNN과 GNN의 공통(local aggregation)·차이(위치별 vs 공유 가중치, 순서 의존성)". 파라미터 수 계산(112 vs 28).

### C15. GCN & GraphSAGE   ⭐中 (3주)
- **출처**: 3.graphNN.pdf p.16–17
- **선행**: C13
- **한 줄 직관**: "GCN=차수 정규화로 이웃 집계, GraphSAGE=대형 그래프용으로 이웃을 샘플링하고 다양한 aggregator."
- **🧮 수식 ⭐**:
  - **GCN**: $H^{(k+1)} = \sigma\big(\tilde{D}^{-1/2}\tilde{A}\tilde{D}^{-1/2} H^{(k)} W^{(k)}\big)$ — $\tilde{A}=A+I$(self-loop 추가), $\tilde{D}$=$\tilde{A}$의 차수 대각행렬, $\tilde{D}^{-1/2}\tilde{A}\tilde{D}^{-1/2}$=**대칭 차수 정규화**(degree normalization), $H^{(k)}$=노드표현 행렬, $W^{(k)}$=가중치.
  - **GraphSAGE**: aggregate $m_v^{(k)}=\text{AGGREGATE}^{(k)}(\{h_u^{(k)}:u\in\mathcal{N}(v)\})$, update $h_v^{(k+1)}=\sigma\big(W^{(k)}[h_v^{(k)}\,\|\,m_v^{(k)}]\big)$ ($\|$=concat). 대형 그래프 대상, **이웃을 고정 수만큼 샘플링**, aggregator=mean/pooling/LSTM.
- **시험 포인트 ⭐**: GCN의 $\tilde{D}^{-1/2}\tilde{A}\tilde{D}^{-1/2}$ 의미(차수 정규화·self-loop), GraphSAGE의 concat update와 샘플링. C13 일반형의 구체 변주.

### C16. Graph Attention Network (GAT)   ⭐⭐⭐高 (3주, 핵심 논문 + 식 문제)
- **출처**: 3.graphNN.pdf p.22–25 / (Paper) Graph Attention Networks (ICLR 2018) 전체
- **선행**: C13, C15
- **한 줄 직관**: "모든 이웃이 똑같이 기여할 필요 없다 → message passing은 유지하되, **학습된 attention 가중치**로 이웃을 차등 집계."
- **🧮 수식 (슬라이드 3-step ⭐⭐⭐ = 시험 식 후보)**:
  - **Step 1 (attention score)**: $e_{vu}^{(k)} = a\big(W^{(k)} h_v^{(k)},\, W^{(k)} h_u^{(k)}\big)$ — 노드 $u$의 feature가 $v$에 얼마나 중요한지.
  - **Step 2 (normalized coeff)**: $\alpha_{vu}^{(k)} = \dfrac{\exp(e_{vu}^{(k)})}{\sum_{r\in\mathcal{N}(v)}\exp(e_{vr}^{(k)})}$ — softmax로 이웃에 대해 정규화.
  - **Step 3 (node update)**: $h_v^{(k+1)} = \sigma\big(\sum_{u\in\mathcal{N}(v)} \alpha_{vu}^{(k)}\, W^{(k)} h_u^{(k)}\big)$ — attention 가중 합.
- **🧮 논문판 수식 (GAT paper eq 1–4) ⭐**:
  - eq(1) $e_{ij}=a(\mathbf{W}\vec{h}_i,\mathbf{W}\vec{h}_j)$. eq(2) $\alpha_{ij}=\text{softmax}_j(e_{ij})=\frac{\exp(e_{ij})}{\sum_{k\in\mathcal{N}_i}\exp(e_{ik})}$.
  - eq(3) 완전전개: $\alpha_{ij}=\dfrac{\exp\big(\text{LeakyReLU}(\vec{a}^T[\mathbf{W}\vec{h}_i\,\|\,\mathbf{W}\vec{h}_j])\big)}{\sum_{k\in\mathcal{N}_i}\exp\big(\text{LeakyReLU}(\vec{a}^T[\mathbf{W}\vec{h}_i\,\|\,\mathbf{W}\vec{h}_k])\big)}$ — attention $a$=single-layer FFN(가중치 $\vec{a}\in\mathbb{R}^{2F'}$) + LeakyReLU(slope 0.2), $\|$=concat.
  - eq(4) $\vec{h}_i'=\sigma\big(\sum_{j\in\mathcal{N}_i}\alpha_{ij}\mathbf{W}\vec{h}_j\big)$.
- **Multi-head (CAT, slide p.25 / paper eq 5–6) ⭐**: $K$개 독립 attention head 병렬 → **concat**: $\vec{h}_i'=\big\Vert_{k=1}^{K}\sigma\big(\sum_{j\in\mathcal{N}_i}\alpha_{ij}^k\mathbf{W}^k\vec{h}_j\big)$ (출력 $KF'$차원). **최종 예측층은 average**: $\vec{h}_i'=\sigma\big(\frac{1}{K}\sum_{k=1}^K\sum_{j\in\mathcal{N}_i}\alpha_{ij}^k\mathbf{W}^k\vec{h}_j\big)$.
- **GCN vs GAT**: GCN=차수 기반 **고정** 정규화, GAT=이웃마다 **학습된** attention 가중치(엣지 두께=중요도).
- **masked attention**: $e_{ij}$를 $j\in\mathcal{N}_i$(1-hop 이웃, self 포함)에 대해서만 계산 → 그래프 구조 주입. eigendecomposition·전체 그래프 구조 사전지식 불필요 → inductive 가능.
- **시험 포인트 ⭐⭐⭐**: GAT 3-step(score→softmax→가중합) 식과 기호. "왜 attention인가"(이웃 차등). multi-head concat/avg. GCN과 대조. **이것이 "GNN 식 문제"의 강력 후보**(논문이 핵심).

### C17. 【논문 breakthrough】 GAT 평가 결과   ⭐中 (3주, 논문 해석)
- **출처**: (Paper) Graph Attention Networks p.6–8 (Table 1–3)
- **선행**: C16
- **한 줄 직관**: "GAT가 spectral 방법의 한계(그래프 구조 고정 의존)를 벗고, transductive·inductive 모두에서 SOTA."
- **표 인사이트 ⭐**:
  - 데이터(Table1): Cora(2708노드/7클래스/transductive), Citeseer(3327/6/transductive), Pubmed(19717/3/transductive), **PPI(56944노드/24그래프/121멀티라벨/inductive, 테스트 그래프 unseen)**.
  - **Transductive(Table2 정확도)**: GAT **Cora 83.0%**(GCN 81.5 대비 +1.5), Citeseer **72.5%**(+1.6 vs GCN 70.3), Pubmed **79.0%**(=GCN).
  - **Inductive(Table3 micro-F1)**: GAT **0.973**(GraphSAGE 최고 0.768 대비 +20.5%), Const-GAT(attention 상수=GCN류) 0.934 대비 +3.9% → **이웃 차등 가중의 효과 직접 입증**.
  - GAT transductive 하이퍼: 2-layer, 1층 K=8 head×F'=8, ELU, dropout 0.6, L2 0.0005; inductive 3-layer.
- **시험 포인트 ⭐**: "GAT가 GCN/GraphSAGE보다 우월한 이유 = 이웃마다 다른 importance 학습"(Const-GAT 대비 +3.9%가 직접 근거). transductive vs inductive 구분.

### C18. GNN 활용: 노드/엣지/그래프 분류 + 손실   ⭐中 (3주)
- **출처**: 3.graphNN.pdf p.26–34
- **선행**: C13
- **한 줄 직관**: "GNN encoder로 노드 임베딩 Z를 얻고, task decoder(노드/엣지/그래프)로 예측."
- **인코더-디코더**: $(G,X)\xrightarrow{\text{GNN enc}} Z \xrightarrow{\text{task dec}} \hat{Y}$. $z_v=h_v^{(K)}$. 노드 디코더→$\hat{y}_v$, 엣지 디코더(inner-product $s(u,v)=z_u^T z_v$ 또는 MLP, sigmoid)→$\hat{y}_{uv}$, 그래프 디코더(READOUT pooling→$h_G$)→$\hat{y}_G$.
- **세팅**: Transductive(같은 그래프 train/test, 라벨 노드만 loss) vs Inductive(unseen 그래프 일반화).
- **🧮 손실 ⭐**: Node: $\mathcal{L}_{\text{node}}=-\sum_{v\in V_{\text{train}}}\sum_c \mathbf{1}(y_v=c)\log\hat{y}_{v,c}$ ($\hat{y}_v=\text{softmax}(W_{\text{out}}h_v^{(K)}+b_{\text{out}})$). Graph-cls: BCE $-\sum_i[y_i\log\hat{y}_{G_i}+(1-y_i)\log(1-\hat{y}_{G_i})]$. Graph-reg: MSE $\frac{1}{N}\sum(y_i-\hat{y}_{G_i})^2$.
- **응용**: 소셜 노드분류(homophily; GAT가 informative 이웃에 큰 가중치), link prediction(inner-product/MLP decoder), 분자 그래프분류(원자=노드, 결합=엣지, READOUT→독성/용해도), **GNN 단백질 기능예측**(residue graph + ESM2 embedding + GVP + graph attention pooling → GO 다중라벨, Bioinformatics 2025).
- **시험 포인트 ⭐**: 3예측레벨별 디코더·손실. transductive/inductive. GNN의 의생명 응용(protein function, molecule).

### C19. 【범위 밖(참고)】 실습 1·2 (ProtBERT 단백질 분류)   ⭐低 (시험 제외)
- **출처**: (실습) 1번/2번 실습 가이드.pdf + toy_train.csv·toy_valid.csv
- **선행**: C7, C11
- **한 줄 직관**: "사전학습 단백질 LM(ProtBERT) + 분류기로 막관통 이진분류 — 딥러닝 4단계(데이터→모델→학습→평가)를 코드로."
- **요지(참고)**: 입력=아미노산 서열, 출력=0(non-membrane)/1(membrane). mean pooling → Linear 분류기, CrossEntropy + Adam, BP로 학습. 학습곡선에서 valid loss 재상승=과적합.
- ⚠️ **범위 밖**: 시험범위가 "실습 제외"로 명시 → 설계·HTML에서 비중 최소(참고 라벨). 토큰화·BP 개념은 C8·C4에서 이미 본문으로 다룸.

---

## 3. 개념 의존성 맵

- **1주 (Intro+DeepLearning)**: C1(central dogma) → C2(NGS 변이탐지) ; C3(표현학습) → **C4(BP)** → C5(CNN/FC 파라미터) → C6(DeepVariant/NeuSomatic CNN)
- **2주 (gLM+DNABERT)**: C7(gLM 패러다임) → {C8(single-nt), C9(k-mer/DNABERT), C10(BPE)} ; C7 → C11(MLM/CLM) → (DNABERT = C9 k-mer × C11 MLM)
- **3주 (graphNN+GAT)**: C12(그래프/GNN 동기) → **C13(노드업데이트 message passing)** → {C14(CNN vs GNN), C15(GCN/GraphSAGE), **C16(GAT)** → C17(GAT 결과)} → C18(예측레벨/손실)
- **주차 간 연결**: C4(BP)는 모든 학습의 공통(C6·C11·C13 다 BP로 학습). C5(CNN 가중치공유) ↔ C14(GNN 가중치공유 대조). C11(self-attention) → C16(graph attention).
- **시험 우선 경로 ⭐**: **C4(BP) · C8–C9(토큰화) · C13(노드업데이트 식) · C16(GAT 식)** 이 4개가 6문제의 핵심. C13/C16에서 식 문제 1개 확정.

---

## 4. 누락·확인 필요 목록

- ⚠️ **Notion 보강 전무**: 과목 Notion 페이지 토큰 만료로 fetch 실패 → 교수 구두 강조점·기출 원문 미확보. 시험 가이드는 사용자 제공 시험범위로 대체(충분히 구체적).
- ⚠️ **기출 문항 원문 없음**: "주차별 2문제·GNN 식 1개·입력출력 서술형" 형태만 명시, 구체 문제·정답은 자료에 없음 → 예상문제는 형태에 맞춰 설계하되 정답은 PDF 근거로만.
- ⚠️ **실습 최종 수치**: 범위 밖이며 구체 AUROC/F1 미확보(참고만).
- ✅ **확보 완비**: BP 수식(Deep Learning Fig.1c/d), genomic 토큰화 3종+DNABERT vocab=4^k+5·입력형상, GNN 노드업데이트 일반식+worked example(슬라이드 p.11 숫자), GCN/GraphSAGE/GAT 식, GAT 논문 eq1–6·multi-head·Table 결과. 식 문제 대비 완전.

---

## 5. 핵심 논문 카드 (심층 — 포스터 전개용, 종전 2~3배 볼륨)

> 각 논문 PDF를 끝까지 정독해 재구성. terminology 8~12개 + method 다단계(입력→처리→출력 형상) + figures/results 정량 + breakthrough + 시험연결.

### PAPER-A. Deep Learning (LeCun, Bengio, Hinton — Nature 521:436–444, 2015)  [1주 · BP의 원천]

- **🏷️ 한 줄**: 딥러닝 = 여러 비선형 층이 raw 데이터에서 표현(feature)을 **스스로** 배우고, 그 가중치를 **역전파(backpropagation)**로 갱신하는 표현학습 — 비전·음성·NLP·genomics를 석권.

- **📚 Terminology (10)**:
  1. **Representation learning** — raw 입력에서 분류/검출에 필요한 표현(feature)을 사람이 설계하지 않고 자동으로 발견하는 방법군. 딥러닝의 정의 그 자체.
  2. **Deep learning** — 단순하지만 비선형인 모듈을 여러 개 합성(5~20층)해 표현을 점점 추상화하는 표현학습. "사람이 층을 설계하지 않고 데이터로 학습"이 핵심.
  3. **Backpropagation (BP)** — 다층 모듈의 가중치에 대한 목적함수의 gradient를 chain rule로 **출력→입력** 방향으로 효율 계산하는 절차. "마법이 아니라 chain rule의 실용 적용".
  4. **Gradient / gradient vector** — 각 가중치를 아주 조금 키울 때 에러가 얼마나 변하는지의 벡터. 학습은 이 반대(가장 가파른 하강) 방향으로 이동.
  5. **Objective / cost function** — 출력 점수와 정답(desired) 사이의 에러(거리). 가중치 공간의 "언덕 풍경(hilly landscape)"으로 비유, 최저점을 찾음.
  6. **SGD (Stochastic Gradient Descent)** — 소수 예제(미니배치)로 평균 gradient를 noisy하게 추정해 가중치를 반복 갱신. 정교한 최적화보다 놀랄 만큼 빠르게 좋은 해를 찾음.
  7. **ReLU** — \(f(z)=\max(0,z)\). 깊은 망에서 sigmoid/tanh보다 빠르게 학습(비지도 사전학습 없이도), gradient 소실 완화.
  8. **CNN (ConvNet)** — 4대 아이디어(국소 연결·가중치 공유(filter bank)·풀링·다층)로 격자형 신호(이미지)를 처리. 합성곱=local motif 탐지, 풀링=위치 불변성·차원 축소.
  9. **Distributed representation** — 한 개념을 여러 feature의 조합(활성 벡터)으로 표현 → 학습한 조합을 넘어 일반화(지수적 이득). word vector가 대표.
  10. **RNN / LSTM** — 시퀀스를 한 원소씩 처리하며 과거를 state vector에 누적. BPTT로 학습하나 gradient 폭발/소실 → LSTM(게이트 메모리 셀)으로 완화.

- **💡 Core Concept / Method (입력→처리→출력, 형상까지)**:
  - **지도학습 골격**: 입력 이미지/서열 → 망 → **점수 벡터(클래스 수만큼)** 출력 → 목적함수 \(E\)로 정답과 비교 → 가중치(수억 knobs)를 gradient 반대 방향으로 조정. 반복.
  - **Forward (Fig.1c)**: 각 층 총입력 \(z_l=\sum_k w_{kl}\,y_k\)(아래 층 출력의 가중합), 출력 \(y_l=f(z_l)\). \(f\)=ReLU/sigmoid/tanh. 2은닉층 망: 입력→H1(\(z_j=\sum_i w_{ij}x_i,\ y_j=f(z_j)\))→H2(\(z_k=\sum_j w_{jk}y_j\))→출력(\(z_l=\sum_k w_{kl}y_k\)).
  - **Chain rule (Fig.1b)**: \(\Delta z=\frac{\partial z}{\partial y}\Delta y\), \(\Delta y=\frac{\partial y}{\partial x}\Delta x\) → \(\frac{\partial z}{\partial x}=\frac{\partial z}{\partial y}\frac{\partial y}{\partial x}\).
  - **Backward (Fig.1d)**: 출력층 \(\frac{\partial E}{\partial y_l}=y_l-t_l\)(cost \(\tfrac12\sum_l(y_l-t_l)^2\)일 때) → \(\frac{\partial E}{\partial z_l}=\frac{\partial E}{\partial y_l}f'(z_l)\). 은닉층 \(\frac{\partial E}{\partial y_k}=\sum_l w_{kl}\frac{\partial E}{\partial z_l}\), \(\frac{\partial E}{\partial z_k}=\frac{\partial E}{\partial y_k}f'(z_k)\)를 입력층까지. 가중치 gradient \(\frac{\partial E}{\partial w_{jk}}=y_j\frac{\partial E}{\partial z_k}\)(아래 출력 × 위 에러신호). 갱신 \(w\leftarrow w-\eta\frac{\partial E}{\partial w}\).
  - **CNN 데이터 흐름(Fig.2)**: 입력 (H,W,3=RGB) → [Conv+ReLU → Max pool] 반복 → 특징맵이 bottom-up으로 edge→motif→part→object 위계. 풀링이 작은 이동·왜곡에 불변성 부여.
  - **표현 위계(왜 깊이가 이득)**: 자연 신호는 compositional hierarchy(낮은 층 조합 = 높은 층). 깊이가 깊을수록 같은 함수를 지수적으로 적은 유닛으로 표현.
- **📊 Figures / Results (정량)**:
  - **ImageNet 2012(ref1)**: 깊은 ConvNet이 기존 최고 대비 **에러를 거의 절반**으로 → 컴퓨터비전 혁명 촉발(논문이 직접 인용).
  - **Fig.2 Samoyed 예**: 출력층이 Samoyed 16점 / Papillon 5.7 / Pomeranian 2.7 / Arctic fox 1.0… 로 미세 구분(selectivity)하면서 포즈·배경엔 불변(invariance).
  - **saddle point 통찰**: 큰 망에서 poor local minima는 드묾 — 대부분 saddle point라 SGD가 잘 탈출(이론·실험 근거).
  - **ConvNet 규모**: 10~20 ReLU 층, 수억 가중치, 수십억 연결. GPU·dropout·데이터 증강으로 학습.
- **🚀 Breakthrough (한계→돌파→중요성→한계)**:
  - **이전 한계**: 전통 ML은 사람이 feature extractor를 손설계(도메인 전문성·노동). shallow 분류기는 selectivity-invariance 딜레마를 못 풀음.
  - **돌파**: **범용 학습 절차(BP+SGD)**로 feature를 자동 학습 → 손설계 제거. 깊은 표현 위계로 복잡 함수 표현.
  - **중요성**: 음성·비전 SOTA, **비코딩 DNA 변이→유전자 발현·질병 예측(genomics)**까지 적용(논문이 명시) → 본 과목 모든 모델(DNABERT·GNN·CNN 변이탐지)의 **공통 학습 엔진**.
  - **남은 한계/전망**: 라벨 의존(지도학습) → 비지도/표현학습이 장기 과제. RNN의 long-term 의존 어려움(→LSTM). reasoning과의 결합 미해결.
- **🔗 강의·시험 연결**: C3(표현학습)·**C4(BP — 1주 출제 확정급)**·C5(CNN). 예상출제: "forward→loss→backward→update를 식으로 서술", "출력층 ∂E/∂z 유도", "왜 ReLU/가중치 공유인가".

### PAPER-B. DNABERT (Ji, Zhou, Liu, Davuluri — Bioinformatics 37(15):2112–2120, 2021)  [2주 · 토큰화·MLM]

- **🏷️ 한 줄**: BERT(양방향 Transformer)를 DNA에 — **k-mer 토큰화 + 양방향 MLM 사전학습**으로 비코딩 DNA의 "문법"을 한 모델이 배워, 적은 라벨로 promoter·TFBS·splice를 동시에 SOTA.

- **📚 Terminology (12)**:
  1. **cis-regulatory element (CRE)** — 유전자 발현을 조절하는 비코딩 DNA 구간(promoter·enhancer 등). 문맥에 따라 기능이 변함(polysemy).
  2. **Polysemy / distant semantic relationship** — 같은 서열이 문맥마다 다른 기능(polysemy), 멀리 떨어진 구간이 협력(distant). 자연어의 다의성·장거리 의존과 유사 → 언어모델이 적합한 근거.
  3. **k-mer tokenization** — 연속 k개 염기를 한 토큰으로(주변 문맥을 토큰에 주입). 예 ATGGGCT의 3-mer={ATG,TGG,GGG,GGC,GCT}, 4-mer/5-mer/6-mer도. DNABERT-3/4/5/6 학습.
  4. **Vocabulary 크기 \(4^k+5\)** — k-mer 종류 \(4^k\) + 특수토큰 5개. k=6 → \(4^6+5=4101\).
  5. **[CLS] / [SEP] / [PAD] / [UNK] / [MASK]** — 분류용(전체 문장 의미)/분리/패딩/미지/마스킹 특수토큰.
  6. **MLM (Masked Language Model)** — 입력 토큰의 ~15%(DNABERT는 **연속 k-mer span**)를 [MASK] → 나머지로 예측하는 self-supervised 사전학습. 양방향 문맥.
  7. **Bidirectional self-attention** — 좌·우 문맥을 동시에 보는 인코더 attention(이해 과제에 강함). BERT 계열의 핵심.
  8. **Multi-head self-attention** — \(\text{head}_i=\text{softmax}(\frac{MW_i^Q (MW_i^K)^T}{\sqrt{d_k}})MW_i^V\)를 h개 병렬 후 concat·\(W^O\). 토큰 행렬 M 위에서 전역 문맥 통합.
  9. **Fine-tuning** — 사전학습 가중치 + 태스크 head(문장수준/토큰수준 분류기)로 적은 라벨 학습. 마스킹 없이 통째 입력.
  10. **TFBS / promoter / splice site** — 전사인자 결합부위 / 전사 시작 부근 조절구간 / 엑손-인트론 경계. DNABERT의 3대 다운스트림 과제.
  11. **DNABERT-viz** — attention 맵으로 어느 염기 구간이 중요한지 시각화 → 모티프·기능변이 후보를 비지도로 발견(해석가능성).
  12. **DNABERT-XL** — 512 초과 긴 서열을 조각내 표현을 concat(예 prom-300의 1001bp 스캔). BERT의 512 한계 우회.

- **💡 Core Concept / Method (입력→처리→출력, 형상까지)**:
  - **입력 구성**: DNA 서열(길이 ≤510) → k-mer 토큰화 → **[CLS] + k-mer 토큰들 + [SEP]**, max length **512**(부족분 [PAD]). 각 토큰 = Token Embedding + Positional Embedding → 임베딩 행렬 **M (512, 768)**.
  - **백본**: BERT-base와 동일 — **12 Transformer 인코더 층, hidden 768, attention head 12**. eq(1) MultiHead, eq(2) head의 scaled dot-product attention. 전 층 동일 설정.
  - **사전학습(MLM)**: 각 서열에서 **15% k-mer를 연속 span으로 마스킹** → 나머지로 [MASK] 예측. 손실 **cross-entropy \(L=-\sum_i y_i'\log(y_i)\)**(\(y_i'\)=정답 one-hot, \(y_i\)=예측확률). self-supervised(라벨 불필요).
  - **학습 디테일(논문 §2.2.2)**: 사람 유전체에서 non-overlap split + random sampling으로 길이 5~510 서열 생성. 120k step 사전학습(앞 100k는 mask 15%, 뒤 20k는 20%), batch 2000, LR warm-up 0→4e-4 후 0까지 선형감소, mixed precision, 8×NVIDIA 2080Ti, 약 25일.
  - **파인튜닝/출력**: [CLS]의 마지막 은닉 → **sentence-level 분류**(promoter 여부), 토큰별 은닉 → **token-level 분류**(splice·변이 위치). AdamW + dropout.
- **📊 Figures / Results (정량)**:
  - **Fig.1**: (a) RNN/CNN/Transformer 문맥 처리 비교 — Transformer만 self-attention으로 전역 문맥. (b) DNABERT 입력→임베딩(Token+Positional)→12 블록→분류 head 도식. (d) attention이 알려진 결합부위 2곳에 정확히 집중.
  - **DNABERT-Prom(Fig.2)**: prom-300 TATA에서 DeePromoter 대비 accuracy·MCC를 **+0.335·+0.554** 초과. core promoter도 CNN·CNN+LSTM·CNN+GRU 모두 상회.
  - **DNABERT-TF(Fig.3)**: 690 ENCODE TF에서 mean·median accuracy·F1 **>0.9 (0.918, 0.919)**, 2위(DeepSEA) 대비 유의(P=4.5e-100). FP·FN 최소.
  - **DNABERT-Splice**: adversarial 데이터 포함 시 baseline 급락에도 accuracy **0.923**, F1 0.919, MCC 0.871로 최고.
  - **사전학습 효과(Fig.6d/e)**: pre-trained가 random-init보다 **훨씬 낮은 loss로 수렴**, random-init은 local minima에 갇힘. 사람 유전체로 사전학습한 모델이 **78개 mouse ENCODE**로 일반화(전이) — 단백질 코딩 85% 직교·비코딩 50% 유사인데도 성공.
  - **변이 분석**: dbSNP 7억 변이 중 high-attention 영역의 변이가 ClinVar/GWAS와 유의 연관 → 기능변이 후보 발견.
- **🚀 Breakthrough (한계→돌파→중요성→한계)**:
  - **이전 한계**: CNN은 필터 크기 탓 장거리 의미 의존 포착 불가, RNN/LSTM은 순차처리·장입력 병목+gradient 문제. 대부분 대량 라벨 필요(데이터 희소 시 무력).
  - **돌파**: **'one-model-does-it-all'** — 한 번 사전학습(self-supervised)으로 promoter·TFBS·splice를 동시에 커버. 전역 self-attention으로 장거리·문맥. 적은 라벨에서 강함. **top-down**(먼저 일반 DNA 이해→과제 적용) 접근.
  - **중요성**: 비코딩 DNA 해독·기능변이 식별·교차종 전이. 2주 토큰화·MLM의 실제 구현체.
  - **남은 한계**: 512 토큰 한계(→XL), k 고정(→DNABERT-2의 BPE), 사전학습 25일·8 GPU의 자원 비용.
- **🔗 강의·시험 연결**: **C8·C9(토큰화 — 2주 출제), C11(MLM)**. 예상출제: "DNA→k-mer→토큰ID→임베딩 (512,768) 형상", "vocab=4^k+5 계산", "MLM 손실식·self-supervised", "MLM↔CLM", "사전학습이 random-init보다 우월한 근거(Fig.6)".

### PAPER-C. Graph Attention Networks (Veličković et al. — ICLR 2018)  [3주 · GNN 식·attention]

- **🏷️ 한 줄**: GNN의 message passing은 유지하되 이웃 집계를 **masked self-attention(학습된 가중치)**으로 — 행렬연산·그래프 구조 사전지식 없이 이웃을 차등 가중해 transductive·inductive 모두 SOTA.

- **📚 Terminology (10)**:
  1. **Graph attentional layer** — GAT의 유일한 빌딩 블록. 입력 노드 feature 집합 \(\mathbf{h}=\{\vec h_1,...,\vec h_N\},\ \vec h_i\in\mathbb{R}^F\) → 출력 \(\mathbf{h}'=\{\vec h_1',...\},\ \vec h_i'\in\mathbb{R}^{F'}\).
  2. **Attention coefficient \(e_{ij}\)** — 노드 \(j\)의 feature가 \(i\)에 얼마나 중요한지의 비정규화 점수. \(e_{ij}=a(W\vec h_i,W\vec h_j)\).
  3. **Normalized coefficient \(\alpha_{ij}\)** — \(e_{ij}\)를 \(i\)의 이웃에 대해 softmax 정규화(\(\sum_{j\in\mathcal N_i}\alpha_{ij}=1\)). 노드 간 비교 가능하게.
  4. **Masked attention** — 모든 노드쌍이 아니라 **1-hop 이웃 \(\mathcal N_i\)(자기 포함)**에 대해서만 \(e_{ij}\) 계산 → 그래프 구조를 주입.
  5. **Shared weight matrix \(\mathbf{W}\in\mathbb{R}^{F'\times F}\)** — 모든 노드에 공유되는 선형변환(차원 \(F\to F'\)). 이웃 순서 없으므로 공유.
  6. **Attention mechanism \(a\)** — single-layer FFN(가중벡터 \(\vec a\in\mathbb{R}^{2F'}\)) + **LeakyReLU(slope 0.2)**. eq(3)이 완전전개형.
  7. **Multi-head attention** — K개 독립 attention을 병렬. **중간층=concat**(출력 \(KF'\)), **최종(예측)층=average** 후 비선형.
  8. **Transductive vs Inductive** — 같은 그래프에서 학습·평가(라벨 노드만 loss) vs **학습에 전혀 없던 그래프**로 일반화(PPI 테스트 그래프 unseen).
  9. **Spectral method (대비군)** — 그래프 Laplacian eigendecomposition 기반. 필터가 eigenbasis(그래프 구조)에 의존 → **다른 구조 그래프에 전이 불가**. GAT가 극복하는 한계.
  10. **GCN / Const-GAT (baseline)** — GCN=고정 차수 정규화. Const-GAT=GAT와 동일 구조지만 attention을 상수(=동등 가중)로 → attention 효과의 순수 측정용.

- **💡 Core Concept / Method (3-step, 형상까지)**:
  - **입력**: 노드 feature \(\vec h_i\in\mathbb{R}^F\), 그래프 구조(이웃 \(\mathcal N_i\)). **Step1 score** eq(1) \(e_{ij}=a(\mathbf W\vec h_i,\mathbf W\vec h_j)\). **Step2 normalize** eq(2) \(\alpha_{ij}=\text{softmax}_j(e_{ij})=\frac{\exp(e_{ij})}{\sum_{k\in\mathcal N_i}\exp(e_{ik})}\). **완전전개** eq(3) \(\alpha_{ij}=\frac{\exp(\text{LeakyReLU}(\vec a^T[\mathbf W\vec h_i\Vert\mathbf W\vec h_j]))}{\sum_{k\in\mathcal N_i}\exp(\text{LeakyReLU}(\vec a^T[\mathbf W\vec h_i\Vert\mathbf W\vec h_k]))}\). **Step3 update** eq(4) \(\vec h_i'=\sigma(\sum_{j\in\mathcal N_i}\alpha_{ij}\mathbf W\vec h_j)\).
  - **Multi-head**: eq(5) concat \(\vec h_i'=\big\Vert_{k=1}^K\sigma(\sum_{j\in\mathcal N_i}\alpha_{ij}^k\mathbf W^k\vec h_j)\) (출력 \(KF'\)); eq(6) average(최종층) \(\vec h_i'=\sigma(\frac1K\sum_{k=1}^K\sum_{j\in\mathcal N_i}\alpha_{ij}^k\mathbf W^k\vec h_j)\). Fig.1: 왼쪽=attention 메커니즘, 오른쪽=K=3 head(색별)로 node1 갱신.
  - **복잡도**: 한 head \(O(|V|FF'+|E|F')\) — GCN과 동급. eigendecomposition·역행렬 등 비싼 연산 불필요, 엣지·노드 병렬화 가능.
  - **하이퍼파라미터(§3.3)**: transductive 2-layer(1층 **K=8, F'=8**, ELU; 2층 분류 single head + softmax, dropout 0.6, L2 0.0005; Pubmed은 출력 K=8·L2 0.001). inductive **3-layer**(앞 2층 K=4·F'=256=1024, 최종 K=6·121 average + sigmoid, skip connection, batch 2 graphs). Glorot init, Adam, early stop patience 100.
- **📊 Figures / Results (정량)**:
  - **데이터(Table1)**: Cora(2708노드/5429엣지/7클래스/1433 feat/transductive), Citeseer(3327/4732/6/3703), Pubmed(19717/44338/3/500), **PPI(56944노드/24그래프/121 멀티라벨/50 feat/inductive, 테스트 2그래프 unseen)**.
  - **Transductive(Table2 정확도)**: **Cora GAT 83.0±0.7%** vs GCN 81.5(+1.5); **Citeseer 72.5±0.7%** vs GCN 70.3(+1.6 → 논문은 +1.6 언급, 표상 +2.2); **Pubmed 79.0±0.3%** = GCN 79.0(동률). baseline: MLP Cora 55.1, DeepWalk 67.2, Chebyshev 81.2.
  - **Inductive(Table3 micro-F1)**: **GAT 0.973±0.002** vs GraphSAGE 최고 0.768(**+20.5%**) vs Const-GAT 0.934(**+3.9%**) vs MLP 0.422. → Const-GAT 대비 +3.9%가 "이웃 차등 가중"의 **순수 효과**.
  - **Fig.2 t-SNE**: Cora 1층 feature를 2D로 → 7클래스가 뚜렷이 군집(판별력). 엣지 두께=집계 attention(\(\sum_k\alpha_{ij}^k+\alpha_{ji}^k\)).
- **🚀 Breakthrough (한계→돌파→중요성→한계)**:
  - **이전 한계**: spectral 방법은 그래프 Laplacian eigenbasis에 의존 → 학습한 구조와 다른 그래프에 전이 불가. GraphSAGE는 이웃을 고정 수 **샘플링**(전체 이웃 못 봄)·LSTM aggregator는 노드에 순서 가정.
  - **돌파**: (1) 이웃 쌍별 병렬 → **효율**; (2) **이웃마다 다른 importance**를 암묵 지정(모델 용량↑·해석성); (3) 전역 구조 사전지식 불필요 → **inductive 직접 적용**(unseen 그래프). MoNet의 특수 사례로도 재정식화 가능.
  - **중요성**: GNN 식 문제의 핵심(GCN 고정정규화 → 학습 attention). 의생명 PPI(단백질 상호작용)에서 unseen 조직 일반화.
  - **남은 한계**: rank-2 sparse 행렬곱 한계로 배치 제약, receptive field가 depth로 상한, 엣지 병렬 시 이웃 중복 계산. edge feature·해석 심화는 future work.
- **🔗 강의·시험 연결**: **C16(GAT 식 — GNN 식 문제 강력 후보), C13(노드업데이트), C17(결과)**. 예상출제: "GAT 3-step 식·기호", "GCN의 \(1/|\mathcal N(v)|\)이 GAT에서 무엇으로(→\(\alpha\))", "주어진 score로 α 계산", "multi-head concat vs average", "Const-GAT +3.9%의 의미".
