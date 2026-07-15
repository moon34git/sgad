---
name: sgad
description: 'Story-Gated Agentic Development(SGAD) 진행 상태를 확인하고 다음 액션을 안내한다. BMAD Story 산출물과 Ouroboros Seed/Run을 잇는 Stage 1~5 워크플로우(SGAD.md)를 오케스트레이션하는 라우터 스킬. "sgad", "sgad 상태", "다음 뭐해야돼", "gate 돌려줘", "story 마무리해줘" 등에서 사용.'
---

# SGAD — Story-Gated Agentic Development 라우터

## Purpose

`bmad-help`가 BMAD 안에서, `ooo status`가 Ouroboros 안에서 하는 일을 SGAD 전체
(BMAD→Ouroboros 브릿지 + Guard 문서)에 대해 수행한다: 지금 Stage/Step이 어디인지
파악하고, 다음에 실행할 정확한 커맨드를 안내한다. 전체 방법론 원문은 이 저장소의
`SGAD.md` — Stage 정의, Seed YAML 스키마, 자주 하는 실수 등 라우팅을 넘어서는
질문은 그 문서를 직접 읽는다.

## Prerequisite Check

아래를 **이 순서 그대로** 확인한다. 뒤 항목까지 미리 확인해서 후보를 여러 개
나열하지 않는다 (Response Format의 "정확한 커맨드 하나" 원칙은 여기도 적용된다).
1·2번(BMAD/Ouroboros)은 없으면 사용자 승인을 받아 그 자리에서 설치까지 시도하고
재확인한다 — 승인 거부나 설치 실패일 때만 멈춘다. 3~5번은 설치가 아니라 Stage
산출물/문서라 없으면 항상 그 자리에서 멈추고 안내한다(대신 만들어주지 않는다).

> **설치 후 재확인의 함정:** 아래 1·2번에서 Bash로 설치 커맨드를 실행해도 **현재 세션의
> skill/plugin 목록은 즉시 갱신되지 않는다** — `/reload-plugins`는 슬래시 커맨드라 에이전트가
> 대신 실행할 수 없고, 반드시 사용자가 직접 입력해야 반영된다. 따라서 설치 직후 곧바로
> `/bmad-help`나 `ooo help`로 "재확인"을 시도하지 않는다 — 설치 커맨드 실행 후에는 **"설치
> 완료, `/reload-plugins` 실행해달라"고 요청하고 사용자가 그걸 실행한 뒤에만** 재확인한다.

1. **BMAD** — `/bmad-help`(또는 설치된 BMAD 버전의 상응 스킬)가 응답하는지 확인한다.
   응답이 있으면 "BMAD 확인됨"으로 통과. 없으면 프로젝트에 BMAD가 없다는 뜻이므로,
   `npx bmad-method install` 실행 여부를 사용자에게 먼저 물어보고 승인받은 뒤에만
   Bash로 실행한다 — 임의로 npm 설치를 실행하지 않는다. 설치 후 위 함정 안내대로
   사용자에게 `/reload-plugins` 실행을 요청하고, 실행됐다는 응답을 받은 뒤에 다시
   `/bmad-help`로 확인한다. 통과하면 다음 항목으로, 실패하면 에러를 그대로 보고하고 멈춘다.
2. **Ouroboros** — `ooo help`가 응답하는지 확인한다. 응답이 있으면 "Ouroboros 확인됨"으로
   통과 (플러그인은 프로젝트가 아니라 머신 전역 설치이므로, 다른 프로젝트에서 이미
   설치했다면 이 프로젝트에서도 이미 응답할 수 있다 — 정상이다). 응답이 없으면 아래
   두 커맨드를 실행할지 사용자에게 먼저 물어보고 승인받은 뒤에만 Bash로 실행한다 —
   SGAD 플러그인은 Ouroboros를 `dependencies`로 선언하지 않으므로(그러면 마켓플레이스
   미등록 시 SGAD 전체 로드가 실패해버림 — 실측 확인됨) 이 단계가 유일한 자동 설치
   경로다. 임의로 실행하지 않는다.
   ```bash
   claude plugin marketplace add Q00/ouroboros   # Ouroboros 배포처가 바뀌면 이 값도 달라질 수 있음
   claude plugin install ouroboros@ouroboros
   ```
   실행 후 위 함정 안내대로 사용자에게 `/reload-plugins` 실행을 요청하고, 실행됐다는
   응답을 받은 뒤에 다시 `ooo help`로 확인한다. 통과하면 다음 항목으로, 실패하면
   에러를 그대로 보고하고 멈춘다.
