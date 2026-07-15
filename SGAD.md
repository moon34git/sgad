# SGAD — Story-Gated Agentic Development

> BMAD로 설계하고, Ouroboros로 구현하되,
> Story 단위 Gate를 통과해야만 다음으로 넘어가는
> Controlled AI-assisted Development Methodology.

---

## 전제 조건

SGAD를 시작하기 전에 아래가 갖춰져 있어야 한다.

| 항목 | 확인 방법 |
|---|---|
| Claude Code (CLI) | `claude --version` |
| BMAD | `/bmad-help` 입력 시 응답 있음. `npx bmad-method install`로 설치하는 별도 npm 패키지. |
| Ouroboros | `ooo help` 입력 시 응답 있음. Claude Code 플러그인. 없어도 SGAD 설치/로드를 막지 않는다 — `plugin.json`은 Ouroboros를 `dependencies`로 선언하지 않는다(Claude Code의 플러그인 dependency는 대상 마켓플레이스가 미리 등록돼 있을 때만 작동하고, 아니면 로드 자체가 조용히 실패해 오히려 더 깨진다는 게 실측으로 확인됨). 대신 `/sgad` 최초 실행 시 스킬이 직접 감지해서 설치 여부를 물어본다. |
| SGAD 플러그인 | `/plugin install sgad@sgad`로 설치, `docs/workflow/sgad-config.yaml` 작성 완료 |

BMAD와 Ouroboros는 이 저장소에 포함되어 있지 않다 — 각자의 설치 방법대로 별도 설치한다
(둘 다 `/sgad` 최초 실행 시 스킬이 감지해서 설치 여부를 물어봄). 플러그인은 프로젝트가
아니라 Claude Code가 실행되는 머신 전역에 설치되므로, 다른 프로젝트에서 이미 설치한
적이 있다면 새 프로젝트에서도 이미 응답할 수 있다 — 방금 막 설치된 것과 혼동하지 않는다.
SGAD는 그 둘을 잇는 얇은 오케스트레이션/거버넌스 레이어일 뿐, 둘을 대체하지 않는다.

---

## 왜 쓰는가

AI 코딩 도구를 그냥 쓰면 반복적으로 같은 문제가 생긴다.

- 요구사항을 한 번에 넣으면 AI가 여러 모듈을 동시에 수정하다 구조를 깨뜨린다
- Story 간 함수명, 인자, 반환 타입이 후속 구현에서 조용히 달라진다
- "이 Story는 성공"이지만 이전 Story와 함께 돌리면 깨진다
- AI가 무엇을 얼마나 잘 했는지 측정할 방법이 없다

SGAD는 이 네 가지 문제를 **구조적으로 막기 위한 방법론**이다.

---

## 핵심 원칙

```
1 BMAD Story = 1 Ouroboros Seed = 1 Run = 1 Gate = 1 Metrics Record

Local Success + Global Compatibility = Story Done
```

해당 Story만 통과하는 것은 충분하지 않다.
이전 Story들과 함께 동작해야 Gate를 통과한다.

---

## 방법론 출처

| Stage | 방법론 | 설명 |
|---|---|---|
| Stage 1 | **BMAD** | Brief → PRD → Architecture → UX → Epics & Stories. BMAD 에이전트가 설계 산출물을 생성한다. |
| Stage 2 | **BMAD** | Guard 문서(DoD, Registry, Metrics Log)는 BMAD의 Governance 레이어다. 구현 전 완료 기준과 인터페이스 계약을 선제적으로 정의한다. |
| Stage 3 | **Ouroboros** | Seed → QA → Run → Gate 루프는 Ouroboros의 Spec-First 실행 방법론을 차용한다. `ooo seed` / `ooo run` 커맨드로 실행된다. 단, Ouroboros의 `ooo interview` 단계는 생략한다 — BMAD의 Epic/Story/AC가 인터뷰 역할을 대신하며, 각 Story가 Seed의 입력으로 직접 사용된다. |
| Stage 4 | **자동 (AppTest 등)** | 한 Epic/버전의 모든 Story가 끝난 뒤, 여러 Story의 기능을 하나의 이어진 세션으로 자동 구동해 cross-story 통합 문제를 잡는 단계. 여전히 기계적 검증 — 사람 판단 아님. |
| Stage 5 | **직접 실행 (사람)** | 한 Epic/버전의 Stage 4 통과 후, 앱을 직접 실행해보고 눈으로 확인하는 단계 (Stage 4와 같은 Epic 단위, 프로젝트 끝까지 미루지 않음). 별도 방법론 없음. |

