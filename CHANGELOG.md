# Changelog

## 0.1.4 — 2026-04-15

### Added
- Page types 4 → 6 (Karpathy 원문 반영): `comparison`, `overview` 추가. 기존은 `summary | entity | concept | synthesis`였는데 Karpathy는 "summaries, entity pages, concept pages, comparisons, an overview, a synthesis"를 구분함.
- Template에 빈 `comparisons/`, `overview/` 폴더 추가.
- `wiki-querier` 저장 판단 로직 확장: 답변 유형(비교 / 조망 / 종합)에 따라 `comparisons/` / `overview/` / `syntheses/` 중 맞는 폴더로 저장.

### Changed
- `schema.md`의 type union 확장 + 섹션별 바디 가이드(comparison, overview) 추가.
- `wiki-query` SKILL Step 4 저장 판단을 유형별 표로 재작성.
- README vault 구조 섹션 6폴더로 확장.

## 0.1.3 — 2026-04-15

### Changed
- **Template 평탄화**: `template/vault/llm-wiki/*` → `template/*`. Vault는 그냥 마크다운 디렉토리라는 사실을 template 구조에도 반영. 설치자가 `.`(프로젝트 루트)을 vault로 쓰든 하위 폴더를 쓰든 template 복사 타겟만 바꾸면 됨.

### Added
- **Onboarding 대화**: vault 경로를 찾지 못하면 orchestrator가 `(1) 기존 경로 입력 / (2) cwd 사용 / (3) 하위 디렉토리 생성` 중 하나를 묻고, 확정된 경로를 프로젝트 `CLAUDE.md`에 자동 기록. 다음 세션부터 반복되지 않음. CLAUDE.md가 없으면 최소 스텁으로 생성.

## 0.1.2 — 2026-04-15

### Changed (breaking for existing vaults)
- Rename `sources/` folder → `summaries/`, and frontmatter `type: source` → `type: summary`. "Sources" was ambiguous with `raw/` originals; per Karpathy's doc, the LLM-written wiki pages are **summaries**. Schema, all agent definitions, all skills, and the template vault have been updated. Existing vaults need to rename their folder and update frontmatter — see the repair note in this release's commits for a one-liner.

## 0.1.1 — 2026-04-15

### Fixed
- `wiki-ingest`: 로컬 외부 경로로 들어오는 소스를 `raw/`에 복사하지 않고 비규정 `source_path:` 필드로 외부 절대경로만 기록하는 버그. 스킬과 schema를 모두 보강하여 **모든 소스는 `raw/` 아래 사본이 반드시 존재**하도록 명시. 원본 경로는 선택 필드 `external_origin:`으로만 기록.

## 0.1.0 — 2026-04-15

### Added
- Initial release: 3 agents (ingestor / querier / linter) + 4 skills (orchestrator + per-op workflows) + Obsidian vault template. Karpathy의 llm-wiki 패턴 구현.
