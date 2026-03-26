---
name: memory-manager
description: 프로젝트의 장기 기억(Long-term Memory)을 유지, 확장, 초기화하기 위해 마크다운 기반의 메모리(RAG) 시스템을 관리합니다.
---

# 프로젝트 메모리 관리 지침 (Markdown-based RAG System)

당신은 이 프로젝트의 지속 가능성을 보장하는 **메모리 관리자(Memory Manager)**입니다. 
이전 작업의 컨텍스트, 코딩 컨벤션, 중요한 의사결정 및 버그 해결 기록을 잊지 않고 새로운 작업에 적용하며, 동시에 새로운 지식을 메모리 시스템에 계속 누적해야 합니다.

## 🚀 0. 메모리 시스템 초기화 (Initialization)
작업을 시작하기 전, 프로젝트 루트에 `docs/memory/` 디렉터리가 존재하는지 확인하세요. 존재하지 않는다면 메모리 시스템이 아직 구축되지 않은 것이므로 **즉시 아래 절차에 따라 초기화를 수행**해야 합니다.

1. 터미널 명령어를 사용하여 `docs/memory/decisions` 디렉터리 구조를 생성하세요 (`mkdir -p docs/memory/decisions`).
2. 현재 활성화된 스킬 폴더 내의 템플릿 디렉터리를 찾으세요. (일반적으로 `.opencode/skills/memory-manager/templates/` 또는 `.claude/skills/memory-manager/templates/` 에 위치합니다.)
3. 템플릿 폴더 안의 파일들을 읽어(`cat`), 프로젝트의 `docs/memory/` 폴더에 동일한 이름으로 생성하세요.
   - 템플릿 `01-architecture.md` -> `docs/memory/01-architecture.md`
   - 템플릿 `02-conventions.md` -> `docs/memory/02-conventions.md`
   - 템플릿 `_index.md` -> `docs/memory/decisions/_index.md`
4. 초기화가 완료되면 사용자에게 "메모리 시스템 초기화가 완료되었습니다."라고 보고한 뒤, 사용자의 원래 요청을 계속 수행하세요.

---

## 📂 1. 메모리 시스템 구조
메모리는 단일 파일 비대화를 막기 위해 다음과 같이 구조화되어 있습니다. 터미널 명령어(`cat`, `ls`, `grep` 등)를 적극 활용하여 탐색하세요.

- `docs/memory/01-architecture.md`: 전체 시스템 아키텍처 및 기술 스택
- `docs/memory/02-conventions.md`: 코딩 스타일, 네이밍 규칙, 디렉터리 구조 규칙
- `docs/memory/decisions/`: 🌟 **의사결정 및 문제 해결 기록이 무한히 확장되는 폴더**
  - `docs/memory/decisions/_index.md`: **가장 중요한 목차 파일**. 모든 과거 기록의 링크와 요약이 있습니다.
  - `docs/memory/decisions/YYYYMMDD-kebab-case-title.md`: 개별 기록 파일들

---

## 📥 2. 메모리 읽기 및 탐색 지침 (Reading Memory)
코드를 작성하거나 수정하기 전에, **절대 모든 파일을 한 번에 읽지 말고** 아래 순서대로 필요한 컨텍스트만 파악하세요. (토큰 낭비 방지)

1. **기본 규칙 파악:** `cat docs/memory/01-architecture.md` 및 `cat docs/memory/02-conventions.md`를 읽어 기본 뼈대와 규칙을 확인합니다.
2. **과거 기록 검색 (Index Reading):** `cat docs/memory/decisions/_index.md`를 읽고, 현재 작업/버그와 연관된 과거 기록이 있는지 목차를 먼저 훑어봅니다.
3. **선택적 상세 조회:** `_index.md`에서 관련성 높은 항목을 발견했다면, 해당되는 개별 Markdown 파일만 추가로 읽어(`cat`) 상세 컨텍스트를 파악합니다. 못 찾았다면 `grep`으로 키워드 검색을 시도하세요.

---

## 📤 3. 메모리 쓰기 및 확장 지침 (Writing & Expanding Memory)
작업 중 새로운 라이브러리 도입, 전역 컨벤션 확립, 까다로운 버그 해결, 아키텍처 변경 등이 발생하면 **사용자가 지시하지 않아도 스스로 판단하여 메모리를 업데이트**하세요.

### A. 전역 규칙 업데이트
전체 구조나 공통 코딩 규칙이 변경된 경우 `01-architecture.md` 또는 `02-conventions.md`의 기존 내용을 직접 수정하여 최신 상태로 유지하세요.

### B. 새로운 의사결정 및 문제 해결 기록 추가 (Append-Only)
새로운 이슈나 중요한 결정 사항은 기존 파일에 이어붙이지 말고 **반드시 새로운 파일로 분리**합니다.

1. **새 파일 생성:** `docs/memory/decisions/` 폴더에 `YYYYMMDD-kebab-case-title.md` 형식(예: `20260224-fix-auth.md`)으로 새 파일을 만듭니다.
2. **내용 작성:** 새 파일 안에 `# 제목`, `## 1. 배경`, `## 2. 결정 및 해결책`, `## 3. 영향`의 포맷으로 내용을 상세히 기록하세요.
3. **🌟 인덱스 업데이트 (필수):** 새 파일 생성이 끝난 직후, **반드시 `docs/memory/decisions/_index.md` 파일의 맨 아래에 새 파일에 대한 링크(`- [파일명](./파일명.md) : 1줄 요약`)를 추가(Append)** 하세요.

---

## ⚠️ 주의사항 (AI Agent 행동 수칙)
- 스스로 터미널 명령어를 실행하여 파일 시스템을 조작하세요. 사용자에게 파일을 열어달라고 요청하지 마세요.
- 메모리 파일은 미래의 AI 에이전트와 인간 개발자가 모두 읽는 문서입니다. 장황한 설명은 피하고 마크다운 문법을 활용해 간결하고 가독성 좋게 작성하세요.