---

## 전체 흐름

```
[Stage 1] BMAD — 설계 원본 고정
    Product Brief → PRD → Architecture → UX → Epics & Stories
                                    ↓
[Stage 2] BMAD Guard — Governance 문서 3종 생성
    DoD / Registry / Metrics Log
                                    ↓
[Stage 3] Ouroboros Gate — Story 반복 루프 (구현 단계, Epic의 Story 수만큼 반복)
    ┌─────────────────────────────────────────────────────┐
    │ Story 선택+Type 분류 → Seed 작성 → Seed QA → Run     │
    │ → Executable Gate(Layer 1 → Layer 1 확장* → Layer 2) │
    │ → Registry 업데이트 → Metrics Log 업데이트 → 다음 Story│
    └─────────────────────────────────────────────────────┘
                       ↓ (이 Epic의 Story 전부 완료)
[Stage 4] Cross-Story Integration Smoke — 자동, Epic/버전 단위
    ┌─────────────────────────────────────────────────────┐
    │ 이번 Epic의 새 플로우(+의존하는 이전 Epic 코드) 자동 구동│
    │ → 문제 발견 → 분류 → Fix/Change Story → Stage 3 재진입│
    └─────────────────────────────────────────────────────┘
                       ↓ (Stage 4 통과, Stage 5는 대체 불가)
[Stage 5] Validate — 직접 실행해보기 (사람, Epic/버전 단위)
    ┌─────────────────────────────────────────────────────┐
    │ 누적 전체 플로우 직접 실행 → 문제 발견 → 분류         │
    │ → Fix/Change Story → Stage 3 Step 1으로 재진입       │
    └─────────────────────────────────────────────────────┘
                       ↓ (문제 없음)
    다음 Epic 있으면 Stage 1/2 재실행 없이 Stage 3 Step 1로
    (Stage 2 Guard 문서는 append 방식이라 이미 존재함)
```

\* Layer 1 확장은 이 Story의 `story_type`이 UI/Documentation일 때만 적용된다.

---

## Stage 1 — Spec (BMAD Layer)

설계 원본을 만들고 baseline으로 고정한다.
이후 구현 단계에서 임의로 변경하지 않는 Single Source of Truth다.

### 1-1. BMAD 설계 산출물 생성

Claude Code에서 설치된 BMAD 버전이 제공하는 스킬로 아래 순서를 실행한다.

> **주의:** 실제 커맨드/스킬 이름은 설치된 BMAD 버전에 따라 다르다.
> `/bmad-help` 또는 `/help`에서 사용 가능한 BMAD 스킬 목록을 먼저 확인한다.

```
Product Brief 작성
PRD 작성
Architecture + ADR 작성
UX/UI 설계 (UI Story가 있을 경우)
Epics 작성
각 Epic의 Stories + AC 작성
```

### 1-2. 산출물 저장 위치 (BMAD 기본 규칙을 따른다 — 예시)

```
_bmad-output/planning-artifacts/
  ├── prd.md
  ├── architecture.md          ← ADR 포함 (ADR-01, ADR-02 ...)
  ├── ux-design.md
  ├── epics.md
  └── stories/
        ├── story-1.1-*.md
        ├── story-1.2-*.md
        └── ...
```

### 1-3. Baseline 고정 체크리스트

- [ ] 모든 Story에 Acceptance Criteria가 명시되어 있다
- [ ] Architecture에 ADR이 포함되어 있다 (컬럼 스키마, 캐시 전략, 모듈 경계 등)
- [ ] Epic 간 의존 관계가 정리되어 있다
- [ ] **각 Epic이 끝났을 때 실행 가능한 증분(수직 슬라이스)이 된다** — Stage 5(사람이 직접
      실행)가 검증할 유저 플로우가 그 시점에 존재해야 한다. "데이터 레이어만", "API만" 같은
      수평 슬라이스로 Epic을 나누면 Epic이 끝나도 사람이 만져볼 게 없어 Stage 5가 성립하지
      않는다. 정말 독립적으로 슬라이싱이 안 되는 예외(순수 인프라 Epic 등)는 Stage 5 섹션의
      예외 규정을 따른다.