3. **Stage 1 산출물** — `sgad-config.yaml`이 아직 없으면 기본 경로
   `_bmad-output/planning-artifacts/epics.md`를, 있으면 그 안의 `epics_file` 경로를
   확인한다. 없으면 "Stage 1 미완료"로 안내하고 멈춘다 — BMAD로 Epics & Stories부터
   만들어야 한다 (Stage 2 Guard 문서를 먼저 채우라고 안내하지 않는다: Guard 문서는
   Stage 1 산출물의 ADR/Story Type을 전제로 하므로 순서를 바꾸면 다시 고쳐야 한다).
4. `docs/workflow/sgad-config.yaml` — 없으면 이 플러그인의 `templates/sgad-config.yaml`을
   (`${CLAUDE_PLUGIN_ROOT}/templates/sgad-config.yaml`) 복사해서 채우라고 안내하고 멈춘다.
5. `docs/workflow/definition-of-done.md`, `docs/workflow/workflow-metrics-log.md`,
   `docs/contracts/interface-data-contract-registry.md`, `docs/workflow/gate-project.md`
   — 없으면 `templates/`의 대응 파일을 복사해서 채우라고 안내하고 멈춘다 (Stage 2 미완료).

5개 전부 통과해야 아래 State Detection으로 넘어간다. 1·2번(BMAD/Ouroboros)이 이번 실행에서
막 설치됐든 이미 설치돼 있었든, 5개 모두 통과한 시점에 "Prerequisite Check 통과 — BMAD ✅
Ouroboros ✅ SGAD 문서 ✅, SGAD 준비 완료"처럼 짧게 한 번 보고하고 State Detection으로
넘어간다 — 매번 항목을 나열할 필요는 없고, 뭐가 이미 있었고 뭐가 새로 설치됐는지만
구분해서 보고한다.

## State Detection

`sgad-config.yaml`이 있다는 전제하에, 아래 순서로 읽어서 현재 위치를 판단한다.

1. `docs/workflow/workflow-metrics-log.md` — 마지막으로 기록된 Story와
   Run Result(Pass/Repaired/Failed)를 확인한다.
2. `docs/contracts/interface-data-contract-registry.md` — `Proposed` 상태인
   항목이 있는데 Metrics Log에 해당 Story 완료 기록이 없으면 그 Story는
   Stage 3 진행 중이다.
3. `seeds/*.yaml` (경로는 `sgad-config.yaml`의 `seeds_dir` 참조) — Metrics Log의
   마지막 완료 Story보다 최신인 Seed가 있고 그 Story의 Gate 기록이 없으면
   "Seed는 작성됐지만 아직 Run/Gate 전"이다.
4. `sgad-config.yaml`의 `epics_file`(기본 `_bmad-output/planning-artifacts/epics.md`) —
   **현재 Epic(1의 마지막 완료 Story가 속한 Epic) 안에서만** 다음 미착수 Story를 찾는다.
   다른 Epic의 Story는 여기서 찾지 않는다 — 그 Epic의 Stage 4/5(아래 5~7)가 끝나기
   전에 다음 Epic으로 새는 걸 막기 위함이다. 이 Epic 안에 다음 미착수 Story가 있으면
   그걸로 보고를 끝낸다.
5. 현재 Epic 안에 다음 미착수 Story가 없으면(Epic의 모든 Story가 Metrics Log에
   완료로 기록됨) 해당 버전의 "Stage 4" 섹션이 있는지 확인한다. 없으면 →
   "전체 Story 완료, Stage 4 대기"다.
