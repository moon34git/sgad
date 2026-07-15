# SGAD — Story-Gated Agentic Development

*A controlled workflow connecting BMAD specifications to Ouroboros execution
through executable Story-level gates.*

BMAD로 설계하고 Ouroboros로 구현하되, Story 단위 Gate를 통과해야만 다음으로
넘어가는 Controlled AI-assisted Development Methodology.

전체 방법론은 [`SGAD.md`](./SGAD.md) 참고. 이 저장소는 방법론 문서 + 그걸 실제로
라우팅하는 Claude Code 스킬(`sgad`) + 새 프로젝트에 그대로 복사해 쓰는 Guard 문서
템플릿으로 구성된다.

## 전제 조건

| 항목 | 확인 방법 |
|---|---|
| Claude Code (CLI) | `claude --version` |
| BMAD | Claude Code에서 `/bmad-help` 입력 시 응답 있음. `npx bmad-method install`로 설치하는 별도 npm 패키지라 Claude Code 플러그인 마켓플레이스로는 자동 설치되지 않는다 — SGAD 설치 시 `/sgad`가 감지해서 실행 여부를 물어본다. |
| Ouroboros | Claude Code에서 `ooo help` 입력 시 응답 있음. 정식 Claude Code 플러그인이라 아래 SGAD 설치 시 `dependencies`로 자동 함께 설치된다. |

BMAD와 Ouroboros는 이 저장소에 포함되어 있지 않다 — SGAD는 그 둘을 잇는 얇은
오케스트레이션/거버넌스 레이어일 뿐, 어느 쪽도 대체하지 않는다.

## 설치

### 플러그인으로 설치 (권장)

```
/plugin marketplace add <이 저장소 URL 또는 로컬 경로>
/plugin install sgad@sgad
```

Ouroboros가 아직 설치되어 있지 않으면 `dependencies` 선언에 따라 자동으로 함께
설치된다. BMAD는 플러그인이 아니라 별도 npm 패키지라 자동 설치 대상이 아니다 —
설치 후 처음 `/sgad`를 실행하면 BMAD 존재 여부를 확인하고, 없으면
`npx bmad-method install` 실행 여부를 먼저 물어본다.

### 수동 설치 (플러그인 마켓플레이스를 쓸 수 없는 경우)

```bash
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

## 왜 쓰는가

BMAD와 Ouroboros를 각각 단독으로 쓸 때 남는 구멍을 메운다.

- **BMAD 단독**: Story/PRD/Architecture 같은 설계 산출물은 잘 나오지만, Story
  완료 여부가 에이전트의 자기 판단에 의존한다 — 기계적으로 검증 가능한
  완료 조건이 없다.
- **Ouroboros 단독**: Seed의 `exit_conditions`로 완료를 기계적으로 검증하지만,
  PRD/Architecture 같은 설계 문서 자산이 안 남고, Seed 크기를 얼마나 작게
  잡을지 강제하는 장치가 없다 — 스코프를 크게 잡을수록 QA 반복과 AC 트리
  분해 비용이 커진다.
- **SGAD**: BMAD Story = Ouroboros Seed 크기를 1:1로 강제하고, 그 사이에
  Interface/Data Contract Registry(인터페이스 드리프트 추적)와 Executable
  Gate(LLM 판단이 아닌 실행 가능한 검증만 Done으로 인정)를 끼워 넣는다.

새 기능을 추가하는 게 아니라, 두 방법론이 각자 챙기지 않는 이음매를 강제하는
얇은 체크리스트에 가깝다. Story 수가 적은 소규모 프로젝트에는 오버헤드가
더 클 수 있다 — [`SGAD.md`의 한계](./SGAD.md#한계) 참고.

## 구조

```
sgad/
├── .claude-plugin/
│   ├── plugin.json                             # 플러그인 매니페스트 (dependencies: ouroboros)
│   └── marketplace.json                        # 이 저장소 자체를 마켓플레이스로 노출
├── SGAD.md                                     # 방법론 전체
├── skills/sgad/SKILL.md                        # /sgad 라우터 스킬
├── templates/
│   ├── sgad-config.yaml                        # Gate 명령어용 프로젝트 설정
│   ├── definition-of-done.md
│   ├── workflow-metrics-log.md
│   ├── interface-data-contract-registry.md
│   └── gate-project.md
└── docs/                                       # (필요 시 예시/노트 추가)
```

## 라이선스

[MIT](LICENSE)