**Stage 1 완료 후 설계 파일은 구현 중 임의로 수정하지 않는다.**
요구사항 변경이 필요하면 먼저 BMAD Story 또는 Architecture Decision을 명시적으로 갱신하고,
해당 변경을 반영한 Seed를 다시 작성한다. 변경 금지가 아니라 **통제된 변경**이다.

---

## Stage 2 — Guard (Governance Layer)

구현 전에 완료 기준·호환성 기준·평가 기준을 세운다.
**Guard 없이 구현에 진입하면 SGAD가 아니다.**

### 2-1. 생성할 문서

```
docs/
  workflow/
    definition-of-done.md          ← Story Type별 완료 판정 기준
    workflow-metrics-log.md        ← Story별 품질 점수 기록
    sgad-config.yaml               ← Gate 명령어가 참조할 프로젝트별 경로 설정
    gate-project.md                ← Layer 2 Project Gate 정의 (프로젝트 ADR 기준)
  contracts/
    interface-data-contract-registry.md  ← 함수/데이터 계약 등록
```

이 저장소의 `templates/`에 위 문서들의 빈 템플릿이 있다 — 새 프로젝트에 그대로 복사해서 채운다.

### 2-2. Definition of Done

Story Type별로 다른 완료 기준을 적용한다.

| Type | 핵심 DoD |
|---|---|
| Backend | unit test, compile check, import check, Project ADR 위반 없음 |
| UI | + UI 프레임워크 smoke test, 화면/패널 import 가능 |
| Integration | + 전체 테스트 스위트, 전체 regression |
| Documentation | 문서-구현 일치 확인 |

### 2-3. Interface/Data Contract Registry

Story 간 호환성을 유지하는 핵심 문서.
AI가 후속 Story에서 함수 시그니처를 임의로 바꾸는 것을 통제한다.

**상태 흐름:**
```
Seed 작성 시점  → Proposed  (예상 interface 등록)
Story 완료 시점 → Active    (실제 구현 기준 확정)
변경 발생 시    → Changed   (Consumer Story에 migration 필요)
사용 중단 시    → Deprecated
```

**Registry 항목 형식:**
```markdown
### IF-001

| 항목 | 값 |
|---|---|
| Module | (모듈 경로) |
| Function | (함수명) |
| Signature | `func(arg: type) -> ReturnType` |
| Returns | (계약 요약) |
| Producer | Story X.X |
| Consumers | Story Y.Y, Story Z.x |
| Status | Active |
| Changed | — |
```

### 2-4. Workflow Metrics Log

Story별 결과를 기록한다. 단일 총점보다 항목별 실행 결과를 우선 기록한다.

```markdown
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
```

### 2-5. sgad-config.yaml

Gate 명령어(Stage 3 Step 5)가 프로젝트마다 다른 경로/모듈 구조를 참조할 수 있도록,
프로젝트별 설정을 이 파일 하나에 모은다. 형식은 `templates/sgad-config.yaml` 참고.

---

## Stage 3 — Gate (Story 반복 루프)

Stage 1+2가 완료된 뒤 각 Story를 아래 순서로 반복한다.

---

### Step 1. Story 선택 및 Type 분류

`epics.md` / `stories/`에서 다음 Story를 고른다. **같은 Epic 안에서만** 고른다 —
현재 Epic의 Story가 다 끝나기 전에 다른 Epic의 Story로 건너뛰지 않는다. 현재
Epic의 Story가 모두 끝났으면 Step 1로 돌아오지 않고 Stage 4로 넘어간다(Stage 4/5
를 거치지 않고 바로 다음 Epic Story를 고르면 Epic 단위 검증이 통째로 스킵된다).

```
Type 분류 기준:
  Backend      → 계산 로직, 데이터 처리, API
  UI           → 화면, 위젯
  Integration  → 두 모듈 이상 연결, end-to-end 흐름
  Documentation → 문서 업데이트만
```

이 분류는 Step 2의 Seed `metadata.story_type`에 그대로 기록한다 — Step 5
Executable Gate가 이 값을 읽어 Type별로 다른 검증을 추가로 실행한다
(definition-of-done.md의 Story Type별 항목과 1:1 대응).

---

### Step 2. Seed 작성

> **Ouroboros interview 생략:** 일반 Ouroboros 워크플로우는 `ooo interview`로 요구사항을 도출하지만,
> SGAD에서는 BMAD Stage 1에서 이미 Story/AC가 완성되어 있으므로 인터뷰를 건너뛴다.
> `epics.md`의 해당 Story가 Seed의 입력으로 직접 사용된다.