6. "Stage 4" 섹션은 있는데 같은 버전의 "Stage 5" 섹션이 없으면 → "Stage 4 완료, Stage 5 대기"다
   (Stage 4 통과가 Stage 5를 대체하지 않는다 — SGAD.md 참고).
7. 해당 버전의 Stage 4/5 섹션이 둘 다 있으면 → "Stage 1~5 완료, 다음 Epic 대기 중"으로
   보고한다. 새 Story를 지어내지 않는다. `epics.md`에 다음 Epic의 Story가 이미 있으면
   Stage 1/2를 재실행하지 않고 바로 Stage 3 Step 1(Story 선택 및 Type 분류)로 돌아가
   그 Epic의 첫 Story를 다음 액션으로 안내한다 (Stage 2 Guard 문서는 append 방식이라
   이미 존재함).

## Response Format

해당하는 항목만 간결하게 보고한다.

- **현재 위치**: `Stage X, Step Y` (또는 "완료 — 다음 Epic 대기")
- **마지막 완료 Story**: Metrics Log 기준
- **다음 액션**: 정확한 커맨드 하나
  - Stage 1 미완료 → 설치된 BMAD 버전에서 Epics & Stories를 만드는 스킬 (`/bmad-help`로 확인)
  - Stage 2 문서 없음 → 위 Prerequisite Check로 안내
  - Story는 있는데 Seed 없음 → `ooo seed`
  - Seed는 있는데 Run 기록 없음 → `ooo run <seeds_dir>/story-X.X-name.yaml`
  - Run은 됐는데 Gate 기록 없음 → 아래 Gate를 바로 실행할지 제안
  - Gate 통과, Registry/Log 갱신 안 됨 → 지금 갱신할지 제안
  - 이 Epic/버전 전체 Story 완료, Stage 4 섹션 없음 → Stage 4(Cross-Story Integration
    Smoke) 지금 실행할지 제안 (SGAD.md Stage 4 "수행 방법" 참고)
  - Stage 4 섹션은 있는데 Stage 5 섹션 없음 → "Stage 4 통과, Stage 5(직접 실행 검증)는
    사람이 해야 함"으로 안내 — Stage 5는 대신 실행할 수 없다
  - 이 Epic/버전 Stage 4/5 둘 다 완료, 다음 Epic의 Story가 `epics.md`에 있음 →
    Stage 1/2 재실행 없이 그 Story의 Step 1(Type 분류)부터 안내

## Gate 실행 (Executable Gate — Layer 1 + Layer 1 확장 + Layer 2)

사용자가 "gate 돌려줘"라고 하거나 Run 직후, `docs/workflow/sgad-config.yaml`을 읽어
아래를 순서대로 실행한다. **하나라도 실패하면 조용히 넘어가지 말고 그 자리에서 보고를 멈춘다.**

```bash
# Layer 1 — Universal Gate (sgad-config.yaml의 값으로 치환)
python -m compileall <src_dirs> <test_dir> -q
python -m py_compile <entry_point>
<entry_point_import_check>          # 예: (cd <entry_point_dir> && python -c "import <module>")
<test_command>                      # 예: pytest <test_dir> -q
```

Backend/Integration Story는 여기까지로 `definition-of-done.md`의 해당 Type 항목이
전부 커버된다 — 아래 Layer 1 확장은 스킵하고 바로 Layer 2로 넘어간다.

**Layer 1 확장 — Story Type Gate.** 이 Story의 Seed(`seeds/*.yaml`)에서
`metadata.story_type`을 읽는다.

```bash
# story_type: UI 일 때만 (sgad-config.yaml의 ui_smoke_command / ui_panels)
<ui_smoke_command>
python -c "import <ui_panels의 각 모듈>"
```

`story_type`이 UI인데 `ui_smoke_command`/`ui_panels`가 둘 다 비어있으면 조용히
스킵하지 않는다 — "UI Story인데 sgad-config.yaml에 설정 없음"으로 멈추고 채우라고
안내한다.

