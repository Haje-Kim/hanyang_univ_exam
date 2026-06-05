---
name: study-hub-builder
description: 한양대 AI융합대학원 기말 학습 허브(4과목 HTML)를 제작·개선하는 오케스트레이터. 목차 정리·선행지식·개념 설명 심화(안될공학식 빌드업·영어 병기)·3Blue1Brown 스타일 애니메이션 시각화·실전 예상문제(모범답안)·페르소나 워딩을 전문 에이전트 팀으로 조율하고, 필요 시 Codex 병렬 빌드(apple-to-apple 비교)를 띄운다. 학습자료/학습허브/과목 페이지를 만들거나 고칠 때, "설명 깊이·비유·영어 병기·3B1B·애니메이션·시뮬레이션·탐색 트리·선행지식·논문 포스터·예상문제·모범답안·워딩 바꿔·목차 정리·Codex 병렬·다시 실행·재실행·업데이트·보완·이 과목만 다시" 류 요청에 반드시 사용.
---

# Study Hub Builder — 학습 허브 제작 오케스트레이터 (v1.1)

4과목 학습 허브(`ai-intro.html` 인공지능개론 · `algorithms.html` 컴퓨터 알고리즘 · `biomed-ai.html` 의생명 AI · `medical-dx-ai.html` 의료진단 AI)를 전문 에이전트 팀으로 제작·개선한다.

## 팀 구성 (실행 모드: 하이브리드)

| 에이전트 | 역할 | 스킬 | 모드 |
|---|---|---|---|
| content-architect | 목차 정리 + 선행지식 | content-architecture | 과목별 병렬 |
| concept-explainer ⭐ | 개념 설명 심화(빌드업·비유·영어 병기) | concept-explanation | 과목별 병렬, 구조 확정 후 |
| interactive-viz-engineer ⭐ | 3B1B 애니메이션 씬·시뮬레이션·탐색트리·논문포스터 | interactive-viz | VIZ 마커 받아 병렬 |
| exam-designer ⭐ | 실전 예상문제 + 모범답안 | exam-design | 개념 커버리지 확정 후 |
| persona-wordsmith | 표면 워딩 마감 + voice/tone | persona-wording | 콘텐츠 확정 후 |
| harness-qa | 오프라인·렌더·앵커·반응형 점진 검증 | (general-purpose) | 모듈 완성 직후 |

> 모든 Agent 호출에 `model: "opus"` 명시. 과목 단위는 독립적이라 병렬, 과목 내부는 **구조 → 설명 → 시각화 → 예상문제 → 워딩** 파이프라인 순서 의존.
> v1.0→v1.1 핵심 변화: ① concept-explainer 신설(나열→이해), ② viz가 3B1B 애니메이션으로, ③ exam-designer 신설(박스3줄→실전), ④ 영어(한글) 병기 전면, ⑤ Codex 병렬 트랙.

## Phase 0: 컨텍스트 확인 (먼저)

1. `_workspace/` 존재 여부 확인.
   - 없음 → **초기 실행**(전체 파이프라인)
   - 있음 + 부분 수정 요청("이 과목만", "예상문제만", "시각화만") → **부분 재실행**(해당 에이전트만, 나머지 산출물 재사용)
   - 있음 + 새 전면 요청 → 기존 `_workspace/`를 `_workspace_prev/`로 이동 후 새 실행
2. `voice/tone.md`·`working-style/explanation-style.md`(`11_wiki/haje/`) 상태 확인 — explanation-style은 v1.1 설명·시각 DNA 원장, 작업 전 팀에 로드 지시.
3. **Codex 병렬 여부** 확인 — 사용자가 apple-to-apple/Codex 비교를 요청했으면 Phase 5 병렬 트랙 활성화(기본 산출 위치 `codex-v1.1/`).
4. 변경 범위를 사용자에게 한 줄 확인(모호할 때만).

## Phase 1: 구조 (과목별 병렬, content-architect)

- 목차 재점검·정리 → `_workspace/{과목}_toc.md`
- 선행지식 섹션(본문 진입 전) — 용어는 영어(한글) 병기
- id 안전 규칙(기존 앵커 보존)

→ **harness-qa**: 앵커 정합성(toc id ↔ 본문 섹션 id) 즉시 검증.

## Phase 2: 설명 심화 (과목별 병렬, concept-explainer) — v1.1 신규

`_workspace/{과목}_toc.md`의 각 핵심 개념을 6단계(Hook→직관→비유→정의→메커니즘→함의)로 재구성:
- "요약·번역 나열" 제거, 이해시키는 설명으로 전환
- 영어 원문 용어 전부 `영어(한글)` 병기 → `_workspace/{과목}_glossary.md`
- 단계형·구조형 개념에 `<!-- VIZ: ... | type: step|morph|structure -->` 마커

→ **harness-qa**: 수식·앵커 무손상 + 사실관계 검증.

## Phase 3: 인터랙티브 (interactive-viz-engineer) — v1.1 3B1B 전환

concept-explainer의 VIZ 마커를 입력으로:
- 단계형·전개형 → **3B1B 애니메이션 씬**(자체 경량 엔진, 부드러운 모핑, CDN 0)
- 파라미터형 → **시뮬레이션**(슬라이더·스테퍼)
- 과목 구조 내비 → **D3 탐색 트리**(CDN+폴백)
- 의생명 → **인터랙티브 논문 포스터**(DNABERT·Deep Learning·GAT)