Claude Code에서 아래를 입력한다.

```
ooo seed
```

Ouroboros가 Story 명세를 읽고 Seed YAML 초안을 자동 생성한다.
저장 위치: `seeds/story-X.X-[name].yaml`

**Seed 필수 포함 항목:**

```yaml
metadata:
  seed_id: seed_story_X_X_name
  version: 1.0.0
  story: "Story X.X: 제목"
  story_type: Backend | UI | Integration | Documentation  # Step 1 분류 결과

goal: >
  한 문장으로 이 Story가 달성할 것

constraints:
  - "ADR 번호와 함께 구체적인 제약 조건"
  - "변경 금지 파일 목록"
  - "Brownfield / Greenfield 구분"

acceptance_criteria:
  - >
    Given [전제 조건],
    when [행위],
    then [검증 가능한 결과].

ontology_schema:
  name: SchemaName                     # 이 Story가 다루는 도메인 모델 이름 (예: TodoCliStore)
  description: 한 문장으로 이 도메인 모델이 뭘 표현하는지
  fields:
    - name: function_name
      type: function
      attribution: greenfield | brownfield

evaluation_principles:
  - name: Correctness
    weight: 0.40
    description: ...

exit_conditions:
  - name: RegressionPass
    description: 모든 기존 테스트 통과
    criteria: "<프로젝트 테스트 커맨드> exits 0"
```

> **중요 (Ouroboros seed parser 기준):** `acceptance_criteria`는 flat `- >` 문자열 형식이어야 한다.
> dict 형식(`- id: AC-1`) 사용 시 Seed 검증에서 실패할 수 있다. Ouroboros 버전이 바뀌면 이 정책도 달라질 수 있다.

> **중요 (Ouroboros seed parser 기준):** `ontology_schema`는 `fields`만으로는 검증을 통과하지
> 못한다 — 최상위 `name`(도메인 모델 이름)과 `description`(한 문장 요약)이 반드시 있어야 한다.
> 빠뜨리면 "Seed 검증 실패"로 멈춘다. Ouroboros 버전이 바뀌면 이 정책도 달라질 수 있다.

**AC 작성 시 검증 범위 판단 기준 (동일 코드 경로 vs 실제 분기):**

> Ouroboros는 Seed의 각 AC를 Discover → Define → atomicity 체크(단일 관심사·1~2 파일 여부) →
> (non-atomic이면) 2~5개 child AC로 재귀 분해하는 구조로 실행한다.
> AC를 쓸 때는 "이 값이 코드 동작의 분기를 만드는가?"를 먼저 판단한다:
>
> - **분기를 만들지 않는 파라미터** → 대표값 1개를 깊게 검증하는 AC 하나 + 나머지 값들은
>   "동일 경로이므로 존재/카운트만 확인" 수준의 가벼운 AC 하나로 충분하다.
> - **분기를 만드는 파라미터** (값에 따라 실제로 다른 코드 경로를 실행하는 경우) →
>   분기 클래스별로 최소 1개씩은 반드시 AC에 포함한다.

---

### Step 3. Seed QA

Seed 작성 직후 QA를 실행한다. `ooo seed` 내부에서 자동 진행되며,
ambiguity_score가 임계값 이하가 될 때까지 반복한다.

| Score | Verdict | 처리 |
|---|---|---|
| ≥ 0.90 | PASS | Run 진입 가능 |
| 0.75–0.89 | REVISE | 이슈 수정 후 재평가 |
| < 0.75 | REWORK | Seed 전면 재작성 |

QA가 REVISE를 반환하면 지적된 이슈를 수정하고 동일 session_id로 재평가한다.
PASS 전까지 `ooo run`을 실행하지 않는다.

---

### Step 4. Run

```
ooo run seeds/story-X.X-[name].yaml
```

Ouroboros executor가 Seed의 AC를 순서대로 구현한다.

**실행 중 rate limit으로 중단된 경우:**
이미 생성된 파일은 유지된다. Gate를 먼저 실행해서 어디까지 됐는지 확인한 뒤
미완성 AC만 수동으로 보완하거나 Run을 재실행한다.

---

### Step 5. Executable Gate

Run 완료 후 Gate를 순서대로 실행한다. **모두 통과해야 다음 Step으로 넘어간다.**

