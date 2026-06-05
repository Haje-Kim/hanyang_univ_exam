# 의료진단을 위한 AI (AAI0035, 김정희 교수) — 강의 자료 추출본
> 출처: 강의자료 PDF 3개(Week1 56p / Week2 60p / Week3 58p, 모두 "Blank" 버전) + Notion Day 노트 3개 | 추출일: 2026-06-02
> 비고: PDF는 빈칸(Blank) 슬라이드라 표·정의가 비어있는 곳이 있으나, **Week2 PDF가 Day2 Notion의 빈칸(ECG파형·EEG대역·영상모달리티)을 거의 전부 채워줌**. PDF↔Notion 상호 보강으로 작성.

---

## 0. 과목 개요

- **이 과목이 푸는 문제**: 바이오메디컬·헬스케어 문제(특히 **신경계 질환** 진단·재활)의 특성을 이해하고, 거기에 맞는 AI(ML/DL) 기법과 **센서·하드웨어·모바일 시스템**을 설계·적용하는 전체 흐름을 익힌다. 교수 연구실(HY-BPNE Lab, 바이오 신호처리·신경공학)의 핵심 키워드는 **Human-centered AI**: 진단(Diagnosis) → 치료(Treatment, therapeutic/assistive)로 이어지는 파이프라인.
- **큰 줄기(4주 강의 + 1주 시험)**:
  - Week1(Day1): 교과 소개 + **의료 진단과 AI의 관계**(AI/ML/DL 계층, 학습 패러다임, 분류/회귀/군집)
  - Week2(Day2): **현 진단 시스템의 문제** + **생체신호(ECG/EEG/EMG…) + 의료영상(X-ray/CT/MRI/US/PET…)** + 영상진단 딥러닝·멀티모달
  - Week3(Day3): **의료진단 AI용 하드웨어·센서·모바일 시스템(계측 시스템)** + **신경망(뉴런→퍼셉트론→MLP→역전파→CNN)**
  - Week4: 의료 AI의 한계·윤리·규제 (PDF 미제공)
  - Week5: **시험** + 최신 연구 사례
- **평가 방식 (Week1 p.9, GRADING POLICY)**: 출석 10 + 과제(Homework) 20 + **기말고사 70** = 100. 등급 A+ 95–100 … F <60.
- **시험 형식 (PDF + Notion 교차)**: PDF "시험: 객관식, 단답형, 서술형". **Notion Day1: "시험은 객관식이 70%~80%, 어렵지 않음!"** → ⭐ 객관식 비중이 절대적. 개념 정의·비교·매칭 위주로 대비.
- **과제(발표) (Week1 p.11~ / Day1·Day2 Notion)**: 의료 진단 적용 AI 최신 논문 발표(인당 4–5분, 3·4주차 후반). Literature Survey — Notion Day1은 "최소 10개 논문 비교"(PDF는 4–5개로 표기), 100편 정도 찾아 abstract만 읽고 20편 추리기, 핵심 `비교`표 + `lesson learned` 정리. 학교 IP+구글 스칼라, jcr.clarivate.com(임팩트 랭킹) 활용.

---

## 0-1. 선행 지식 키워드 (배경지식 — 본문에서 설명 없이 전제되는 것)