→ **harness-qa**: 오프라인(CDN 0 확인)·다크모드 색·reduced-motion 폴백·콘솔에러·반응형 검증.

## Phase 4: 예상문제 (exam-designer) — v1.1 신규

concept-explainer 커버리지 확정 후, 과목 출제 특성대로:
- **객관식**(의료진단): 보기 + 정답 + **각 오답 해설**
- **서술형/증명**(AI개론·알고리즘·의생명): **모범답안 full-text** + 채점 포인트 + formatting
- 셀프테스트 토글 UI 보존, `_workspace/{과목}_exam.md` 커버리지 매핑

→ **harness-qa**: 토글 동작·수식 렌더·정답 정확성 검증.

## Phase 5: 워딩 마감 (persona-wordsmith)

콘텐츠·시각화·문제 확정 후 표면 워딩만:
- presentation-style DNA → 산문, 번역투·LLM투 제거
- 영어(한글) 병기 누락 보정(glossary 표준)
- `voice/tone.md` 동기화 + `log.md` 기록
- 수식·코드·id·앵커 무변경

→ **harness-qa**: 수식·앵커 무손상 최종 확인.

## Phase 5-P: Codex 병렬 트랙 (apple-to-apple, 옵션)

사용자가 비교를 요청하면, Claude 트랙(Phase 1~5)과 **병렬로** Codex가 동일 스펙 v1.1을 빌드한다.
- **실행:** `codex:codex-rescue` 에이전트(또는 codex:rescue 스킬)에 동일 요구사항 스펙을 전달. Bash 런타임으로 Codex가 작업.
- **산출 위치:** `codex-v1.1/`(기존 4 HTML 복사본을 그 디렉터리에서 개편). Claude 트랙은 루트 `*.html` 제자리 수정 → 한 브랜치에서 나란히 비교.
- **동일 입력 보장:** 같은 요구사항 4개 영역(설명 심화·3B1B 시각화·예상문제·영어 병기)을 그대로 전달. 평가 공정성을 위해 Claude 산출을 Codex에 보여주지 않는다(독립 빌드).
- **비교 산출물:** `_workspace/compare_v1.1.md` — 영역별 Claude vs Codex 차이(설명 깊이·시각화 실효성·문제 품질·오프라인 동작)를 표로. 사용자가 최종 판단.

## Phase 6: 통합 검증

1. harness-qa 전체 리포트 종합(`_workspace/qa_*.md`).
2. **`/code-review` 또는 adversarial 검증** — 내용 타당성·과잉설계·교차모델. QA(기술 동작)와 보완. 마무리 전 필수.
3. 수용 항목 반영 → 재검증.
4. 사용자 보고 + 피드백 요청(Phase 7 진화).

## 데이터 전달 프로토콜

- **파일 기반:** `_workspace/{phase}_{agent}_{artifact}` — 목차(`{과목}_toc.md`)·용어집(`{과목}_glossary.md`)·문항(`{과목}_exam.md`)·QA(`qa_{과목}.md`)·비교(`compare_v1.1.md`). 보존(감사 추적).
- **메시지 기반:** 에이전트 간 모듈 완성 알림·QA 회신·VIZ 마커 신호.
- 최종 산출: 루트 4개 HTML(제자리 Edit) + (옵션) `codex-v1.1/` 4개. `_workspace/`는 지우지 않는다.

## 에러 핸들링

- 에이전트 실패 시 1회 재시도 → 재실패하면 해당 산출물 없이 진행, 보고서에 **누락 명시**(빈 화면·dead link 방치 금지).
- 3B1B 씬은 CDN 0이라 오프라인 실패가 없어야 정상. reduced-motion은 폴백(오류 아님). D3 트리 CDN 실패는 정적 트리맵 폴백.
- 틀린 설명·틀린 정답은 치명 — 불확실하면 단정 금지, 근거 병기 + 검증 요청.
- 상충 정보는 삭제하지 않고 출처 병기 후 사용자 판단.
- id 변경이 참조처를 깨면 변경 롤백, 새 id 추가로 우회.
- Codex 트랙 실패는 Claude 트랙을 막지 않는다(병렬 독립). Codex 누락은 비교표에 명시.

## 테스트 시나리오

- **정상(전면):** "4과목 설명 심화·3B1B 시각화·예상문제·영어 병기 전면 개편" → Phase 0(초기) → 1~5 파이프라인 → (옵션 5-P Codex) → Phase 6 → 커밋.
- **부분 재실행:** "의생명 예상문제만 모범답안까지" → Phase 0(부분) → exam-designer만(+QA) → 보고.
- **부분 재실행:** "알고리즘 정렬을 3B1B 애니메이션으로" → interactive-viz-engineer만(3b1b-animation.md) → QA(오프라인·reduced-motion) → 보고.
- **Codex 비교:** "AI개론 v1.1을 Codex로도 만들어 비교" → Claude 트랙 + 5-P Codex(`codex-v1.1/ai-intro.html`) → `compare_v1.1.md`.
- **에러:** D3 CDN 차단 → 정적 트리맵 폴백 확인(빈 화면 아님). 3B1B 씬은 CDN 0이라 그대로 동작.

## 산출 후

- 변경 요약(과목별 추가 요소·대표 before/after)·QA 결과·검증 결과·(옵션)비교표 보고.
- "개선할 부분/팀 구성 바꿀 점 있나요?" 피드백 요청 → CLAUDE.md 변경 이력 갱신.
