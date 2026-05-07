# 메시지 계약

에이전트 간 SendMessage 포맷을 정의한다.
모든 에이전트는 이 파일을 참조하여 메시지를 보내고 받는다.

---

## 공통 메시지 구조

모든 메시지는 아래 공통 필드를 포함한다:

```
FROM: [보내는 에이전트명]
TO: [받는 에이전트명]
TYPE: HANDOFF | REQUEST | RESPONSE | ERROR
PHASE: [현재 워크플로우 단계]
```

---

## 에이전트별 HANDOFF 포맷

### agent-A → agent-B

agent-A가 작업을 완료하고 agent-B에게 넘길 때 사용한다.

```
FROM: agent-A
TO: agent-B
TYPE: HANDOFF
PHASE: 2a
PAYLOAD:
  output_file: _workspace/01_[산출물].md
  summary: |
    [핵심 내용 3줄 이내]
  key_points:
    - [agent-B가 반드시 알아야 할 항목 1]
    - [agent-B가 반드시 알아야 할 항목 2]
  assumptions:
    - [가정한 내용 — 확인이 필요한 경우 명시]
  blockers: [없음 | 있을 경우 상세 기술]
```

### agent-A → agent-C

```
FROM: agent-A
TO: agent-C
TYPE: HANDOFF
PHASE: 2b
PAYLOAD:
  output_file: _workspace/01_[산출물].md
  summary: |
    [핵심 내용 3줄 이내]
  key_points:
    - [agent-C가 반드시 알아야 할 항목]
  blockers: [없음 | 있을 경우 상세 기술]
```

### [추가 에이전트 쌍이 있을 경우 동일한 구조로 반복]

---

## 수정 요청 포맷

reviewer가 🔴 발견 시 해당 에이전트에게 보내는 메시지.

```
FROM: reviewer
TO: [해당 에이전트명]
TYPE: REQUEST
PHASE: 3-revision
PAYLOAD:
  issue_id: RV-[번호]
  severity: 🔴 | 🟡
  target_file: _workspace/0N_[산출물].md
  problem: |
    [무엇이 기준을 충족하지 못했는가 — 구체적으로]
  expected: |
    [어떻게 수정되어야 하는가 — 구체적으로]
  criteria_ref: contracts/review-criteria.md#[섹션명]
  deadline: [재작업 완료 기준 — 예: "다음 reviewer 호출 전"]
```

## 수정 완료 응답 포맷

수정 작업을 마친 에이전트가 reviewer에게 보내는 메시지.

```
FROM: [수정한 에이전트명]
TO: reviewer
TYPE: RESPONSE
PHASE: 3-revision
PAYLOAD:
  issue_id: RV-[번호]
  status: RESOLVED | PARTIAL | BLOCKED
  changes: |
    [무엇을 어떻게 수정했는가]
  remaining: [없음 | PARTIAL/BLOCKED인 경우 이유]
```

---

## 에러 메시지 포맷

에이전트가 작업 실패 시 오케스트레이터에게 보내는 메시지.

```
FROM: [에이전트명]
TO: orchestrator
TYPE: ERROR
PHASE: [발생 단계]
PAYLOAD:
  error_type: [에러 유형]
  description: [무슨 일이 발생했는가]
  attempted: [시도한 것]
  fallback: [대안 제안 또는 "없음"]
```