본문(`## 2. 개념 카탈로그`)이 **설명 없이 전제하는** 기초만 골랐다. 과목 내부 노드(C#)가 아니라 강의에 들어오기 전 갖춰야 할 토대. 카테고리: 수학 / CS기초 / 도메인(의료).

### PK1. [도메인:의료] 민감도(Sensitivity) / 특이도(Specificity)
- **쉬운 정의**: 민감도 = 실제 환자를 환자라고 맞히는 비율(놓치지 않는 능력). 특이도 = 정상인을 정상이라 맞히는 비율(헛경보 안 내는 능력).
- **왜 필요**: 진단 모델·생체신호 분류(C7·C8·C9·C11)의 성능을 말할 때 "이 검사 민감도 95%"처럼 당연히 전제된다. C9의 "치밀유방 민감도 낮음" 같은 서술도 이 개념 위에 선다.
- **출처/연결**: 의료 진단의 기본 평가 어휘 → C7·C8·C9·C11.

### PK2. [도메인:의료] 정밀도(Precision) / 재현율(Recall)
- **쉬운 정의**: 정밀도 = "양성이라 한 것 중 진짜 양성 비율". 재현율(=민감도) = "진짜 양성 중 잡아낸 비율". 둘은 트레이드오프 관계.
- **왜 필요**: 분류(C4)의 평가, 매칭형 시험에서 분류 평가지표를 물을 때 배경으로 깔린다.
- **출처/연결**: 분류 평가 어휘 → C4.

### PK3. [도메인:의료] 혼동행렬(Confusion Matrix) — TP/FP/TN/FN
- **쉬운 정의**: 예측(양성/음성)×실제(양성/음성) 2×2 표. 참양성(TP)·거짓양성(FP)·참음성(TN)·거짓음성(FN)으로 나뉜다. 민감도·특이도·정밀도가 모두 여기서 계산된다.
- **왜 필요**: 임계값(threshold)을 바꾸면 TP/FP가 어떻게 변하는지가 진단 모델 이해의 핵심. C4 분류·C11 영상진단 성능의 토대.
- **출처/연결**: 모든 분류·진단 평가의 뿌리 → C4·C11.

### PK4. [도메인:의료] ROC 곡선 / AUC
- **쉬운 정의**: 임계값을 0→1로 바꿔가며 (1−특이도, 민감도)를 찍은 곡선이 ROC. 곡선 아래 넓이(AUC)가 1에 가까울수록 좋은 분류기.
- **왜 필요**: 진단 모델의 표준 성능 그래프. 시험에서 "곡선/지표 매칭"을 물을 수 있고, C4·C11 평가의 배경.
- **출처/연결**: 진단 성능 시각화 → C4·C11.

### PK5. [수학] 확률 / 베이즈 기초
- **쉬운 정의**: 사건이 일어날 가능성(0~1). 베이즈 = 사전확률을 새 증거로 갱신해 사후확률을 얻는 규칙(검사 양성 → 실제 질병 확률 갱신).
- **왜 필요**: 지도/비지도/강화학습(C3)의 확률적 사고, softmax가 출력을 "확률"로 해석(C17)하는 근거.
- **출처/연결**: ML 전반의 확률 사고 → C3·C17.

### PK6. [CS기초] 과적합(Overfitting) / 정규화(Regularization)
- **쉬운 정의**: 과적합 = 학습 데이터만 잘 맞히고 새 데이터엔 약함(통째로 외운 학생). 정규화 = 모델을 단순화해 과적합을 누르는 기법.
- **왜 필요**: 학습곡선 진단(C18)·데이터 증강으로 과적합 감소(C19)를 이해하는 직접 전제.
- **출처/연결**: 신경망 학습의 일반화 → C18·C19.

### PK7. [CS기초] 교차검증(Cross-validation) / train·validation 분리
- **쉬운 정의**: 데이터를 학습용/검증용으로 나눠(또는 여러 폴드로 돌려) 새 데이터 성능을 추정하는 절차.
- **왜 필요**: C18의 "training vs validation accuracy" 과적합 진단 곡선이 이 분리를 전제한다.
- **출처/연결**: 모델 평가 절차 → C18.

### PK8. [CS기초] 신경망(인공신경망) 기초 — 가중치·층
- **쉬운 정의**: 입력에 가중치(weight)를 곱해 더하고 함수를 통과시키는 단위(뉴런)를 층으로 쌓은 모델. 학습 = 가중치를 데이터로 조정.
- **왜 필요**: C16~C19(뉴런→파라미터→역전파→CNN)가 "신경망이 뭔지"는 아는 상태에서 시작한다.
- **출처/연결**: DL 줄기의 출발선 → C16·C17·C18·C19.

### PK9. [도메인:의료] CNN·영상 픽셀/채널 기초
- **쉬운 정의**: 디지털 영상은 픽셀 격자이며 컬러는 R·G·B 3채널. CNN은 작은 필터(커널)를 영상 위로 슬라이드하며 특징을 뽑는 신경망.
- **왜 필요**: C19(CNN)에서 "RGB 채널·zero-padding·3D(W·H·depth) 배열"을 설명 없이 쓴다.
- **출처/연결**: 영상 입력의 형태 → C11·C19.

### PK10. [도메인:의료] 클래스 불균형(Class Imbalance)
- **쉬운 정의**: 한 부류(예: 정상)가 다른 부류(질환)보다 압도적으로 많은 상황. 그러면 "전부 정상"이라 찍어도 정확도가 높게 나와 속는다.
- **왜 필요**: 비지도 비정상 탐지(C3, "정상데이터만 있을 때")·진단 분류(C4)의 흔한 함정으로 전제된다.
- **출처/연결**: 의료 데이터 특성 → C3·C4.

### PK11. [수학] 벡터·행렬 곱 기초
- **쉬운 정의**: 숫자를 나열한 벡터, 격자로 늘어놓은 행렬. 행렬 곱은 신경망 한 층의 연산($z=xW$)을 한 번에 표현한다.
- **왜 필요**: C17의 행렬표현($z^{(2)}=xW^{(1)}$)·파라미터 수 계산이 벡터·행렬을 전제한다.
- **출처/연결**: 신경망 행렬표현 → C16·C17.

---

## 1. 시험 가이드 (Notion 출처 중심)

- ⭐⭐ **[Day1] 교수 강조: "시험은 객관식이 70~80%, 어렵지 않음"** (출처: Notion Day1). → 정의·개념 비교·매칭형 객관식 대비가 최우선. 깊은 유도/증명형보다 **용어·분류·장단점 매칭**에 집중.
- **[Day1] Ability-based Design (Edwards 1995)** 를 Introduction 첫 개념으로 강조 (출처: Notion Day1 + Week1 p.16 RESEARCH THEME). 예: "손가락 10개가 없는 사용자가 QWERTY 키보드 사용" → User-adaptation-System vs **Ability-Based System with AI**. ⭐ 시험에 개념 정의로 나올 가능성.
- **[Day2 과제] PPT 5장 내외·5분 발표** 구성: 논문 주제 선정 / 선행연구 5개 / 논문 간 비교표 / 내가 한다면 어떻게 (출처: Notion Day2 하단). **[Day3] "한계점도 정리하기"** 강조 (출처: Notion Day3).
- **명시적 "기출"은 없음** — 위 객관식 비중·강조 개념 외에 구체적 출제 문항 정보는 ⚠️ 노트에 없음.
- **출제 추정 핵심**(PDF 구조 + 강조점 종합, 검수 시 확정 필요):
  - AI/ML/DL 계층 구분 + SUMMARY 비교표 (Week1 p.27)
  - 지도/비지도/강화학습 비교표 (Week1 p.29)
  - 분류/회귀/군집 비교 + 평가지표(분류 label, 회귀 MSE/MAE) (Week1 p.34~38)
  - ECG 파형(P/QRS/T) 의미, EEG 5대역(δθαβγ) 주파수·상태 매칭 (Week2)
  - 영상 모달리티 X-ray/CT/MRI/US/PET/SPECT 원리·장단점·방사선 유무 매칭 (Week2)
  - 계측 시스템 5분류(Direct/Indirect, Invasive/Noninvasive, Contact/Remote, Sense/Actuate, Dynamic/Static) (Week3)
  - 신경망 파라미터 수 계산, 활성화함수, forward/backprop 개념 (Week3)

---

## 2. 개념 카탈로그

### C1. 의료 AI의 등장 배경 / Human-centered AI   ⭐中
- **출처**: Week1 p.3(Course Info), p.16(Research Theme), p.17(Research Direction) / Notion Day1
- **선행 개념**: 없음
- **한 줄 직관**: 고령화 + 센서·HW 발전 → 정량 데이터로 의료 진단을 보조. 시스템을 사람에게 맞추는 **Ability-based Design** 사상.
- **정의**: Ability-based Design(Edwards 1995) — 일반 시스템은 대표 사용자를 상정해 사용자가 시스템에 적응(User→adaptation→System)하지만, **Ability-Based System(+AI)**은 사용자의 능력에 시스템이 적응. 연구 3축: Custom Hardware + Signal Processing & AI + Human-centered Evaluation.
- **연구 방향**: Diagnosis(진단) / Treatment(치료: Therapeutic=비침습 신경조절, Assistive=보조기술) — 신경계 질환 중심.
- **시험 포인트**: "Ability-based Design"의 정의·예시(키보드 예) 객관식. 진단 vs 치료(therapeutic/assistive) 구분.

### C2. AI / ML / DL 계층 구조   ⭐⭐高
- **출처**: Week1 p.23–27 (개념·장단점·SUMMARY 표) / Notion Day1 헤더
- **선행 개념**: 없음 (가장 기초)
- **한 줄 직관**: AI ⊃ ML ⊃ DL. 포괄적 지능 → 데이터 학습 → 다층 신경망 자동 특징학습.
- **정의**:
  - **AI**: 인간의 인지능력(학습·추론·판단·문제해결)을 기계가 수행. 규칙기반~자가학습 포괄.
  - **ML**: 데이터로 시스템이 스스로 규칙·패턴 학습. 프로그래밍된 규칙 없이 학습. 지도/비지도/강화로 분류.
  - **DL**: 다층 인공신경망 기반. 비정형 데이터(영상·음성·자연어)에 강력. **특징(feature) 자동 추출**(End-to-End). CNN/RNN/Transformer.
- **핵심 메커니즘 (ML vs DL, Week1 p.26)**: **ML = Feature engineering**으로 문제를 여러 파트로 쪼개 답을 합침(Raw→사람이 feature 설계→ML모델→출력). **DL = feature engineering 불필요**(Raw→DNN 표현학습→출력).
- **표 인사이트 (SUMMARY, Week1 p.27)**: 비교축 = 데이터 필요량(AI 적음 < ML 중간 < DL 매우많음), 설명가능성(AI 높음 > ML 보통 > **DL 낮음=black box**), 처리 데이터(AI 정형/비정형, ML 정형 위주, DL 비정형 강함). **시험: "데이터 적게 필요 / black box / 비정형에 강함" 같은 특성을 AI·ML·DL에 매칭하는 객관식.**
- **의료 예시**: AI=응급실 우선순위 분류·챗봇 / ML=혈액검사 기반 질병예측 / DL=흉부 X-ray 폐렴 진단(CNN)·심전도 이상(RNN)·병리 슬라이드 암 진단.
- **흔한 함정**: DL은 ML의 하위 개념(AI⊃ML⊃DL). DL이 항상 우월한 게 아님(데이터·연산 많이 필요, 과적합·설명력 낮음).

### C3. 학습 패러다임: 지도 / 비지도 / 강화학습   ⭐⭐高
- **출처**: Week1 p.28–32 (비교표 + 각 상세) / Notion Day1 헤더
- **선행 개념**: C2
- **한 줄 직관**: 정답 있음(지도) / 구조 발견(비지도) / 보상 최적화(강화).
- **정의·메커니즘**:
  - **지도학습**: 입력 X + 정답 y(label) 쌍으로 학습, 손실함수(Loss) 최소화로 파라미터 조정. 분류(SVM·Logistic·Random Forest·k-NN·NN), 회귀(Linear·Ridge/Lasso·XGBoost). 의료: X-ray 폐렴 판단, 혈액검사 질병예측, 암환자 생존 회귀.
  - **비지도학습**: 입력만, 정답 없음. 거리·유사도·분포 기준으로 내재 구조 발견. 클러스터링(K-means·DBSCAN·Hierarchical), 차원축소(PCA·t-SNE·Autoencoder). 의료: 유전체 환자 군집, 생체신호 비정상 탐지(정상데이터만 있을 때).
  - **강화학습(RL)**: Agent가 Environment와 상호작용, Reward로 최적 policy 학습. 절차: 행동→환경이 보상+새 상태 반환→정책 개선→누적 보상 최대화. 기본 Q-learning·SARSA, 심화 DQN·PPO·A3C. 의료: 로봇 재활, 수술로봇 제어, 의료상담 AI.
- **표 인사이트 (Week1 p.29)**: **시험 핵심 매칭표**. 입력데이터(지도=입력+정답 / 비지도=입력만 / 강화=상태·행동·보상), 학습방식(오차 최소화 / 유사도 군집 / 누적보상 최대화), 단점(지도=라벨링 비용 큼 / 비지도=평가 어려움 / 강화=느리고 복잡·의료선 실험환경·윤리 어려움).
- **흔한 함정**: 강화학습은 "정답이 없는데도" 학습. 비지도는 "평가가 어려움(정답 없음)"이 핵심 단점.

### C4. 분류 / 회귀 / 클러스터링   ⭐⭐高
- **출처**: Week1 p.33–38 (비교표 + 각 상세, 입출력 예시표)
- **선행 개념**: C3
- **한 줄 직관**: 출력이 범주면 분류, 연속값이면 회귀, 라벨 없이 그룹화면 군집.
- **정의 + 표 인사이트 (Week1 p.34 비교표)**:
  | | 분류 | 회귀 | 클러스터링 |
  |---|---|---|---|
  | 문제유형 | 지도 | 지도 | **비지도** |
  | 출력 | 범주형(label, **이산 discrete**) | 연속형(숫자, **continuous**) | 그룹(군집 번호) |
  | 평가 | (정확도 등) | **MSE, MAE** (오차 크기 중요) | (군집 품질) |
  | 대표 알고리즘 | SVM·Decision Tree·CNN | Linear Regression·XGBoost | K-means·DBSCAN·Autoencoder |
- **의료 예시**: 분류=폐 CT 폐암 유무/병리슬라이드 양성·악성, 회귀=혈당/생존기간(개월)/심박출량(ml/beat) 예측, 군집=유전체 발현 환자 그룹화/정신건강 설문 증상군 분류.
- **시험 포인트**: "출력이 이산 vs 연속 vs 그룹" → 분류/회귀/군집 매칭. 회귀 평가지표 MSE/MAE 암기.

### C5. 의료 AI의 4대 장점   ⭐中
- **출처**: Week1 p.39 (2-4)
- **선행 개념**: C2
- **정의**: **속도 / 정확도 / 일관성 / 예측** (4가지). PDF엔 키워드만(Blank). ⚠️ 각 항목의 상세 설명은 PDF·Notion 모두 비어있음 → 일반 상식 기반 보강 필요 시 추측 금지.
- **시험 포인트**: 4대 장점 나열형 단답 가능.

### C6. 생체신호 처리 (Biomedical Signal Processing) 개요   ⭐中
- **출처**: Week2 p.6–7 / Notion Day2 헤더·표
- **선행 개념**: 없음
- **정의**: 신호처리 = 소리·이미지·생체신호 등을 분석·변형·합성해 유용 정보 추출·품질 향상. 기술=필터링·증폭·압축·특징추출. **생체신호처리** = 인체 생리신호를 분석해 임상 진단·모니터링·치료에 활용하는 특화 분야.
- **대표 생체신호**: ECG(심전도), EEG(뇌파), EMG(근전도), 그 외 EOG·ENG·PCG·VAG.
- **연구 4분야 (Week2 p.27)**: 향상(Enhancement, 잡음제거) / 탐지·추출(Detection&Extraction) / 해석·분석(Interpretation) / 분류·예측(Classification&Prediction).
- **처리 파이프라인 (Week2 p.27 표 — Day2 Notion 빈칸 채움)**: 데이터수집 → 전처리(필터링·정규화·기준선보정) → 세분화(Segmentation) → 노이즈제거(Artifact Removal: **ICA, PCA**) → 피크탐지(예: ECG R-피크) → 분류·패턴인식(SVM·DL) → 모델링·예측 → 시각화(시계열/주파수 스펙트럼).
- **라이브러리 (Week2 p.28)**: NumPy(수치), SciPy(필터·신호변환 scipy.signal), Pandas(시계열 전처리), Scikit-learn(SVM·PCA), TensorFlow/PyTorch(DL), Matplotlib(시각화·주파수분석).

### C7. ECG (심전도)   ⭐⭐高
- **출처**: Week2 p.8–10 / Notion Day2 (Notion은 P/QRS/T 의미 빈칸 → **Week2 PDF가 채움**)
- **선행 개념**: C6
- **정의**: 심장의 전기적 활동을 피부 표면에서 측정·기록. 리듬·속도·전도이상·허혈·심근경색 정보.
- **원리**: 전기자극이 **동방결절(SA node)** 에서 시작 → 방실결절(AV node) → 히스속 → 푸르키네섬유로 심장 전체 전파. 피부 표면 전압차로 기록.
- **유도 (Week2 p.8)**: 사지유도(I, II, III, aVR, aVL, aVF — 수평/수직), 흉부유도(V1~V6 — 전방·측면·후방).
- **파형 (Week2 p.9 — Day2 Notion 핵심 빈칸 채움)**:
  | 파형 | 의미 | 설명 |
  |---|---|---|
  | **P파** | 심방 탈분극 | 심방 수축 위한 전기자극 |
  | **QRS 복합파** | 심실 탈분극 | 심실 수축 위한 전기자극 |
  | **T파** | 심실 재분극 | 심실이 휴식상태로 복귀 |
  | U파(드물) | 심실 마지막 재분극 | 명확하지 않음 |
- **정량 (Week2 p.9 그림)**: 1회 심박동 0.8~1초, 1분당 60~80회. P wave 0.08–0.10s, QRS 0.06–0.10s, PR interval 0.12–0.20s, QTc ≤0.44s.
- **임상 활용 (Week2 p.10)**: ① 부정맥(심방세동·심실빈맥·조기박동) ② **심근경색(MI): ST상승(STEMI) vs 비ST상승(NSTEMI)** ③ 전도장애(방실차단 1~3도, 각차단 LBBB/RBBB) ④ 전해질이상(고칼륨→T파 고출현, 저칼슘→QT연장) ⑤ 심장비대.
- **장단점**: 장점=비침습·저렴·빠름·실시간. 단점=구조적 문제 확인 어려움·순간 상태만·판독자 의존.
- **시험 포인트**: ⭐ P/QRS/T = 심방탈분극/심실탈분극/심실재분극 매칭. STEMI vs NSTEMI. 전해질 이상의 ECG 변화.

### C8. EEG (뇌파)   ⭐⭐高
- **출처**: Week2 p.11–13 / Notion Day2 (Notion 주파수표는 채워져 있고 PDF도 동일)
- **선행 개념**: C6
- **정의**: 뇌의 전기적 활동을 두피 전극으로 측정. 뉴런 동기 활동 시 전위차를 증폭 기록. **10-20 시스템**으로 전극 배치.
- **5대 주파수 대역 (Week2 p.12 / Notion Day2 — 시험 단골 매칭)**:
  | 파형 | 주파수 | 상태 |
  |---|---|---|
  | **Delta (δ)** | 0.5–4 Hz | 깊은 수면 |
  | **Theta (θ)** | 4–8 Hz | 졸림·깊은 명상 |
  | **Alpha (α)** | 8–13 Hz | 안정·깨어있음(눈 감음) |
  | **Beta (β)** | 13–30 Hz | 집중·문제해결 |
  | **Gamma (γ)** | >30 Hz | 고차원 인지·기억 |
- **P300 (ERP)**: 자극 후 ~300ms에 나타나는 양(positive) 전위. 활용: 인지기능 평가(알츠하이머·ADHD), 정신질환 연구(조현병·우울증 진폭 감소), **BCI P300 Speller**(목표 글자 주의집중 시 ERP 인식).
- **임상 활용 (Week2 p.13)**: 간질(뇌전증) 진단, 수면장애(REM/NREM), 의식상태 모니터링(혼수), 정신과 평가, 두부외상(TBI) 후 감시, BCI.
- **특성**: 신호가 매우 약해 노이즈 민감 → 고해상도 기기·전처리 중요.
- **시험 포인트**: ⭐ 5대역 주파수↔상태 매칭(δ수면, α안정·눈감음, β집중 등). P300=300ms positive ERP.

### C9. EMG (근전도)   ⭐中
- **출처**: Week2 p.14–22 (가장 상세) / Notion Day2
- **선행 개념**: C6
- **정의**: 근육 수축 시 전기적 활동 측정 → 근육·신경계 상태 평가. 신경근 접합부 기능·말초신경 손상·근육질환.
- **측정방식**: **표면 EMG(sEMG)** — 비침습, 재활·로봇제어·운동분석 / **침습 EMG(needle)** — 바늘 전극, 진단 정확도 높음, 임상 진단용.
- **신호 특성 (Week2 p.16)**: Amplitude(μV~mV, 힘 세기), Frequency(20~500 Hz, 신경지배·근섬유반응), Duration(신경-근육 전도시간), Recruitment pattern(점진적 활성화). 운동단위(Motor Unit) 발화빈도·동원패턴 → 신경·근육 건강성.
- **임상 (Week2 p.17)**: 말초신경질환(근위축증·신경병증·**루게릭병 ALS**), 근육기능 평가, 재활·보행분석(뇌졸중·척수손상), 로봇·웨어러블 제어(EMG 의수·외골격), 운동/근피로 평가.
- **표/그래프 인사이트 (Week2 p.18–19, Delsys)**: **Proper sensor location → 신호↑·SNR↑·cross-talk 감소.** Cross-talk 그래프: 전극 spacing↑·bar width↑ → cross-talk(%)↑ (bar width 7.5mm가 1mm보다 높음). EMG 필터링 대역 **20–450 Hz**(baseline noise·artifact 제거).
- **시스템 블록도 (Week2 p.21 = Week3 EMG Controlled Car)**: Muscle→EMG→[Instrumentation Amp→Analog Filters→Rectifier = Signal Conditioning]→ADC(NI DAQ)→Signal Processing(Computer)→Command(MCU)→Motor Control→Car Motors(Transducer). → **계측 시스템 3블록(Transducer/Electronics Interface/Computation Unit) 실제 적용 예.**
- **시험 포인트**: sEMG vs needle EMG. 신호 특성 4요소. 적절한 센서 위치의 효과(SNR↑, cross-talk↓).

### C10. 기타 생체신호: EOG / ENG / PCG / VAG   ⭐低
- **출처**: Week2 p.23–26 (Day2 Notion은 헤더만 — **PDF가 정의 보강**)
- **선행 개념**: C6
- **정의 (Week2 PDF)**:
  - **EOG (Electrooculography, 눈전도)**: 눈 움직임 감지 전기신호. 안구 움직임에 따라 눈 주변 전극 전위차 → 눈동자 위치·움직임 추적. 활용: gaze tracking, 수면연구·안과, 장애인 통신 인터페이스, VR/AR.
  - **ENG (Electronystagmography, 전정안진검사)**: 안구의 무의식적 움직임(안진, nystagmus) 측정 → **전정기관(귀 평형감각)** 평가. 원리는 EOG 유사, 어지럼증·평형장애 특화. 활용: 어지럼증·전정신경염·메니에르병, VEMP·체평형 검사 병용.
  - **PCG (Phonocardiography, 심음도)**: 심장 심음(heart sounds)을 마이크로폰으로 기록한 음향 생체신호. 판막 기능·잡음(murmur) 청진음 시각화. 활용: 심잡음 탐지(판막협착·폐쇄부전), **AI 기반 심음분석** 조기진단, 디지털 청진기. (그림: ECG-PPG-PCG 동시기록, S1/S2)
  - **VAG (Vibroarthrography, 진동관절음 검사)**: 관절 움직일 때 미세 진동/소리 감지 → 관절 상태 평가. 연골 손상·퇴행성 변화 시 비정상 마찰음. 활용: 무릎·팔꿈치·**턱관절(TMJ)**, 퇴행성 관절염(osteoarthritis) 조기진단, 재활 모니터링.
- **시험 포인트**: 각 신호 이름↔측정 대상 매칭(EOG=눈, ENG=전정/평형, PCG=심음, VAG=관절음).

### C11. 의료영상 모달리티 (Medical Imaging)   ⭐⭐⭐高
- **출처**: Week2 p.29–60 (방대) / Notion Day2 (대부분 빈칸 → **Week2 PDF가 전면 보강**)
- **선행 개념**: 없음 (영상진단 딥러닝 C12의 입력)
- **모달리티별 핵심 (원리/대상/방사선/장단점)**:
  - **X-ray**: 고에너지 전자파, 조직 밀도차 흡수. 뼈=칼슘 高→흰색, 공기·연조직=어둡게. 활용=정형외과(골절)·흉부(폐렴·폐암·기흉·결핵)·치과·소화기(Barium)·심혈관. 장점=빠름·저렴·비침습·골격 탁월 / 한계=연조직 해상도 낮음·방사선·2D 깊이감 부족.
  - **CT**: 여러 방향 X-ray를 컴퓨터 재구성 → 단면(슬라이스)·**3D 볼륨렌더링**. 밀도차 정밀. 활용=뇌신경계(뇌출혈·뇌경색)·흉부·복부/골반·근골격·CT Angio. 장점=고해상도·3D·빠름 / 한계=방사선·조영제 부작용·고가.
  - **MRI**: 강한 자기장+고주파(RF). **수소 원자핵(프로톤)** 정렬→RF 자극·방출신호 분석. **연조직 정밀**(뇌·척수·관절·인대·연골). 확장: **fMRI**(뇌혈류→기능), **DWI/DTI**(뇌졸중 조기·신경섬유 추적), **MR Spectroscopy**(대사물→종양 성격). 장점=**방사선 없음**·연조직 탁월·다양기법(T1/T2/fMRI) / 한계=고비용·시간 길고 소음·금속 제한.
  - **Ultrasound(초음파)**: 고주파 음파(1~20MHz) 투사·반사파 분석. **실시간**·태아·혈류. **Doppler**=혈류 속도·방향. 장점=**방사선 없음**·실시간·휴대·저렴 / 한계=공기·뼈 통과 어려움·화질 제한·검사자 숙련 의존.
  - **PET**: 방사성 동위원소(**FDG=포도당 유사**) 주입 → 대사 활발한 부위(암세포)에 집중 → 양전자 방출·전자 충돌→감마선(γ) → 3D 재구성. **대사·기능 영상**. 활용=종양학(전이·치료반응)·신경과(알츠하이머)·심장학·정밀의학(theranostics). 장점=기능변화 조기탐지·구조+기능통합(PET/CT, PET/MRI)·전이/재발 탁월 / 한계=고비용·방사성물질(반감기 짧음)·시간 김.
  - **SPECT**: 감마선 방출체(Tc-99m, I-123) 주입 → **단일 광자** 감지 → 감마카메라 회전 → 3D. 활용=심장(심근관류→허혈/경색)·신경과(뇌혈류·치매)·근골격(골전이)·내분비(갑상선)·종양. 장점=혈류·장기기능·기기 보편·3D / 한계=해상도 낮음·시간 김·대사분석 PET보다 낮음.
  - **Mammography(유방촬영)**: 유방 전용 저용량 X-ray. 미세 종양·석회화 조기발견. 2D / **3D(Tomosynthesis)** / 조영증강(CESM). 장점=조기발견·효율·접근성 / 한계=**젊은 여성 치밀유방 민감도 낮음**·방사선 누적·압박 불편.
  - **Fluoroscopy(투시조영)**: X-ray 실시간 연속 조사 → 내부 구조·움직임 관찰. 조영제로 혈관·소화기·요로 흐름. C-arm. 활용=소화기·심혈관(스텐트)·정형/신경외과·비뇨기·인터벤션. 장점=실시간·시술 위치조정 / 한계=**방사선 노출시간 증가**·해상도 제한·조영제 알레르기.
  - **DSA(디지털 감산 혈관조영)**: 조영제 주입 전(mask)·후 영상을 **디지털 감산(subtraction)** → 뼈·연조직 배경 제거·**혈관만 선명**. 고급 fluoroscopy. 활용=신경과(뇌동맥류·AVM)·심혈관·인터벤션. 장점=고대비 혈관·실시간 가이드·작은 병변 / 한계=방사선·조영제↑·환자움직임 민감·고가.
  - **OCT(광간섭 단층촬영)**: 근적외선 빛의 **간섭(interference)** → 미세 층구조를 **μm 단위** 초고해상도·비침습. 활용=안과(망막·황반변성·녹내장·당뇨망막병증)·심혈관 내시경(IVOCT 죽상경화)·피부·치의학. 장점=μm 고해상도·비침습/비접촉·실시간·정량 / 한계=빛 투과 어려운 조직 제한·깊은 조직 어려움·고가.
  - **Dermatoscopy(피부경)**: 피부 표면 병변 비침습 고배율(10~20배) 관찰. 광학조명+편광필터로 반사광 제거. **ABCD Rule**(Asymmetry/Border/Color/Diameter), 7-point checklist, Menzies, Chaos&Clues, **AI 멜라노사이트 병변 자동분류**. 활용=피부암(흑색종·BCC·SCC) 감별·양성병변·경계추적. 장점=비침습·빠름·고배율·조기진단율↑ / 한계=숙련 필요·깊이 한계·병리 대체 불가.
- **비교표 인사이트 (시험 단골)**:
  - **X-ray vs CT vs MRI vs US (Week2 p.41)**: 방사선 유무 = X-ray/CT **있음**, MRI/US **없음**. 실시간 = US만 ✅. 금기 = MRI 금속이식물, US 피부상처/공기.
  - **CT vs MRI vs PET (Week2 p.44)**: 목적 = CT 해부구조 / MRI 연조직해상도 / PET 대사·기능. 조기진단 = PET만 "가능(대사이상 포착)".
  - **PET vs SPECT (Week2 p.46)**: PET=양전자 방출체(F-18, Ga-68)·쌍감마선 동시검출·고해상도·고가 vs SPECT=감마선 방출체(Tc-99m, I-123)·단일감마선·저해상도·저렴·보편.
  - **유방암 적합 도구 (Week2 p.47)**: Mammography(1차 스크리닝, 저용량 X-ray) + Ultrasound(보완, 치밀유방) + MRI(고위험군) + PET(전이평가).
  - **CT vs MRI vs OCT (Week2 p.52)**: 해상도 = CT mm / MRI mm~sub-mm / **OCT μm**. OCT는 얇은 조직 단면(망막·혈관) 완전 비접촉.
- **시험 포인트**: ⭐⭐ 모달리티↔원리/방사선유무/주대상/장단점 매칭이 핵심 객관식. "방사선 없는 것은?"(MRI·US·OCT), "실시간은?"(US·Fluoroscopy), "대사 영상은?"(PET), "혈관만 보는 것은?"(DSA), "μm 해상도는?"(OCT).

### C12. 디지털 전환 (Digital Transformation) — 의료   ⭐低
- **출처**: Week2 p.4–5
- **정의**: 디지털 기술을 조직 전반에 통합해 운영방식·고객가치를 근본 변화. 의료 미래기술 = 개인맞춤의학·디지털헬스 플랫폼·웨어러블·AI/ML 진단·원격진료·3D프린팅/로봇·블록체인 보안.
- **워크플로우 (p.5)**: ①사전평가·전략 ②이해관계자 참여 ③기술도입(EHR·원격진료·웨어러블) ④교육·채택 ⑤모니터링·피드백 ⑥반복개선.
- **시험 포인트**: 미래 헬스케어 기술 나열형 가능. (비중 낮음)

### C13. 계측 시스템 기본 모델 (Instrumentation System)   ⭐⭐⭐高
- **출처**: Week3 p.5–14 / Week1 p.43–45 / Notion Day3 (Day3 핵심 주제)
- **선행 개념**: C6
- **한 줄 직관**: 몸의 신호를 측정/제어하는 3블록 파이프라인.
- **3 Primary Subsystems**: (1) **Transducer** (2) **Electronics Interface** (3) **Computation Unit**. (Tissue/Physiologic signal과 Energy source 포함)
- **세부 (Week3)**:
  - **Transducer (Sensor + Actuator)**: 센서=생리신호 감지, 액추에이터=자극/제어. 신호의 modality(bioelectric·biochemical·pressure)·source(body 위치)에 매칭, dynamic range·time response 일치. **Sensor must**: 생화학·생전기·생물리 파라미터 감지 / 생리적 시간응답 재현 / 안전 인터페이스. **Actuator must**: 외부 agent 전달 / 파라미터 제어 / 안전 인터페이스. 대부분 transducer는 **voltage 신호** 생성 → electronics interface로 conditioning.
  - **Electronics Interface (Pre-amplifier → Signal Conditioning → Buffer)**: transducer 출력을 A/D 변환·샘플링 가능한 신호로 변환. 기능=센서/액추에이터와 computation unit 전기특성 매칭·**SNR 보존**·**대역폭(시간응답) 보존**·안전.
    - **Pre-amplifier**: 센서 매칭 특수 증폭기. fixed gain, 센서 대역폭 초과, **저잡음**, differential inputs(특히 bioelectric). **시스템 잡음 특성은 주로 pre-amp가 지배.**
    - **Signal Conditioning**: 전압변환·증폭·필터링·정류(Rectification)·변조(Modulation)·수학적 조작.
    - **Buffer**: 긴 케이블 구동용 증폭(+전력). **front-end와 back-end가 1~2m 이상 떨어질 때만** 필요.
  - **Computation Unit**: 기능=primary UI·전체 제어·데이터 저장·신호처리·안전 운영. 구성=Data converter·DSP·Data storage·UI/Display/Input·Communications·Control.
- **시험 포인트**: ⭐ 3블록 이름·순서, 각 블록 기능. Pre-amp가 시스템 잡음 지배. Buffer는 원거리 구성에서만 필요.

### C14. 측정·계측의 정의 (Measurement / Instrumentation)   ⭐中
- **출처**: Week3 p.15–17
- **정의**: **Measurement system** = 물리·전기량(질량·온도·압력·정전용량·전압) 측정, 사건/위치 탐지(지진 진앙), 객체 식별·계수(적혈구). "측정 못하면 제어 못한다." 요구사항=정확도·해상도·무게·부피·비용·환경. **Instrumentation** = 측정기기로 프로세스를 monitor·control 하는 기술. **Biomedical Instrumentation** = 몸으로부터/몸으로 데이터를 측정·기록·전송하는 기기 분야.
- **시험 포인트**: 측정 시스템 요구사항, "측정 못하면 제어 못한다" 명제.

### C15. 생체계측 시스템 5대 분류 (Types)   ⭐⭐⭐高
- **출처**: Week3 p.18–24 / Notion Day3 (Day3 표로 강조 — **PDF가 예시 보강**)
- **선행 개념**: C13, C14
- **5가지 dichotomy + 예시 (시험 핵심 매칭)**:
  | 분류 | 정의 | 예시 (Week3) |
  |---|---|---|
  | **Direct / Indirect** | 생리 파라미터를 직접 vs 관련 파라미터로 측정 | 직접=동맥 평균 혈류량 / **간접=ECG**(체표면 기록은 심장 활동전위 전파와 관련될 뿐, 전파 파형 자체 측정 아님) |
  | **Invasive / Noninvasive** | 몸에 장치 삽입 여부 | 침습=신경섬유 활동전위 직접 기록(implantable electrode) / 비침습=초음파 color flow(경동맥 혈류) |
  | **Contact / Remote** | 대상 접촉 여부 | 접촉=근섬유 strain gauge / 원격=MRI·초음파(비접촉 내부 변형·힘 측정) |
  | **Sense / Actuate** | 읽기 vs 액션 | 직접·접촉 actuator=**자동 인슐린 펌프** / 원격·비침습 actuator=**HIFU**(고강도 집속초음파 수술) |
  | **Dynamic / Static** | 시계열 변화 정도 | Dynamic(실시간, 시간응답 ≤ 생리 시상수)=초음파 Doppler(심주기 내 동맥혈속) / Static=생리 파라미터의 **시간 평균(temporal average)** 측정 |
- **시험 포인트**: ⭐⭐ 각 분류의 예시 매칭이 단골(특히 ECG=간접, 인슐린펌프=actuator/contact, HIFU=actuator/remote). "동적 vs 정적" 정의(시간응답 vs 시간평균).

### C16. 인공뉴런 / 활성화함수 / 퍼셉트론   ⭐⭐高
- **출처**: Week3 p.30–33
- **선행 개념**: C2
- **수식 (뉴런, Week3 p.30)**: $z = w_1 x_1 + w_2 x_2 + w_3 x_3 + \text{bias} = \sum_i w_i x_i + \text{bias}$ — 입력 가중합 + 편향. (data input · weights · summation+bias · activation)
- **활성화함수 (Week3 p.31)**: 층 출력에 적용해 **비선형성 부여**(없으면 NN이 선형회귀로 축소). 3종:
  - **Sigmoid**: $f(x) = \dfrac{1}{1+e^{-x}}$ — (0,1)
  - **TanH**: $\tanh(x) = \dfrac{2}{1+e^{-2x}} - 1$ — (−1,1)
  - **ReLU**: $f(x) = \begin{cases} 0 & x<0 \\ x & x\ge 0 \end{cases}$
- **퍼셉트론 (Week3 p.32)**: 뉴런 + 계단형 활성화 ($f(z)=1$ if $\sum wx+\text{bias}\ge0$, else 0). 가중치 작은 변화가 큰 출력 변화 유발 → 다른 활성화함수 사용. **단일 퍼셉트론은 XOR 표현 불가**. **MLP**(multilayer perceptron) = 여러 퍼셉트론·완전연결층 + 다양한 활성화함수.
- **시험 포인트**: ⭐ 활성화함수의 목적(비선형성, 없으면 선형회귀). Sigmoid/TanH/ReLU 식·치역. 단일 퍼셉트론 XOR 불가.

### C17. 신경망 구조 / 파라미터 수 계산 / 행렬표현   ⭐⭐高
- **출처**: Week3 p.34–35
- **선행 개념**: C16
- **파라미터 수 계산 (Week3 p.34 — 시험에 나올 계산형)**: 각 층 파라미터 = (출력노드 수) × (입력노드 수 + 1[bias]).
  - 예: 1st hidden = 4×(6+1)=**28**, 2nd hidden = 3×(4+1)=**15**, output = 1×(3+1)=**4**, **Total = 47**.
- **행렬표현 (Week3 p.35)**: $x=[x_1,x_2,x_3]$(=$a^{(1)}$), $W^{(1)}$=3×4, $z^{(2)}=xW^{(1)}$(4-dim), $a^{(2)}=f(z^{(2)})$ … 최종 $y=\text{softmax}(z^{(4)})$. **softmax**는 마지막 층 선호(값을 확률로 해석 가능).
- **시험 포인트**: ⭐ 파라미터 수 계산 = (출력×(입력+1)) 합. softmax는 출력층(확률 해석).

### C18. Forward / Backward Propagation / 학습 절차   ⭐⭐高
- **출처**: Week3 p.36–48
- **선행 개념**: C17
- **Forward propagation**: 입력 X → 은닉층 $a^{(2)},a^{(3)},a^{(4)}$ 순차 계산 → 출력 $\hat{y}$.
- **Backward propagation**: 출력 $\hat{y}$ vs 정답 $y$ 손실 $J(y,\hat{y})$ → 출력층 가중치부터 역순으로 gradient $\dfrac{\partial J(y,\hat{y})}{\partial W_4}, \dfrac{\partial J}{\partial W_3}, \dots, \dfrac{\partial J}{\partial W_1}$ 계산(체인룰, 역순 전파).
- **학습 5단계 (Week3 p.43 TRAINING STEPS)**: ① Feed network(입력·batch·epoch·가중치 초기화) → ② Make prediction(출력) → ③ Compare with ground truth(**손실함수 J 계산**) → ④ Calculate gradients·Update params(**∂J/∂wᵢ, backpropagation**) → ⑤ Iterate & monitor.
- **그래프 인사이트 (Week3 p.44 TRAINING: ITERATE & MONITOR — 시험 진단형)**:
  - **Loss vs epoch (학습률 진단)**: very high LR → 발산(상승), low LR → 너무 느림, high LR → 정체(플래토), **good LR → 부드럽게 수렴**. → "곡선 보고 학습률 진단" 출제 가능.
  - **Accuracy vs epoch (과적합 진단)**: training accuracy↑인데 validation accuracy가 크게 뒤처지면 **strong overfitting**, 약간 뒤처지면 little overfitting.
- **시험 포인트**: ⭐ forward=입력→출력, backward=손실→gradient 역전파. 학습 5단계 순서. Loss 곡선으로 학습률·과적합 진단.

### C19. CNN (Convolutional Network) / 이미지 분석   ⭐⭐高
- **출처**: Week3 p.45–56
- **선행 개념**: C18, C11
- **왜 CNN? (Image Analysis, Week3 p.45–48)**: 이미지를 1D로 펴서 FC(완전연결)에 넣으면 **FC는 이미지의 공간 구조를 무시**(픽셀 permute해도 가중치만 permute되고 결과 동일). → 공간 구조 보존 위해 CNN 필요.
- **합성곱 메커니즘 (Week3 p.49–53)**: 입력 = color channels(RGB, zero-padding 적용) → **kernels(필터)** 와 합성곱 + **bias** → output channel. 여러 kernel 세트 → 여러 output channel. CNN은 뉴런을 **3차원(width·height·depth)** 으로 배열(일반 NN은 1D).
- **필터 예시 (Week3 p.54)**: Vertical Line Detector / Horizontal Line Detector / Corner Detector (각 3×3 가중치 패턴). **Filter(kernel) = 입력과 합성곱되는 가중치 집합.**
- **과적합 감소 (Week3 p.55 OVERFITTING REDUCTION)**: **데이터 증강(data augmentation)** — rotation, flip, crop, stretching, translation, resize, contrast, blur, RGB→BGR, color inversion.
- **시험 포인트**: ⭐ "FC는 공간 구조 무시" → CNN 필요성. Filter=합성곱 가중치 집합. CNN은 3D(W·H·depth) 배열. 데이터 증강이 과적합 감소.

### C20. (교수 연구 사례) Tremor 진단·치료·보조 기술   ⭐低
- **출처**: Week1 p.18–24 (DIAGNOSIS/TREATMENT)
- **선행 개념**: C1
- **내용 (정성 인사이트)**:
  - **진단(Tremor)**: 기존=종이 나선/필기 그리기(주관적). 신규=컴퓨터 표준화 과제(Gyration Air Mouse·Wrist Device)로 정량화. **TETRAS 0~4 등급별 나선/직사각 추적 궤적 악화**. Fitts' law 기반 과제로 **파라미터 차원(dimensionality)↑**. 성능지표: Movement time, Path efficiency, Error rate, **Throughput**, Outside area.
    - **그래프 인사이트 (Week1 p.19)**: Path Efficiency vs Tremor Power = 음의 상관(R²=0.48, 떨림 강할수록 효율↓), Throughput vs Tremor Frequency = 양의 상관(R²=0.70). 응용: 임상 등급 참조·고해상도/고차원·반복성·재택 장기추적.
  - **치료(Therapeutic)**: 비침습 E-Stim(전기자극)으로 떨림 변조/억제. 가속도계 spectrogram에서 자극 ON 시 떨림 감소 확인. 폐루프 phase-locked 자극 파라미터 최적화(Amplitude·Frequency·Duty Cycle·Phase).
  - **치료(Assistive)**: 마비 환자의 혀 움직임 추출 → **Tongue Pattern Recognition**(자기 센서, PCA 공간 분류). MORA(intra-Oral Assistive). 휠체어·텍스트입력·로봇 외골격 제어.
- **시험 포인트**: 객관식 비중 낮음. Throughput·Path efficiency 등 성능지표 용어 정도. (교수 본인 연구라 맥락 이해용)

---

## 3. 개념 의존성 맵

```
[Week1 / Day1 — AI 기초]
C1(의료AI배경·Ability-based) ─┐
C2(AI/ML/DL) → C3(학습패러다임) → C4(분류/회귀/군집)
C2 → C5(의료AI 4대장점)
C2 → C16 → C17 → C18 → C19(CNN)   ← (신경망 흐름, Week3에서 전개)

[Week2 / Day2 — 신호·영상]
C6(생체신호처리) → C7(ECG), C8(EEG), C9(EMG), C10(EOG/ENG/PCG/VAG)
C11(의료영상 모달리티) ──→ C19(CNN 영상분석의 입력)
C12(디지털 전환)  ← 독립 도입

[Week3 / Day3 — 계측 + 신경망]
C6 → C13(계측 3블록) → C14(측정·계측 정의) → C15(5대 분류)
C9(EMG) ──예시──→ C13(EMG controlled car = 계측 3블록 실례)
C16(뉴런) → C17(구조·파라미터수) → C18(forward/backprop) → C19(CNN)
```

핵심 선수 관계 요약:
- AI/ML/DL(C2)을 알아야 학습패러다임(C3)·신경망(C16~)을 이해.
- 생체신호처리(C6)가 ECG/EEG/EMG(C7–9)와 계측 시스템(C13)의 공통 뿌리.
- 뉴런(C16)→파라미터(C17)→역전파(C18)→CNN(C19)은 순차 누적, CNN은 영상(C11)과 결합.

---

## 4. 누락·확인 필요 목록 (⚠️)

- ⚠️ **Week4 PDF 미제공**: 스케줄상 "의료 AI의 한계·윤리·규제·실무 적용"(4주차)은 자료가 없음. 시험 범위에 포함되면 별도 보강 필요. (Notion Day4 노트도 없음 — Day1/2/3만 존재)
- ⚠️ **Week1 p.39 "의료 AI 4대 장점"(속도·정확도·일관성·예측)**: 키워드만 있고 각 설명 Blank. PDF·Notion 모두 상세 없음 → 임의 보강 금지.
- ⚠️ **Notion Day1 헤더 "#의료 진단과 인공지능의 관계 이해"** 아래 본문이 empty-block. 그러나 해당 내용은 Week1 PDF(C2~C5)가 충분히 커버하므로 실질 공백 아님.
- ⚠️ **Notion Day3 마지막 "#연구 과제 - 한계점도 정리하기 -"** 외 본문 빈약. Day3의 강의 본체(계측 5분류·신경망)는 Week3 PDF가 충분히 보강.
- ⚠️ **Notion Day2의 다수 표 빈칸**(CT/MRI/US/PET/SPECT/Mammo/Fluoro/DSA/OCT/Dermato 정의·활용, 생체신호처리 단계·연구분야, ECG 파형 의미, EMG 임상 예시): **대부분 Week2 PDF가 채움**(C7·C11에 반영). 남은 미세 공백: EMG "임상 및 응용 예시" 표(Notion 완전 빈칸)는 Week2 p.17이 채움. CT/MRI 등 "원리·활용 목적" 세부는 Week2 PDF 텍스트로 모두 보강됨 → **실질 미해결 빈칸 거의 없음**.
- ⚠️ **시험 기출 문항·구체 출제 범위**: Notion에 "객관식 70~80%, 안 어렵다"는 강조만 있고 구체 기출은 없음. C1·C13·C15·C11 비교표가 출제 핵심일 가능성이 높다는 것은 PDF 구조 기반 추정.
- ⚠️ Week1 p.20·p.24 등 일부 슬라이드(SUMMARY 등)는 본문 없이 제목만(Blank). 강의 중 구두 설명분 → 자료상 복원 불가.