```
Layer 1        — Universal Gate   : 모든 SGAD 프로젝트에 공통 적용
Layer 1 확장   — Story Type Gate  : Seed의 story_type이 UI/Documentation일 때만 추가 적용
Layer 2        — Project Gate     : 프로젝트 ADR / 아키텍처 규칙에 맞게 정의 (gate-project.md + sgad-config.yaml)
```

#### Layer 1 — Universal Gate

`sgad-config.yaml`의 `entry_point` / `src_dirs` / `test_dir`를 참조해 실행한다.

```bash
python -m compileall <src_dirs> -q
python -m py_compile <entry_point>
python -c "import <entry_point 모듈명>"
<프로젝트 테스트 커맨드> (예: pytest <test_dir> -q)
```

> `grep`은 결과가 없으면 exit code 1을 반환한다. if/else로 감싸서
> "결과 없음 = 통과"를 "명령 실패"로 오해하지 않게 한다.

Backend/Integration Story는 여기까지로 definition-of-done.md의 해당 Type 항목이
전부 커버된다 — 아래 확장/Layer 2로 넘어간다.

#### Layer 1 확장 — Story Type Gate

Seed의 `metadata.story_type`을 읽어 아래 경우에만 추가로 실행한다. Backend/Integration이면
스킵하고 바로 Layer 2로 넘어간다.

**story_type: UI** — `sgad-config.yaml`의 `ui_smoke_command` / `ui_panels`를 실행한다.

```bash
<ui_smoke_command>          # 엔트리 포인트를 실제 구동해 예외 없이 뜨는지 확인
python -c "import <ui_panels의 각 모듈>"   # 화면/패널 단위 import 체크
```

`ui_smoke_command`와 `ui_panels`가 둘 다 비어있으면 조용히 스킵하지 않는다 —
"UI Story인데 sgad-config.yaml에 ui_smoke_command/ui_panels 설정 없음"으로
멈추고 채우라고 안내한다.

**story_type: Documentation** — 범용 자동화가 불가능하므로, 이 Story 전용 PG
항목이 `gate-project.md`에 있는지 확인하고 그 항목을 실행한다 (`gate-project.md`
템플릿의 PG-04 예시 참고). 대응 PG 항목이 없으면 "Documentation Story인데 대응
PG 항목 없음"으로 멈춘다 — 문서-구현 일치를 확인 없이 통과시키지 않는다.

#### Layer 2 — Project Gate

`docs/workflow/gate-project.md`에 프로젝트별로 정의한다. 형식은 `templates/gate-project.md` 참고.

```markdown
## Project Gate — [프로젝트명]

### PG-01. [체크 이름]
- 대상: [파일 또는 디렉토리]
- 규칙: [금지/허용 패턴]
- 명령어:
  if grep -R "<pattern>" <target>/; then
    echo "FAIL: <reason>"
    exit 1
  else
    echo "PASS"
  fi
- 근거 ADR: ADR-XX
```

**Gate 실패 시 처리:**

| 실패 항목 | 처리 방법 |
|---|---|
| compile check | 문법 오류 직접 수정 |
| 테스트 실패 | 실패 테스트 원인 분석 후 코드 또는 테스트 수정 |
| Project Gate 위반 | 해당 파일에서 위반 패턴 제거 |

수동으로 Repair한 경우 Metrics Log에 Repair Count를 기록한다.

---

### Step 6. Registry 업데이트

Gate 통과 후 `docs/contracts/interface-data-contract-registry.md`를 업데이트한다.

```
Seed 작성 시 등록한 Proposed 항목 → Active로 전환
새로 생긴 함수 / 클래스 / 데이터 계약 → 신규 항목으로 추가
기존 계약 위반 발생 시 → Status: Changed, 영향받는 Consumer Story 명시
```

---

### Step 7. Metrics Log 업데이트

`docs/workflow/workflow-metrics-log.md`에 해당 Story 결과를 기록한다.

---

### Step 8. 다음 Story로 반복

Step 1로 돌아가 다음 Story를 선택한다.

---

## Story Completion Checklist

| 항목 | 의미 |
|---|---|
| AC 충족 | Story Acceptance Criteria 전체 통과 여부 |
| Unit Test | 해당 Story 테스트 100% 통과 여부 |
| Regression | 이전 Story 전체 테스트 통과 여부 |
| Smoke Test | entry point import 성공 여부 |
| Type-Specific Check | story_type=UI/Documentation일 때 Layer 1 확장 통과 여부 (Backend/Integration은 N/A) |
| I/F Contract | Interface Registry 위반 0건 |
| Data Contract | Data Contract Registry 위반 0건 |
| Architecture | Project ADR 위반 0건 |
| Registry 업데이트 | Proposed → Active 전환 완료 |
| Metrics Log 업데이트 | 결과 기록 완료 |