`story_type`이 Documentation이면, `docs/workflow/gate-project.md`에 이 Story
전용 PG 항목(PG-04 예시 참고: 문서가 언급하는 파일/함수 존재 확인, 코드 예시
실행)이 있는지 확인하고 실행한다. 없으면 "Documentation Story인데 대응 PG
항목 없음"으로 멈춘다 — 문서-구현 일치를 확인 없이 통과시키지 않는다.

```bash
# Layer 2 — Project Gate (docs/workflow/gate-project.md에 정의된 PG-XX 항목을 그대로 실행)
# 각 PG 항목은 gate-project.md의 "명령어" 블록을 그대로 복사해서 실행한다.
# grep -R은 결과 없으면 exit 1을 반환하므로 반드시 if/else로 감싼다 (SGAD.md 함정 참고).
```

> **결과가 FAIL로 나오면 매칭된 줄을 먼저 읽는다.** 실제 위반 코드인지, 아니면
> "~를 import하면 안 됨" 같은 규칙 설명 주석/docstring 자체가 걸린 false positive인지
> 구분한다. 후자라면 `gate-project.md`의 해당 PG 명령어가 import 문/실제 호출에
> 앵커링되지 않은 것 — 패턴을 고쳐야지 Gate를 무시하고 넘어가지 않는다.

`docs/workflow/gate-project.md`가 아직 비어있거나 PG 항목이 하나도 없으면, Layer 2는
"정의된 Project Gate 없음 — Stage 2에서 채워야 함"이라고 보고하고 넘어가지 않는다
(무조건 통과로 취급하지 않는다).

## Registry / Metrics Log 갱신 (Gate 통과 후)

**Gate → Registry → Metrics Log 순서를 지킨다 — Gate 실패 상태에서는 절대 갱신하지 않는다.**

1. `docs/contracts/interface-data-contract-registry.md` — 이 Story가 만든
   `Proposed` 항목을 `Active`로 전환, 새 인터페이스는 신규 항목으로 추가.
2. `docs/workflow/workflow-metrics-log.md` — `## Story X.X — [제목]` 블록 추가
   (Story Type / Run Result / Repair Count / Unit Test / Regression /
   Type-Specific Check / Project ADR Checks / I/F Contract / Notes).
   Type-Specific Check는 Backend/Integration이면 N/A, UI/Documentation이면
   위 Layer 1 확장 결과를 기록한다.
3. Registry 갱신 없이 Metrics Log만 먼저 쓰지 않는다 (Proposed→Active 누락 방지).

## Constraints

- `ooo evaluate`의 긍정적 판정만으로 Story Done을 선언하지 않는다 — 위
  Executable Gate 통과만이 유일한 기준이다 (SGAD Evaluation Policy).
- 현재 Story의 Gate→Registry→Log가 전부 끝나기 전에 다음 Story로 넘어가지 않는다.
- Stage 1/2 산출물이 아직 없으면 그렇다고 말하고 생성부터 안내한다 — 없는 Story를
  지어내지 않는다.
- Stage 5(또는 Stage 4)에서 이슈가 여러 개 한 번에 나오면, 이슈마다 Story를 Stage 3에
  태울 때마다 Epic의 Stage 4/5를 즉시 재실행하도록 제안하지 않는다 — 그 배치의 Story가
  전부 Stage 3를 통과한 뒤 Stage 4/5를 한 번만 재실행하도록 안내한다 (SGAD.md 참고).
- Story 수가 적을수록 Guard 문서 3종의 ROI가 낮아진다 — 아주 소규모 범위라면
  SGAD 전체를 강제하기보다 이 사실을 사용자에게 알리고 계속할지 확인한다
  (SGAD.md 한계 섹션). 정확한 개수 기준은 없다 — 판단 근거가 없는 숫자를
  임계값처럼 쓰지 않는다.
- BMAD/Ouroboros의 실제 커맨드/스킬 이름은 설치 버전에 따라 다르다 — 하드코딩된
  이름을 가정하지 말고 `/bmad-help` / `ooo help`로 실제 사용 가능한 인터페이스를 확인한다.
