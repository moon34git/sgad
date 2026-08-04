# SGAD — Story-Gated Agentic Development

*A governance layer over BMAD and Ouroboros — no Story is Done until its
executable gates pass.*

AI 에이전트에게 실제로 코드를 맡기면 마지막에 이 질문이 남는다 — **"이거 다 된 건가?"**
지금은 그 판정을 대체로 에이전트 자신이 한다. SGAD는 판정 주체를 LLM의 자기 평가에서
**실행 가능한 게이트**로 옮긴다. 컴파일·단위 테스트·전체 회귀·아키텍처 규칙 검사를 통과한
Story만 Done으로 인정하고, 통과하지 못하면 다음 Story로 넘어가지 않는다.

> LLM이 좋다고 한 코드가 아니라, 실행 가능한 Gate를 통과한 코드만 Done이다.

구현 방식은 새 도구를 만드는 게 아니라, 이미 있는 두 도구 사이에 게이트를 끼워 넣는
것이다 — **[BMAD](https://github.com/bmad-code-org/BMAD-METHOD)**가 만든 Story와
**[Ouroboros](https://github.com/Q00/ouroboros)**의 Seed 실행 사이에 Story 단위
Executable Gate, 인터페이스 계약 추적, 사람 직접 검증 단계를 강제하는 얇은 거버넌스
레이어다. 둘 다 서드파티 도구이고 이 저장소에 포함되지 않는다 — 각자 여기 쓰이는 것보다
넓은 기능을 갖고 있고, SGAD는 그중 일부만 입력으로 쓴다.

전체 방법론은 [`SGAD.md`](./SGAD.md) 참고. 이 저장소는 방법론 문서 + 그걸 실제로
라우팅하는 Claude Code 스킬(`sgad`) + 새 프로젝트에 그대로 복사해 쓰는 Guard 문서
템플릿으로 구성된다.

## 왜 쓰는가

BMAD와 Ouroboros를 각각 단독으로 쓸 때 남는 구멍을 메운다.

- **BMAD 단독**: 설계 산출물만 만드는 도구가 아니다 — 구현(`bmad-dev-story`),
  코드 리뷰(`bmad-code-review`), E2E 테스트 생성(`bmad-qa-generate-e2e-tests`)까지
  커버한다. 다만 그 전체 사이클 어디에도 **"검증을 통과해야 다음 Story로 넘어간다"는
  강제가 없다.** 완료 판정이 에이전트의 리뷰 판단에 맡겨지고, 판정이 부정적이어도
  진행이 막히지 않는다.
- **Ouroboros 단독**: Seed의 `exit_conditions`로 완료를 기계적으로 검증하고,
  `ooo pm`으로 PRD도 생성한다. 다만 Architecture·UX 같은 설계 문서 자산은 남지
  않고, Seed 크기를 얼마나 작게 잡을지 강제하는 장치가 없다 — 스코프를 크게
  잡을수록 QA 반복과 AC 트리 분해 비용이 커진다.
- **SGAD**: BMAD Story = Ouroboros Seed 크기를 1:1로 강제하고, 그 사이에
  Interface/Data Contract Registry(인터페이스 드리프트 추적)와 Executable
  Gate(LLM 판단이 아닌 실행 가능한 검증만 Done으로 인정)를 끼워 넣는다.
  Gate는 그 Story만 통과하는 걸로 부족하다 — **이전 Story 전체의 회귀를 함께
  요구한다**(Local Success + Global Compatibility). 여기에 더해 Epic이 끝날 때마다
  Stage 4(자동 cross-story 통합 스모크)와 Stage 5(사람이 직접 실행해보는 검증)를
  Epic 단위로 강제한다 — 특히 Stage 5는 두 도구 어디에도 워크플로우 단계로 없고,
  실제 적용에서 게이트를 통과한 코드의 결함이 드러난 지점이 대부분 여기였다
  (아래 [적용 사례](#적용-사례) 참고).

> 위 비교는 **BMAD 6.10.0 / Ouroboros 0.50.4** 기준이다(2026-08-04 확인). 두 도구
> 모두 활발히 개발 중이라 버전이 올라가면 달라질 수 있다.

새 기능을 추가하는 게 아니라, 두 방법론이 각자 챙기지 않는 이음매를 강제하는
얇은 체크리스트에 가깝다. Story 수가 적은 소규모 프로젝트에는 오버헤드가
더 클 수 있다 — [`SGAD.md`의 한계](./SGAD.md#한계) 참고.

부수 효과로 진행 상태가 전부 파일에 남는다 — Metrics Log의 마지막 완료 Story,
Registry의 확정된 계약, 진행 중인 Seed. 세션이 끊기거나 사용 한도에 걸려도
"Story X.X까지 완료, 다음은 Y.Y" 한 줄로 재개된다.

## 적용 사례

두 프로젝트에 실제로 적용했다. 각 저장소의 `docs/workflow/workflow-metrics-log.md`에
Story별 Gate 결과가 그대로 남아 있어서, 이 문서의 설명보다 그쪽이 더 정확한 증거다.

### [occupancy-edge-monitor](https://github.com/moon34git/occupancy-edge-monitor) — SGAD를 처음부터 적용한 프로젝트

Raspberry Pi 엣지 노드 → AWS 중앙 서버 → Streamlit 대시보드로 이어지는 재실 감지
파이프라인. Epic 1~5 / 24 Story, Guard 문서 5종 전부 갖추고 진행했다.

- Story마다 Seed QA 점수 궤적이 기록돼 있다 (`0.83 → 0.88 → 0.92` 식으로, 몇 번 만에
  임계값을 넘겼고 무엇을 고쳐서 넘겼는지).
- **Registry가 실제로 계약 변경을 통제한 사례**: Story 2.3에서 엣지가 보내는 envelope를
  중앙이 버리고 있다는 걸 착수 전에 발견했는데, 등록된 계약 `IF-010`을 고쳐 쓰는 대신
  optional 필드를 더하는 additive 확장으로 처리했다. 이미 그 계약을 쓰던 이전 Story들이
  깨지지 않는다. Registry 47건이 이런 판단의 근거로 남아 있다.
- Epic 1~4는 Stage 4(자동 통합 스모크) → Stage 5(사람/실환경)까지 종결. Epic 4는 실제
  Raspberry Pi + AWS에서 회복탄력성을 실측했다 — `kill -9` 후 systemd 재시작 ~5.3초,
  리부트 후 자동 재개 ~40초, 파이프라인 ~15.6시간 무중단.
  (Epic 5의 7개 Story는 Epic 4 Stage 5에서 나온 이슈들을 배치로 처리한 것이라 Stage 3까지
  완료된 상태다.)
- **Stage 5에서만 잡힌 결함 2건**이 이 방법론의 핵심 주장("Executable Gate 통과 ≠ Done")을
  실증한다:
  - *Story 3.1* — 차트 spec 구조 검증은 통과했지만, 앱이 실제로 호출하는 빌더에는 pan/zoom이
    배선돼 있지 않았다. 테스트가 검증한 빌더와 앱이 쓰는 빌더가 갈라져 있었고, 사람이 렌더된
    화면을 보고서야 드러났다.
  - *Story 4.2* — SQLite 테스트 더블이 `bool`↔`integer` 타입 불일치와 트랜잭션 오염을
    구조적으로 잡지 못했다. 실제 Postgres에 배포하고서야 나타났다.

### [turbofan-phm-dashboard](https://github.com/moon34git/turbofan-phm-dashboard) — SGAD의 규칙들이 나온 곳

CMAPSS 터보팬 데이터 기반 드리프트/RUL 모니터링 대시보드. Epic 1~11 / Story 기록 55건
규모로, **SGAD 초기 버전으로 진행됐다.** Seed는 48개 — 나머지 7건은 Stage 5에서 발견해
Seed를 거치지 않고 직접 수정한 것으로, 그 자체가 초기 버전의 느슨함을 보여주는 기록이다.
지금 `SGAD.md`에 있는 규칙 중 여럿이 여기서 겪은 문제의 결과물이다.

- `grep -R "streamlit"`이 *"streamlit import 금지"* 라고 적어둔 주석 한 줄에 걸려 정상 코드를
  위반으로 오탐 → "자주 하는 실수"의 앵커링 규칙
- Stage 4가 v2부터 뒤늦게 도입됐고 v3에서는 누락된 걸 발견해 소급 수행 → Stage 4를 별도
  단계로 정의하고 Epic 단위 수행을 명문화
- Layer 2 Gate가 별도 문서 없이 Story마다 애드혹 grep으로만 남음 → `gate-project.md` /
  `sgad-config.yaml` 도입

그래서 이 프로젝트는 SGAD의 적용 사례라기보다 **출처**에 가깝다. 방법론이 완성된 상태에서
적용된 결과를 보려면 위의 occupancy 쪽을 보는 편이 낫다.

## 전제 조건

| 항목 | 확인 방법 |
|---|---|
| Claude Code (CLI) | `claude --version` |
| BMAD | Claude Code에서 `/bmad-help` 입력 시 응답 있음. `npx bmad-method install`로 설치하는 별도 npm 패키지라 Claude Code 플러그인 마켓플레이스로는 자동 설치되지 않는다 — SGAD 설치 시 `/sgad`가 감지해서 실행 여부를 물어본다. |
| Ouroboros | Claude Code에서 `ooo help` 입력 시 응답 있음. 정식 Claude Code 플러그인. 없어도 SGAD 설치를 막지 않는다 — 설치 후 처음 `/sgad`를 실행하면 자동으로 감지해서 설치 여부를 물어본다. |

BMAD와 Ouroboros는 이 저장소에 포함되어 있지 않다 — SGAD는 그 둘을 잇는 얇은
오케스트레이션/거버넌스 레이어일 뿐, 어느 쪽도 대체하지 않는다.

## 설치

### 플러그인으로 설치 (권장)

```
/plugin marketplace add moon34git/sgad
/plugin install sgad@sgad
```

BMAD/Ouroboros 설치 여부와 순서를 신경 쓸 필요 없다 — SGAD 플러그인은 둘 중 뭐가 있든
없든 항상 그대로 설치·로드된다(플러그인 매니페스트에 Ouroboros를 `dependencies`로
선언하지 않는다 — Claude Code의 플러그인 dependency는 대상 마켓플레이스가 미리 등록돼
있을 때만 작동하고, 아니면 전체 로드가 조용히 실패해 오히려 설치를 더 깨뜨린다는 게
실측으로 확인됐다). 설치 후 처음 `/sgad`를 실행하면 BMAD/Ouroboros 각각의 존재 여부를
스킬이 직접 확인해서, 없는 것만 설치 여부를 물어보고 승인받은 뒤 설치까지 진행한다 —
자세한 동작은 `skills/sgad/SKILL.md`의 Prerequisite Check 참고.

### 수동 설치 (플러그인 마켓플레이스를 쓸 수 없는 경우)

```bash
# 0. 이 저장소를 받는다 (아래 경로들이 여기 기준이다)
git clone https://github.com/moon34git/sgad.git

# 1. sgad 스킬 (Claude Code가 /sgad로 인식)
cp -r sgad/skills/sgad <대상 프로젝트>/.claude/skills/sgad

# 2. Guard 문서 템플릿 (채워서 사용)
mkdir -p <대상 프로젝트>/docs/workflow <대상 프로젝트>/docs/contracts
cp sgad/templates/sgad-config.yaml            <대상 프로젝트>/docs/workflow/
cp sgad/templates/definition-of-done.md       <대상 프로젝트>/docs/workflow/
cp sgad/templates/workflow-metrics-log.md     <대상 프로젝트>/docs/workflow/
cp sgad/templates/gate-project.md             <대상 프로젝트>/docs/workflow/
cp sgad/templates/interface-data-contract-registry.md <대상 프로젝트>/docs/contracts/
```

어느 방식으로 설치하든, `docs/workflow/sgad-config.yaml`을 프로젝트 구조(엔트리
포인트, 소스 디렉토리, 테스트 커맨드)에 맞게 채우고, `docs/workflow/gate-project.md`에
프로젝트 ADR 기반 Layer 2 Gate 체크를 채운다.

## 사용

Stage 1(BMAD 설계) + Stage 2(Guard 문서 채우기)를 마친 뒤, Claude Code에서:

```
/sgad
```

지금 Stage/Step이 어디인지, 다음에 뭘 실행해야 하는지(`ooo seed`, `ooo run ...`,
Gate 실행 등) 알려준다. Gate를 바로 돌려달라고 하면 `sgad-config.yaml` +
`gate-project.md` 기준으로 실행하고 결과를 보고한다.

## 구조

```
sgad/
├── .claude-plugin/
│   ├── plugin.json                             # 플러그인 매니페스트
│   └── marketplace.json                        # 이 저장소 자체를 마켓플레이스로 노출
├── SGAD.md                                     # 방법론 전체
├── skills/sgad/SKILL.md                        # /sgad 라우터 스킬
└── templates/
    ├── sgad-config.yaml                        # Gate 명령어용 프로젝트 설정
    ├── definition-of-done.md
    ├── workflow-metrics-log.md
    ├── interface-data-contract-registry.md
    └── gate-project.md
```

## 라이선스

[MIT](LICENSE)