---

## 자주 하는 실수

### Seed에 AC를 dict 형식으로 쓴 경우

```yaml
# 잘못된 형식 — Run 시 실패
acceptance_criteria:
  - id: AC-1
    given: "..."
    when: "..."
    then: "..."

# 올바른 형식
acceptance_criteria:
  - >
    Given ..., when ..., then ...
```

### patch() 대상 모듈을 잘못 지정한 경우 (Python 예시)

```python
# 잘못된 예 — 소스 모듈을 patch
patch('module.original.func', ...)

# 올바른 예 — import한 모듈을 patch
# (caller.py에서 `from module.original import func` 했다면)
patch('caller.func', ...)
```

### Gate 대상 파일을 전체 파일 단위로 검사한 경우 (false positive)

UI 프레임워크의 캐시 데코레이터 등, 정당한 이유로 특정 import를 쓰는 파일이 있을 수 있다.
파일 전체가 아니라 문제되는 함수 범위로 grep을 한정한다.

```bash
# 잘못된 체크 (false positive 발생 가능)
grep "<pattern>" <file>

# 올바른 체크 (함수 범위 한정)
grep -A 50 "def <function_name>" <file> | grep "<forbidden_call>"
```

### 규칙을 설명하는 주석/docstring 자체가 매칭되는 경우 (false positive)

파일 상단 docstring이나 주석에 "이 모듈은 streamlit을 import하면 안 됨(ADR-06)" 같은
규칙 설명을 그대로 적어두는 경우가 흔하다. 이때 `grep "streamlit"`처럼 키워드만 찾는
체크는 실제 위반이 전혀 없어도 그 설명 문장 자체에 걸려 매번 FAIL을 보고한다 — 실제로
SGAD 초기 버전에서 `grep -R "streamlit" charts/ drift/`가 "ADR-06: streamlit import
금지"라는 주석 한 줄 때문에 정상 코드를 위반으로 오탐한 사례가 있었다.

```bash
# 잘못된 체크 (규칙 설명 주석 자체에 매칭)
grep -R "streamlit" charts/ drift/

# 올바른 체크 (실제 import 문에만 앵커링 — 줄 시작이 import/from인 경우만)
grep -RE "^\s*(import streamlit|from streamlit)" charts/ drift/
```

함수 호출 금지 체크도 마찬가지로, 여는 괄호까지 요구하면 "NO st.error calls" 같은
설명 문장(괄호 없음)과 실제 호출 `st.error(...)`(괄호 있음)을 구분할 수 있다.

```bash
# 잘못된 체크 (설명 문장에도 매칭)
grep "st\.error" <file>

# 올바른 체크 (실제 호출만 — 여는 괄호 요구)
grep -E "st\.error\(" <file>
```

---

## Evaluation Policy

SGAD는 LLM 기반 evaluate를 Story 완료의 1차 판정 기준으로 사용하지 않는다.

Story 완료 여부는 실행 가능한 검증을 우선한다.

- Story-specific test
- Regression test
- Compile check
- Import / smoke check
- Project ADR violation check
- Interface/Data Contract check

`ooo evaluate`는 선택적 보조 리뷰로 사용할 수 있다.
그러나 evaluate 결과가 긍정적이더라도 Executable Gate를 통과하지 못하면 Story Done으로 인정하지 않는다.

> **LLM이 좋다고 한 코드가 아니라, 실행 가능한 Gate를 통과한 코드만 Done이다.**

---

## 세션 복원력

모든 핵심 상태가 파일로 저장되기 때문에
Claude capacity limit이나 세션 중단이 발생해도 작업이 유실되지 않는다.

```
SGAD.md                                            ← 방법론 전체 (이 파일)
seeds/story-X.X-name.yaml                          ← 현재 Story Seed
docs/workflow/workflow-metrics-log.md              ← 어디까지 완료됐는지
docs/contracts/interface-data-contract-registry.md ← 확정된 계약
```

새 세션에서 Metrics Log를 보면 마지막으로 완료된 Story를 확인할 수 있다.
"Story X.X까지 완료, 다음은 Y.Y" 한 줄로 재개가 가능하다.

---

## Stage 4 — Cross-Story Integration Smoke (자동, 사람 판단 아님)

