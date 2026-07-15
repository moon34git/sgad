# Project Gate — [프로젝트명]

Layer 2 Project Gate 정의. 프로젝트의 Architecture Decision Record(ADR)를 근거로
Story마다 실행 가능한 체크로 옮겨 적는다. `sgad` 스킬이 Stage 3 Step 5에서 이 파일의
"명령어" 블록을 그대로 실행한다.

`grep -R`/`grep -A`는 결과가 없으면 exit code 1을 반환한다 — "결과 없음 = 통과"가
되도록 반드시 if/else로 감싼다. `&&`로 이어서 "명령 실패 = Gate 실패"와
혼동하지 않는다.

> **주석/docstring 자체가 규칙을 설명하며 금지어를 그대로 적어두는 경우가 흔하다**
> (예: 파일 상단에 "이 모듈은 streamlit을 import하면 안 됨" 같은 주석). 단순
> `grep "<pattern>"`은 이 설명 문장 자체에 걸려 늘 FAIL로 잘못 보고한다. 반드시
> 아래 PG-01처럼 **실제 import 문/호출 형태로 앵커링**해서 규칙 설명 텍스트와
> 실제 위반 코드를 구분한다. (SGAD 초기 버전에서 실제로 이 함정에 걸려
> `grep -R "streamlit"`이 "ADR-06: streamlit import 금지"라는 주석 자체를
> 위반으로 오탐한 사례가 있었다.)

---

## PG-01. [금지 import 체크 예시]

- 대상: [파일 또는 디렉토리]
- 규칙: [금지/허용 패턴]
- 명령어:

```bash
# 줄 시작이 import/from인 경우만 매칭 — "~를 import하면 안 됨" 같은 주석 문장은
# 매칭되지 않는다
if grep -RE "^\s*(import <module>|from <module>)" <target>/; then
  echo "FAIL: <reason>"
  exit 1
else
  echo "PASS: <reason>"
fi
```

- 근거 ADR: ADR-XX

---

## PG-02. [금지 호출 체크 예시]

- 대상: [파일]
- 규칙: [금지 함수 호출]
- 명령어:

```bash
# 여는 괄호까지 요구해서 실제 호출만 매칭 — "NO <call> allowed" 같은 규칙 설명
# 주석 자체는 괄호가 없어서 매칭되지 않는다
if grep -E "<forbidden_call>\(" <file>; then
  echo "FAIL: <reason>"
  exit 1
else
  echo "PASS: <reason>"
fi
```

- 근거 ADR: ADR-XX

---

## PG-03. [함수 범위 한정 체크 예시]

파일 전체가 아니라 특정 함수 내부만 검사해야 할 때 (예: 캐시 데코레이터 때문에
파일 자체는 정당하게 특정 모듈을 import하지만, 특정 함수 내부에서는 금지해야 하는 경우).

```bash
if grep -A 50 "def <function_name>" <file> | grep -E "<forbidden_call>\("; then
  echo "FAIL: <reason>"
  exit 1
else
  echo "PASS: <reason>"
fi
```

- 근거 ADR: ADR-XX

---

## PG-04. [Documentation Story 전용 — 문서-구현 일치 체크 예시]

Story Type이 Documentation인 Story는 definition-of-done.md의 "문서-구현 일치"
항목을 범용 자동화로 검증할 수 없다 — Documentation Story마다 이 형식으로
전용 PG 항목을 추가한다.

- 대상: `<문서 파일>` (예: README.md의 설치 절차 섹션)
- 규칙: 문서가 언급하는 파일 경로/함수명이 실제로 존재한다. 코드 예시가 있으면
  그대로 실행했을 때 성공한다.
- 명령어:

```bash
# 1) 문서가 언급하는 파일/모듈이 실제로 존재하는지 확인
if [ ! -f "<문서가 언급하는 파일 경로>" ]; then
  echo "FAIL: <문서>가 언급하는 <파일>이 존재하지 않음"
  exit 1
fi

# 2) 문서의 코드 예시를 그대로 실행 (예: bash 블록을 추출해 실행하거나,
#    최소한 예시에 등장하는 커맨드/함수가 현재 코드베이스에 존재하는지 grep)
<문서의 코드 예시와 동일한 커맨드>
```

- 근거: 이 Story의 Documentation DoD 항목 (definition-of-done.md 참고)
