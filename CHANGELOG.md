# Changelog

## 0.1.1 — 2026-04-15

### Fixed
- `wiki-ingest`: 로컬 외부 경로로 들어오는 소스를 `raw/`에 복사하지 않고 비규정 `source_path:` 필드로 외부 절대경로만 기록하는 버그. 스킬과 schema를 모두 보강하여 **모든 소스는 `raw/` 아래 사본이 반드시 존재**하도록 명시. 원본 경로는 선택 필드 `external_origin:`으로만 기록.

## 0.1.0 — 2026-04-15

### Added
- Initial release: 3 agents (ingestor / querier / linter) + 4 skills (orchestrator + per-op workflows) + Obsidian vault template. Karpathy의 llm-wiki 패턴 구현.