Stage 3 Gate는 **Story 단위**로 "이 Story가 이전 것들을 안 깨뜨렸는가"를 본다 (개별 함수/단위 회귀).
Stage 5는 **사람이 직접** 앱을 써보며 "실제로 쓸만한가"를 판단한다.

그 사이에 빈 자리가 있다: **여러 Story가 실제로 이어질 때만** 드러나는 문제 — Story별 단위 회귀는
각 조각을 따로 검증하지, 여러 기능이 하나의 이어진 흐름 안에서 실제로 상태를 주고받을 때만
나타나는 버그는 잡지 못한다.

이걸 **Stage 4**로 정의한다 — 여전히 기계적 자동화 검증이며(사람 판단 아님), 한 Epic/버전의 모든 Story가
끝난 뒤 1회 수행한다. **Stage 5(사람 직접 검증)와 절대 혼동하지 말 것** — 기록할 때 항상
"(자동)" / "(사람)"을 타이틀에 명시한다.

### 수행 방법

- 프로젝트의 UI 프레임워크가 제공하는 자동화 테스트 도구(예: Streamlit의 `AppTest`, 웹 프레임워크의
  E2E 테스트 도구)로 **실제 DB/스토리지의 복사본**(절대 원본 아님)에 대해 엔트리 포인트를 처음부터
  끝까지 구동한다.
- 여러 상호작용을 하나의 연속된 세션으로 이어서 실행한다.
- 브라우저 자동화 도구(Playwright 등)가 있다면 대신 그걸로 실제 렌더링을 확인한다.
- 확인 대상: 예외 발생 여부, 상태 전환이 여러 Story의 함수들 사이에서 일관되게 흐르는지, 빈 상태/에러
  상태 메시지가 실제 조건과 맞게 뜨는지.

> **테스트 시나리오 범위 (Stage 5와의 차이):** Stage 4는 **이번 Epic이 새로 추가한 플로우와
> 그 플로우가 의존하는 이전 Epic 코드**를 중심으로 시나리오를 짠다 — 그 과정에서 이전 Epic
> 코드가 같이 구동되는 건 자연스럽지만, 이번 Epic과 무관한 이전 Epic의 독립적인 플로우까지
> 일부러 다시 훑지는 않는다. Stage 5는 반대로 "이전 Story에서 통과한 기능이 여전히 동작하는지"
> (smoke regression)를 검증 방법에 명시하므로, 이번 Epic과 무관하더라도 누적된 전체 플로우를
> 다시 훑는다. 자동화(Stage 4)는 새 통합 지점 위주로 자주 싸게, 사람 검증(Stage 5)은 전체를
> 드물게 비싸게 — 라는 구분이다.

### 발견된 문제 처리

Stage 5와 동일한 분류표를 그대로 쓴다 — 전부 Stage 3 Step 1로 돌아가 처리한다.
**Stage 4는 Stage 5를 대체하지 않는다** — 여기서 통과해도 사람이 직접 써보는 Stage 5는 반드시 별도로 수행한다.

### 기록 위치

`docs/workflow/workflow-metrics-log.md`에 "Stage 4 — Cross-Story Integration Smoke (자동)" 섹션으로
검증한 플로우와 발견 사항을 기록한다 (Stage 5 섹션과 별개).

---

## Stage 5 — Validate (Full Manual Product Validation, 사람)

해당 Epic의 모든 Story가 완료된 뒤 직접 실행해서 검증한다. Stage 4와 마찬가지로 **Epic/버전
단위**로 수행한다 — 전체 프로젝트의 마지막 Epic까지 미루지 않는다.
Stage 3 Gate는 "코드가 깨지지 않았는가", Stage 4는 "여러 기능이 이어져도 안 깨지는가"를 본다면,
Stage 5는 "실제로 써보니까 이게 필요하다"가 나오는 단계다.
이슈가 나오는 것은 정상이며, 이것이 SGAD 루프가 닫히는 지점이다.

