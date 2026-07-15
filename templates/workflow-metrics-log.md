# Workflow Metrics Log

Story별 결과 기록. `sgad` 스킬의 State Detection이 "마지막으로 완료된 Story"를
판단하는 근거로 이 파일을 읽는다 — 새 Story를 끝낼 때마다 반드시 갱신한다.

---

<!-- Story마다 아래 블록을 추가한다 -->

## Story X.X — [Story 제목]

| 항목 | 결과 |
|---|---|
| Story Type | Backend / UI / Integration / Documentation |
| Run Result | Pass / Repaired / Failed |
| Repair Count | 0 |
| Unit Test | Pass (N passed) |
| Regression | Pass |
| Type-Specific Check | Pass / N/A (UI smoke·panel import 또는 Documentation PG 결과, Backend/Integration은 N/A) |
| Project ADR Checks | Pass |
| I/F Contract | Pass |
| Notes | — |

---

<!-- Epic/버전의 모든 Story가 끝나면 Stage 4/5 섹션을 추가한다 -->

## Stage 4 — Cross-Story Integration Smoke (자동) — v[X]

검증한 플로우와 발견 사항을 기록.

## Stage 5 — Validate (사람) — v[X]

직접 실행 검증 결과와 발견된 문제, 처리 방법을 기록.
