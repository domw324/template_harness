# Harness Template 설명

Claude Code 에이전트 팀 하네스를 만들기 위한 범용 템플릿입니다.

에이전트 간 메시지 계약과 리뷰어 판단 기준을 명시적으로 분리하여 파이프라인이 깨지는 두 가지 주요 원인을 구조적으로 해결합니다.

## 왜 이 템플릿을 만들었는가

이 템플릿은 `contracts/` 레이어를 추가해 기존 하네스 구조에서 자주 발생하는 문제를 해결합니다.

- **에이전트 간 포맷 불일치** — 보내는 쪽과 받는 쪽이 서로 다른 구조를 기대해서 파이프라인이 중간에 끊김
- **리뷰어 판단 기준 불명확** — "검증한다"고만 정의되어 있어서 🔴를 언제 붙여야 하는지 모호함, 불필요한 재작업 루프 발생

## 구조

```
.claude/
├── CLAUDE.md                 # 프로젝트 개요 + 에이전트-스킬 연결 매핑
├── agents/
│   ├── agent-A.md            # 에이전트 정의 템플릿 (복제해서 사용)
│   └── reviewer.md           # 검증자 (대부분의 하네스에서 재사용 가능)
├── skills/
│   ├── orchestrator/
│   │   └── skill.md          # 워크플로우 + 실행 순서 + 에러 핸들링
│   └── domain-skill-N/
│       └── skill.md          # 도메인 전문 지식 (복제해서 사용)
└── contracts/
    ├── messages.md           # 에이전트 간 SendMessage 포맷 정의
    └── review-criteria.md    # 리뷰어 판단 기준 + 심각도 분류
```

### 기존 구조 대비 추가된 것

| 레이어 | 역할 | 해결하는 문제 |
|--------|------|-------------|
| `contracts/messages.md` | 에이전트 간 메시지 포맷을 한 곳에서 선언 | 포맷 불일치로 인한 파이프라인 단절 |
| `contracts/review-criteria.md` | 리뷰어의 🔴/🟡 판단 기준 명시 | 모호한 검증 기준으로 인한 불필요한 재작업 |

> [!info] 🔴/🟡 판단 이란?
> 
> LLM에게 명확한 기준을 주기 위한 심각도 레이블
> 
> ```
> 🔴 = 블로킹. 이게 있으면 다음 단계로 못 넘어감. 반드시 수정 후 재검증.
> 🟡 = 논블로킹. 진행은 가능하지만 개선 권고.
> ⚪ = 그냥 메모. 판단에 영향 없음.
> ```

## 사용법

### 1. 복제

```bash
git clone https://github.com/{your-username}/template_harness
cp -r template_harness/.claude /path/to/your-project/.claude
```

### 2. 파일 작성 순서

아래 순서로 작성하면 앞에서 정의한 내용을 뒤에서 참조할 수 있어서 일관성이 유지됩니다.

```
1. contracts/review-criteria.md   : 판단 기준 먼저 확정
2. contracts/messages.md          : 에이전트 간 포맷 정의
3. agents/*.md                    : 역할 + 입출력 명세 작성
4. skills/domain-skill-N/skill.md : 도메인 지식 채우기
5. skills/orchestrator/skill.md   : 워크플로우 + 에이전트 연결
6. CLAUDE.md                      : 전체 구조 요약
```

### 3. 파일 복제 규칙

| 템플릿 파일 | 복제 방법 |
|-----------|---------|
| `agents/agent-A.md` | 에이전트 수만큼 복제 후 이름 변경 |
| `skills/domain-skill-N/skill.md` | 도메인 스킬 수만큼 복제 후 디렉토리명 변경 |
| `agents/reviewer.md` | 대부분 그대로 재사용, 도메인 특화 기준은 `contracts/review-criteria.md`에 추가 |

### 4. 플레이스홀더 치환

모든 `[대괄호]` 항목을 실제 내용으로 교체합니다.

```bash
# 치환이 필요한 플레이스홀더 목록 확인
grep -r "\[" .claude/ --include="*.md" -l
```

## 각 파일 설명

##### `CLAUDE.md`
Claude Code가 프로젝트 진입 시 가장 먼저 읽는 파일입니다. 하네스의 전체 구조, 에이전트 목록, 에이전트-스킬 연결 매핑, 산출물 경로를 선언합니다.

##### `agents/agent-A.md`
에이전트 하나의 정의 템플릿입니다. 역할, 작업 원칙, 참조 스킬, 입력 명세, 산출물 포맷, 출력 명세, 에러 핸들링을 포함합니다. 에이전트 수만큼 복제해서 사용합니다.

##### `agents/reviewer.md`
모든 에이전트의 산출물을 교차 검증하는 역할입니다. 판단 기준은 파일 내에 하드코딩하지 않고 `contracts/review-criteria.md`를 참조합니다.

##### `skills/orchestrator/skill.md`
워크플로우 전체를 정의합니다. 실행 순서, 병렬 실행 구간, 에이전트-스킬 연결, 작업 규모별 모드, 에러 핸들링, 테스트 시나리오 3종(정상/기존파일활용/에러)을 포함합니다.

##### `skills/domain-skill-N/skill.md`
특정 도메인의 전문 지식을 담는 파일입니다. 에이전트 정의와 분리되어 있어서 여러 에이전트가 동일한 스킬을 공유할 수 있습니다.

##### `contracts/messages.md`
에이전트 간 SendMessage 포맷을 정의합니다. HANDOFF(작업 인계), REQUEST(수정 요청), RESPONSE(수정 완료), ERROR(실패 보고) 4가지 타입을 포함합니다.

##### `contracts/review-criteria.md`
리뷰어의 판단 기준을 정의합니다. 🔴(필수 수정), 🟡(권장 수정), ⚪(참고 사항) 심각도 분류와 산출물별 체크리스트, 재검증 루프 규칙을 포함합니다.

## 설계 원칙

**에이전트 정의와 계약의 분리**
에이전트 파일(`agents/`)은 역할과 원칙을 정의하고, 에이전트 간 통신 규칙은 `contracts/`에서 관리합니다. 에이전트를 추가하거나 교체할 때 다른 에이전트 파일을 수정하지 않아도 됩니다.

**스킬은 에이전트에 종속되지 않음**
스킬 디렉토리는 1:1 매핑이 아닌 도메인 단위로 구성됩니다. 하나의 스킬을 여러 에이전트가 공유하거나, 하나의 에이전트가 여러 스킬을 참조할 수 있습니다. 연결 매핑은 `CLAUDE.md`에서 중앙 관리합니다.

**리뷰어는 기준을 가진다**
리뷰어 에이전트는 주관적 판단 대신 `contracts/review-criteria.md`의 명시된 기준만으로 판단합니다. 기준을 바꾸고 싶을 때 `reviewer.md`가 아닌 `review-criteria.md`만 수정합니다.

## 참고

이 템플릿은 [revfactory/harness-100](https://github.com/revfactory/harness-100) 프로젝트의 구조를 분석하고 에이전트 간 인터페이스 계약과 리뷰어 판단 기준을 보완한 버전입니다.
