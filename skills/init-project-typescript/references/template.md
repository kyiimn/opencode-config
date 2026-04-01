# 템플릿 레퍼런스 인덱스

`AGENTS.md`와 `RULES.md`를 생성할 때 사용하는 섹션 모음입니다.
파일이 커지는 것을 방지하기 위해 역할별로 분리되어 있습니다.

## 파일 구성

| 파일 | 내용 | 참조 시점 |
|------|------|-----------|
| `template.md` (이 파일) | 인덱스 + 조합 가이드 | 항상 먼저 확인 |
| `template-agents.md` | 모든 `[A]`, `[A:ROOT]`, `[A:PKG]` 섹션 | AGENTS.md 생성 시 |
| `template-rules.md` | 모든 `[R]` 섹션 | RULES.md 생성 시 |

## 파일 역할 구분

| 파일 | 역할 | 읽는 시점 |
|------|------|-----------| 
| `AGENTS.md` | 행동 지침 — 무엇을 언제 어떻게 할지 | **매 세션** 자동 주입 |
| `RULES.md` | 상세 레퍼런스 — 규칙의 구체적 정의·예시 | **필요한 순간** 에이전트가 직접 로드 |

**핵심 원칙**: AGENTS.md는 포인터만 갖는다. 상세 규칙은 모두 RULES.md에 있다.

## 섹션 태그

- `[A]` — AGENTS.md에 포함 (루트·패키지 공통)
- `[A:ROOT]` — AGENTS.md 루트 전용
- `[A:PKG]` — AGENTS.md 패키지 전용
- `[R]` — RULES.md에만 포함

---

## AGENTS.md 조합 가이드

섹션 본문: `template-agents.md`

### 루트

| 조건 | 포함 섹션 |
|------|-----------| 
| 항상 | HEADER, COMMANDS, SAFETY & PERMISSIONS, AGENT WORKFLOW, OBSERVABILITY, LIVING DOCUMENT, GIT, FILE REFERENCES, ORCHESTRATION |
| 설명 있음 | OVERVIEW |
| 싱글 프로젝트 | STRUCTURE (싱글) |
| 모노레포 | STRUCTURE (모노레포) |
| 핵심 경로 제공 | ARCHITECTURE |
| Good/Bad 파일 제공 | CODE EXAMPLES |
| 스킬 포인터 필요 | SKILL POINTERS |

> 스택별 규칙(CLI/Frontend/API)은 RULES.md에만 기술. AGENTS.md HEADER의 Stack 항목에 한 줄로만 표기.

### 패키지

| 조건 | 포함 섹션 |
|------|-----------| 
| 항상 | HEADER[PKG], AGENT WORKFLOW, OBSERVABILITY, LIVING DOCUMENT |
| 설명 있음 | OVERVIEW[PKG] |
| API 앱 | STRUCTURE[PKG-API] |
| Frontend 앱 | STRUCTURE[PKG-WEB] |
| CLI 앱 | STRUCTURE[PKG-CLI] |
| 공유 라이브러리 | STRUCTURE[PKG-CORE] |

---

## RULES.md 조합 가이드

섹션 본문: `template-rules.md`

| 조건 | 포함 섹션 |
|------|-----------| 
| 항상 | HEADER, CODE STANDARDS, JSDOC, TESTING, REFACTORING |
| API 포함 | API {#api} |
| Frontend 포함 | FRONTEND {#frontend} |
| CLI 포함 | CLI {#cli} |
| 공유 라이브러리 포함 | SHARED {#shared} |
