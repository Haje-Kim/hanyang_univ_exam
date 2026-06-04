---
name: study-hub-builder
description: 한양대 AI융합대학원 기말 학습 허브(4과목 HTML)의 콘텐츠를 제작·개선하는 오케스트레이터. 목차 정리·선행지식·D3 탐색트리·개념 시뮬레이션·인터랙티브 논문포스터·페르소나 워딩을 전문 에이전트 팀으로 조율한다. 학습자료/학습허브/과목 페이지를 만들거나 고칠 때, "탐색 트리·시뮬레이션·선행지식·논문 포스터·워딩 바꿔·목차 정리·다시 실행·재실행·업데이트·보완·이 과목만 다시" 류 요청에 반드시 사용.
---

# Study Hub Builder — 학습 허브 제작 오케스트레이터

4과목 학습 허브(`ai-intro.html` 인공지능개론 · `algorithms.html` 컴퓨터 알고리즘 · `biomed-ai.html` 의생명 AI · `medical-dx-ai.html` 의료진단 AI)를 전문 에이전트 팀으로 제작·개선한다.

## 팀 구성 (실행 모드: 하이브리드)

| 에이전트 | 역할 | 모드 |
|---|---|---|
| content-architect | 목차 정리 + 선행지식 파트 | 과목별 병렬 |
| interactive-viz-engineer ⭐ | D3 탐색트리 · 시뮬레이션 · 논문포스터 | 과목별 병렬 |
| persona-wordsmith | 워딩 재작성 + voice/tone.md | 구조 확정 후 |
| harness-qa | 오프라인·렌더·앵커·반응형 점진 검증 | 모듈 완성 직후 |

> 모든 Agent 호출에 `model: "opus"` 명시. 과목 단위는 독립적이라 병렬화에 적합하고, 과목 내부는 구조→시각화→워딩 파이프라인 순서 의존이 있다.

## Phase 0: 컨텍스트 확인 (먼저)

1. `_workspace/` 존재 여부 확인.
   - 없음 → **초기 실행** (전체 파이프라인)
   - 있음 + 사용자가 부분 수정 요청("이 과목만", "논문포스터만") → **부분 재실행** (해당 에이전트만 재호출, 나머지 산출물 재사용)
   - 있음 + 새 전면 요청 → 기존 `_workspace/`를 `_workspace_prev/`로 이동 후 새 실행
2. `voice/tone.md`(`11_wiki/haje/voice/`) 상태 확인 — placeholder면 persona-wordsmith가 작성 포함.
3. 변경 범위를 사용자에게 한 줄 확인(모호할 때만).

## Phase 1: 구조 (과목별 병렬, content-architect)

각 과목에 대해 content-architect 호출:
- 목차 재점검·정리 → `_workspace/{과목}_toc.md`
- 선행지식 섹션 신설(본문 진입 전)
- id 안전 규칙 준수(기존 앵커 보존)

→ **harness-qa**가 즉시 앵커 정합성 검증(toc id ↔ 본문 섹션 id).

## Phase 2: 인터랙티브 (과목별 병렬, interactive-viz-engineer)

`_workspace/{과목}_toc.md`를 입력으로:
- 정적 트리맵·로드맵 → **D3 collapsible 탐색 트리**(CDN+폴백)
- 단계형 개념 → **시뮬레이션**(정렬/DP/CSP/탐색/GNN/ROC)
- 의생명 → **인터랙티브 논문 포스터 3배 심화**(7모듈 × 논문 3편)

→ **harness-qa**가 모듈 완성 직후 렌더·오프라인·콘솔에러 검증.

## Phase 3: 워딩 (persona-wordsmith)

구조·시각화 확정 후 전 과목 워딩 재작성:
- presentation-style DNA → 산문 변환 적용
- `voice/tone.md` 정식 작성 + `log.md` 기록
- 수식·코드·id·앵커 무변경

→ **harness-qa**가 수식·앵커 무손상 최종 확인.

## Phase 4: 통합 검증

1. harness-qa 전체 리포트 종합(`_workspace/qa_*.md`).
2. **`/adversarial-review` 실행** — 내용 타당성·과잉설계·교차모델(CODEX) 검증. QA(기술 동작)와 보완 관계. 작업 마무리 전 필수.
3. adversarial-review 수용 항목 반영 → 재검증.
4. 사용자 보고 + 피드백 요청(Phase 7 진화).

## 데이터 전달 프로토콜

- **파일 기반:** `_workspace/{phase}_{agent}_{artifact}` — 목차 트리·QA 리포트 등 중간 산출물. 보존(감사 추적).
- **메시지 기반:** 에이전트 간 모듈 완성 알림·QA 회신.
- 최종 산출물은 4개 HTML(제자리 Edit). `_workspace/`는 지우지 않는다.

## 에러 핸들링

- 에이전트 실패 시 1회 재시도 → 재실패하면 해당 산출물 없이 진행하고 보고서에 **누락 명시**(빈 화면·dead link 방치 금지).
- D3/CDN 실패는 폴백으로 처리(오류 아님). 폴백조차 깨지면 QA가 차단 보고.
- 상충 정보(예: 용어 정의 불일치)는 삭제하지 않고 출처 병기 후 사용자 판단에 맡김.
- id 변경이 참조처를 깨면 변경 자체를 롤백하고 새 id 추가 방식으로 우회.

## 테스트 시나리오

- **정상 흐름:** "4과목 탐색트리·선행지식·워딩 전면 개편" → Phase 0(초기) → 1~3 파이프라인 → Phase 4 adversarial-review → 커밋.
- **부분 재실행:** "의생명 논문포스터만 더 늘려" → Phase 0(부분) → interactive-viz-engineer만 재호출(paper-poster.md) → QA → 보고.
- **에러 흐름:** D3 CDN 차단 환경 → 폴백 정적 트리맵 노출 확인(빈 화면 아님) → 통과.

## 산출 후

- 변경 요약(과목별 추가 요소·before/after 대표)·QA 결과·adversarial-review 결과 보고.
- "개선할 부분/팀 구성 바꿀 점 있나요?" 피드백 요청 → CLAUDE.md 변경 이력 갱신.
