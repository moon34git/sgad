# Definition of Done

Story Type별 완료 판정 기준. SGAD Stage 2에서 생성, Stage 5에서 문제 발견 시 갱신한다.

이 문서는 "왜/무엇을" 설명하는 사람용 기준이고, 실제 기계적 검증은 `sgad` 스킬의
Gate 실행(Layer 1 / Layer 1 확장 / Layer 2)이 담당한다 — Architecture ADR과
`gate-project.md`의 관계와 동일한 산문↔실행 분리다. 항목마다 어느 Gate 단계가
검증하는지 표시했다.

## Backend

- [ ] Unit test 전체 통과 — **Layer 1** (`test_command`)
- [ ] Compile check 통과 — **Layer 1** (`compileall`)
- [ ] Import check 통과 — **Layer 1** (`entry_point` import)
- [ ] Project ADR 위반 없음 (`docs/workflow/gate-project.md` 기준) — **Layer 2**

Backend Story는 Layer 1 + Layer 2가 항상 실행되므로 이 4개는 별도 조치 없이
자동으로 검증된다.

## UI

Backend 기준 + 아래

- [ ] UI 프레임워크 smoke test 통과 (엔트리 포인트 구동 시 예외 없음) — **Layer 1 확장** (`ui_smoke_command`)
- [ ] 화면/패널 단위 import 가능 — **Layer 1 확장** (`ui_panels`)

Story Type이 UI면 `sgad-config.yaml`의 `ui_smoke_command`/`ui_panels`를 채워야
Gate가 이 두 항목을 검증할 수 있다 — 비어있으면 Gate가 멈추고 채우라고 안내한다.

## Integration

Backend/UI 기준 + 아래

- [ ] 전체 테스트 스위트 통과 — **Layer 1** (`test_command`가 Story Type과 무관하게 항상 전체 스위트를 돈다)
- [ ] 이전 Story 전체 regression 통과 — **Layer 1** (위와 동일한 이유로 자동 충족)

Integration Story도 별도 조치 없이 Layer 1만으로 이미 충족된다.

## Documentation

- [ ] 문서 내용과 실제 구현이 일치 — **Layer 2** (해당 Story 전용 PG 항목 필요)
- [ ] 코드 예시가 있다면 실행 가능 — **Layer 2** (해당 Story 전용 PG 항목 필요)

Documentation Story는 범용 자동 검증이 불가능하다 — `gate-project.md`에 이
Story 전용 PG 항목(문서가 언급하는 파일/함수 존재 확인, 코드 예시 실행)을 직접
추가해야 한다. PG 항목이 없으면 Gate가 "Documentation Story인데 대응 PG 항목
없음"으로 멈춘다 (예시는 `gate-project.md` 템플릿 참고).
