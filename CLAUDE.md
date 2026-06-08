# CLAUDE.md

한양대 AI융합대학원 4과목 기말 시험대비 학습 허브(단일 HTML·오프라인 우선). 상세 구성은 `README.md`.

## 하네스: 학습 허브 제작

**목표:** 4과목 학습 자료를 "이해시키는 설명 + 진짜 시각화"로 만든다. 안될공학식 설명 빌드업·영어(한글) 병기·3Blue1Brown 애니메이션·실전 예상문제(모범답안)·선행지식·정리된 목차·페르소나 워딩을 갖춰 제작·개선한다.

**트리거:** 학습자료/학습허브/과목 페이지 제작·개선 요청 시 `study-hub-builder` 스킬(오케스트레이터)을 사용한다. "설명 깊이·비유·영어 병기·3B1B·애니메이션·시뮬레이션·탐색 트리·선행지식·논문 포스터·예상문제·모범답안·워딩·목차 정리·Codex 병렬·다시/재실행/보완·이 과목만 다시" 류 포함. 단순 질문은 직접 응답.

**원장:** `D:\user\Documents\hj_works\11_wiki\haje\` (페르소나 vault) — voice/tone.md + working-style/presentation-style.md + **working-style/explanation-style.md**(안될공학+3B1B 설명·시각 DNA, 영어 병기 규칙).

**변경 이력:**
| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-06-04 | 초기 구성 (에이전트 4 + 스킬 3 + 오케스트레이터 1) | 전체 | 트리맵→탐색트리·선행지식·인터랙티브·워딩 개편 요청 |
| 2026-06-05 | v1.1 진화: concept-explainer·exam-designer 에이전트 신설(팀 4→6), concept-explanation·exam-design 스킬 신설, interactive-viz 3B1B 전환(자체 경량 엔진 ref), persona-wording 영어 병기 규칙, 오케스트레이터 Phase 재배치+Codex 병렬 트랙. vault explanation-style.md 원장화 | agents/skills/오케스트레이터/vault/CLAUDE.md | 설명 나열·가짜 시각화·부실 예상문제·번역어 단독표기 피드백 |
| 2026-06-05 | explanation-style.md 자막 근거 전면개정(안될공학 3편+3B1B 4편 yt-dlp 정독). 표준 빌드 스펙 `_workspace/v1.1_spec.md` 확립(Codex C16을 기준예시로) | vault/_workspace | 추정 아닌 근거 요구·Codex 워딩·시각화 채택 |
| 2026-06-05 | v1.1 전면 빌드: 4과목×2트랙(Claude 루트 / Codex codex-v1.1)+랜딩 재디자인. 5단 빌드업·영어병기·canvas 모핑 애니메이션·실전 예상문제 | 전체 HTML | 사용자 채택 스타일로 전과목 제작·양트랙 유지 요청 |
| 2026-06-08 | biomed-ai 심화: 유튜브 9편 full-script(yt-dlp) 분석 → MECE 통합 관점 섹션(CNN/GNN/GAT/self-attention=이웃 가중합, 학습엔진=GD+BP 비교표) + convolution 정의·gradient descent step·attention Q/K/V step 신설, BP 직관·BERT NSP·3임베딩·GAT 동기 보강, 계산형 예상문제 4개·신규 SVG(gdSvg·attnSvg)·하단 영상 토글박스 | biomed-ai.html | "개념 이해 어려움 — 영상 기반 breakdown·재연결·계산문제·영상링크" 요청 |
