# [하네스 이름]

[한 줄 설명: 이 하네스가 무엇을 하는가]

## 구조

```
.claude/
├── CLAUDE.md
├── agents/
│   ├── agent-A.md
│   ├── agent-B.md
│   ├── agent-C.md
│   └── reviewer.md
├── skills/
│   ├── orchestrator/
│   │   └── skill.md
│   ├── domain-skill-1/
│   │   └── skill.md
│   └── domain-skill-2/
│       └── skill.md
└── contracts/
    ├── messages.md
    └── review-criteria.md
```

## 사용법

[트리거 방법: 슬래시 커맨드 또는 자연어 예시]

## 에이전트-스킬 연결 매핑

| 에이전트 | 참조 스킬 |
|---------|----------|
| agent-A | `/domain-skill-1` |
| agent-B | `/domain-skill-1`, `/domain-skill-2` |
| agent-C | `/domain-skill-2` |
| reviewer | (스킬 없음 — `contracts/review-criteria.md` 참조) |

## 산출물

모든 산출물은 `_workspace/` 디렉토리에 저장된다:

| 파일 | 생성 주체 | 설명 |
|------|---------|------|
| `00_input.md` | 오케스트레이터 | 사용자 입력 정리 |
| `01_[산출물명].md` | agent-A | [설명] |
| `02_[산출물명].md` | agent-B | [설명] |
| `03_[산출물명].md` | agent-C | [설명] |
| `99_review.md` | reviewer | 검증 보고서 |