> **왜 Epic 단위인가:** Stage 5에서 나온 이슈는 Stage 1으로 돌아가 Story/Architecture를 고치는
> 루프를 연다. 이 피드백을 마지막 Epic까지 미루면, 그 사이 다른 Epic들이 이미 잘못된 가정 위에
> 쌓여서 되돌리는 비용이 커진다 — Stage 3 Gate를 Epic이 아니라 Story 단위로 도는 것과 같은 이유다.
> Stage 1의 Baseline 체크리스트가 각 Epic을 "끝나면 실행 가능한 증분"으로 슬라이싱하도록
> 요구하는 것도 이 때문이다.
>
> **예외:** 정말 독립적으로 슬라이싱이 안 되는 Epic(순수 인프라/데이터 레이어 Epic 등, 그
> 자체로는 실행해볼 유저 플로우가 없는 경우)만, 그 Epic의 Stage 5를 다음 Epic의 Stage 5에
> 묶어서 함께 수행한다. Metrics Log에 "Stage 5 — v[X] (Epic N+M 묶음, Epic N은 유저 플로우
> 없어 단독 검증 불가)"처럼 사유를 기록한다. 이 예외를 기본 전략으로 쓰지 않는다.

### 검증 방법

- 주요 유저 플로우를 직접 실행한다 (happy path)
- 이전 Story에서 통과한 기능이 여전히 동작하는지 확인한다 (smoke regression)
- 정상/에러/빈 데이터 등 주요 상태를 눈으로 확인한다

### 문제 발견 시 분류

| 발견된 문제 | 처리 |
|---|---|
| 단순 구현 버그 | Fix Story → Stage 2 Guard 업데이트 → Stage 3 Step 1 |
| UX/기능 보완 | Enhancement/Change Story → Stage 2 Guard 업데이트 → Stage 3 Step 1 |
| Architecture/ADR 변경 필요 | Stage 1 산출물 통제된 변경 → Stage 2 Guard 업데이트 → Stage 3 Step 1 |
| SGAD Gate 규칙 문제 | `gate-project.md` 수정 → 필요 시 Documentation Story → Stage 3 Step 1 |

> **Stage 2 Guard 업데이트란:** 새 Story가 추가될 때마다 DoD 항목 확인, Registry에 새 인터페이스
> Proposed 등록, Metrics Log에 새 Story 항목 추가. Stage 2를 처음부터 재작성하는 게 아니라 기존
> 문서에 내용을 추가하는 것이다.

> **한 Stage 5 세션에서 문제가 여러 개 나오면 배치로 처리한다.** 이슈마다 Story를 즉시
> Stage 3에 태우고 그때마다 Epic 전체의 Stage 4/5를 재실행하지 않는다 — Stage 4/5가
> Story가 아니라 Epic/버전 단위인 이유(비싼 검증을 자주 돌리지 않기 위함)와 어긋난다.
> 대신: 이번 세션에서 발견한 문제를 전부 Story로 만들어 등록 → 그 Story들을 각각
> Stage 3(Seed→QA→Run→Gate→Registry/Log)로 순서대로 처리 → 배치가 다 끝난 뒤에만
> 해당 Epic의 Stage 4/5를 한 번 재실행한다.

### 전체 루프

```
[최초]
Stage 1 → Stage 2 → Stage 3(Epic의 Story 반복) → Stage 4(자동) → Stage 5(사람)
                                                        ↓
                                        ┌───────────────┴───────────────┐
                                   문제 발견 시                    문제 없음
                                        ↓                               ↓
                    (필요 시) Stage 1 일부 수정                다음 Epic 있음?
                       (Epic/Story/ADR)                              ↓
                              ↓                          있으면 → Stage 1/2 재실행 없이
                     Stage 2 Guard 업데이트                Stage 3 Step 1 (다음 Epic 첫 Story)
                              ↓                          없으면 → 전체 완료
                  Stage 3 Step 1 — 다음 Story 선택
                              ↓
             Seed → QA → Run → Gate → Registry/Log
                              ↓
              Stage 4(자동) → Stage 5(사람) 재실행
```

---

## 한계

- **초기 세팅 비용이 높다.** Stage 1+2 완료 후에야 구현을 시작할 수 있다.
- **소규모 프로젝트에는 과할 수 있다.** Story 수가 적을수록 Guard 문서 3종의 ROI가
  낮아진다. 정확히 몇 개부터 과한지는 근거가 없다 — 프로젝트 규모와 Story 간
  의존도를 보고 판단한다.
- **아직 검증 중이다.** 방법론 자체가 실제 프로젝트 적용을 통해 계속 정제되고 있다.
- **BMAD/Ouroboros 버전에 의존적이다.** 두 플러그인의 커맨드/스킬 이름, Seed 스키마, QA 임계값은
  버전에 따라 달라질 수 있으므로 `/bmad-help`, `ooo help`로 실제 사용 가능한 인터페이스를 먼저 확인한다.
