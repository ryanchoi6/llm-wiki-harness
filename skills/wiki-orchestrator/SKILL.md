---
name: wiki-orchestrator
description: LLM wiki 시스템을 운영하는 오케스트레이터. Obsidian vault에 raw 소스를 ingest하거나, 위키 기반 질문에 답하거나(query), 위키 건강을 점검(lint)하는 모든 작업을 적절한 에이전트로 라우팅. '위키', 'wiki', '자료 정리', '문서 추가', '이 URL 넣어줘', '정리해줘', '비교해줘', '분석해줘', '위키 점검', 'lint', '모순 확인', '고아 페이지' 등 위키 관련 모든 표현에 트리거. 결과 수정, 재실행, 업데이트, 보완, 다시 실행 요청 시에도 반드시 이 스킬을 사용.
---

# Wiki Orchestrator

llm-wiki 시스템의 진입점. 사용자 요청을 `ingest`, `query`, `lint` 중 어디로 보낼지 라우팅한다.

Agents: `wiki-ingestor`, `wiki-querier`, `wiki-linter`

## Phase 0: Vault 경로 확정 (onboarding 포함)

**0-a. 경로 탐색** (우선순위):
1. 프로젝트 `CLAUDE.md`의 `**Vault 경로**:` 필드
2. 환경변수 `$LLM_WIKI_VAULT`
3. cwd 자체가 vault인지 체크: `./schema.md` 존재?
4. 하위 디렉토리 자동 탐지: `./wiki/`, `./vault/`, `./vault/llm-wiki/`, `./llm-wiki/` 중 `schema.md`가 있는 첫 디렉토리

**0-b. 경로가 확정됐고 `schema.md`가 있으면 정상** — `log.md` 마지막 5개 엔트리를 읽어 최근 상태 파악 후 Phase 1로.

**0-c. 경로를 못 찾았으면 onboarding 대화:**

사용자에게 이렇게 묻는다:
```
llm-wiki vault가 아직 설정되지 않았어요. 어떻게 할까요?

1) 기존 vault가 있다 — 절대경로를 알려주세요
2) 현재 디렉토리(<cwd>)를 vault로 사용 (권장, 플랫)
3) 새 하위 디렉토리에 만들기 — 기본값 ./wiki
```

답변에 따라:
- **(1)** 지정 경로 검증 → `schema.md` 없으면 (3)로 fallback 질문
- **(2)** 현 cwd를 vault로 확정
- **(3)** 지정 경로에 새 vault 생성

**0-d. 새 vault 생성 시**: 플러그인 템플릿(`${CLAUDE_PLUGIN_ROOT}/template/` — 평탄 구조의 `schema.md`, `index.md`, `log.md`, `raw/`, `summaries/`, `entities/`, `concepts/`, `syntheses/`)을 대상 경로에 복사. 복사 후 `log.md` 첫 엔트리 `[<today>] manual | vault initialized` 추가.

**0-e. 경로 persistence**: 확정된 경로를 즉시 프로젝트 `CLAUDE.md`에 기록 — 다음 세션부터 onboarding 반복 안 되도록.

- `CLAUDE.md`가 존재하면 `**Vault 경로**: <확정경로>` 라인을 상단(기존 "# ..." 제목 아래)에 삽입/업데이트
- 존재하지 않으면 최소 스텁으로 생성:
  ```markdown
  # LLM Wiki

  **Vault 경로**: <확정경로>

  이 프로젝트는 llm-wiki 플러그인을 사용한다. `vault` 경로의 `schema.md`가 규약.
  ```

경로가 `.`이면 그대로 `.`로 기록(절대경로로 과하게 확장하지 않음).

## Phase 1: 의도 분류

| 신호 | 라우팅 |
|------|------|
| 파일/URL/텍스트를 주고 "넣어줘/ingest/추가/정리/raw에" | **ingest** |
| 자연어 질문, "비교/분석/설명/정리해줘" | **query** |
| "점검/health/lint/정리된 상태/모순/고아/건강" | **lint** |

모호하면 사용자에게 한 줄 질문으로 확인. 복합 요청("ingest 후 바로 질문")은 ingest → query 순차 실행.

## Phase 2: 디스패치

선택된 에이전트를 `Agent` 도구로 호출:

- `subagent_type`: `general-purpose`
- `model`: `opus`
- `description`: 3~5단어
- `prompt`: 에이전트 정의(`.claude/agents/<name>.md`)를 읽고, 해당 스킬(`.claude/skills/wiki-<op>/SKILL.md`)을 따라 작업하도록 지시 + 사용자의 원 요청 전문 + 현재 vault 절대경로.

예시 prompt:
```
Vault 절대경로: <Phase 0에서 결정된 경로>

너는 wiki-ingestor 에이전트다. 다음을 반드시 수행:
1. 에이전트 정의(wiki-ingestor.md)와 워크플로우 스킬(wiki-ingest)을 따른다
2. <vault>/schema.md 를 읽어 규칙을 확인한다
3. 아래 사용자 요청을 처리한다

[사용자 요청 전문]
```

## Phase 3: 결과 전달

에이전트 결과를 사용자에게 요약 전달. 저장된 페이지 목록과 다음 제안(ingest할 만한 후속 소스 / lint 주기 / 관련 질문)을 1~3개 덧붙인다.

## 에러 핸들링

- 에이전트가 실패하면 1회 재시도. 재실패 시 부분 산출물은 유지한 채 사용자에게 보고.
- raw/ 쓰기 시도 차단 — 에이전트가 스키마 위반 시 오케스트레이터가 거부하고 수정 요청.

## 테스트 시나리오

**정상 (ingest)**: 사용자 "이 URL을 위키에 추가해줘 https://example.com/article"
→ Phase 1에서 ingest로 분류 → wiki-ingestor 호출 → `summaries/article.md` 생성, 관련 entity 페이지 업데이트, index/log 갱신.

**정상 (query)**: "Transformer와 RNN의 차이를 위키 내용으로 정리해줘"
→ query 분류 → wiki-querier → citation 있는 답변 + `syntheses/transformer-vs-rnn.md` 저장 제안.

**에러 (empty wiki)**: index/log만 있고 소스 없음 상태에서 query
→ querier가 "위키에 관련 근거 없음" 응답 + ingest 제안.

## 후속 작업

사용자가 "방금 답변에 X 추가해줘" / "다시 점검" / "그 엔티티 페이지 보완" 식으로 요청하면 해당 에이전트 재호출. 이전 산출물은 에이전트가 알아서 읽는다.
