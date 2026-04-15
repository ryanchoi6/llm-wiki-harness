---
name: wiki-ingest
description: 새로운 raw 소스(파일, URL, 텍스트)를 읽어 위키에 incremental하게 통합한다. 소스 요약 페이지 생성, 관련 entity/concept 페이지 업데이트·생성, index.md와 log.md 갱신을 한 번에 처리한다. 'raw에 넣어줘', '이 문서 추가해줘', '위키에 정리', 'ingest', '자료 등록', '이 URL 넣어줘' 등의 요청에 반드시 사용. 기존 ingest 결과 재처리/보완 요청 시에도 사용.
---

# Wiki Ingest Workflow

새 소스를 위키에 통합하는 구체 절차. `wiki-ingestor` 에이전트가 이 스킬을 따른다.

## 전제

- Vault 절대경로는 오케스트레이터가 prompt로 넘겨준다. 이하 모든 경로는 vault 루트 기준.
- 먼저 `schema.md`를 읽는다 — frontmatter/네이밍/링크 규칙의 유일한 진실.

## Step 1: 소스 확보 — raw/에 사본이 반드시 존재해야 한다

**불변 원칙**: 모든 소스는 **vault 내부 `raw/` 아래에 사본이 존재**해야 한다. vault 밖 경로·URL을 단순히 가리키는 포인터로 끝내면 안 된다. raw가 "불변 source of truth"라는 스키마 대전제가 깨지고 vault 재현성이 사라진다.

입력 유형별:
- **이미 `raw/`에 있는 파일**: 경로 확인 후 읽기. 복사 불필요.
- **URL**: WebFetch로 내용 확보 후 `raw/<slug>.md` 로 저장 (원문 유지). 이미지는 `raw/assets/`.
- **로컬 외부 경로** (예: `/Users/.../other-project/doc.md`): **반드시 `raw/<slug>.<ext>` 로 복사**한 뒤 진행. 원본 경로는 소스 페이지 frontmatter에 `external_origin: <원본 절대경로>` (선택 필드)로 기록만. `source_ref`는 항상 vault 상대경로 `raw/<slug>.<ext>`.
- **붙여넣기 텍스트**: 제목을 사용자와 합의하거나 첫 헤더에서 추론하여 `raw/<slug>.md`로 저장.

**금지**:
- `source_ref: external:...` 같은 가상 스킴
- `source_path:` 등 스키마에 없는 frontmatter 필드 신설 (보조 메타는 `external_origin`만 허용)
- vault 밖 절대경로를 `source_ref`로 사용

슬러그는 kebab-case, 최대 ~50자. 파일명 충돌 시 suffix(`-v2` 등).

## Step 2: 관련 기존 페이지 조사

`index.md`를 읽고 이 소스와 겹칠 법한 엔티티/개념 페이지 목록을 만든다. 해당 페이지들을 실제로 읽어 현재 주장을 파악.

## Step 3: 소스 요약 페이지 작성

`sources/<slug>.md` 생성:

```markdown
---
type: source
tags: [...]
created: <today>
updated: <today>
source_ref: raw/<slug>.md
---

# <Title>

<1-paragraph summary>

## Key takeaways
- ...

## Entities mentioned
- [[entity-a]]
- [[entity-b]]

## Concepts
- [[concept-x]]

## Notable quotes
> "..." — key load-bearing claim

## Notes
- 소스 내에서 확실치 않은 부분: ...
```

## Step 4: 엔티티/개념 페이지 업데이트·생성

Step 2에서 식별한 각 엔티티/개념에 대해:

- **존재**: 해당 페이지를 읽고, 이 소스가 더하는 새로운 사실/뉘앙스를 병합한다. 기존 내용 덮어쓰기 금지. `sources:` frontmatter 리스트에 이 소스 추가. `updated` 갱신.
- **부재**: 새 페이지 생성 (최소 뼈대 + 이 소스의 관련 부분).
- **충돌**: 기존 주장과 새 주장이 다르면 `## Contradictions` 섹션을 만들거나 보강하여 양측 + 출처를 기록.

엔티티 페이지 템플릿:
```markdown
---
type: entity
tags: [...]
created: ...
updated: ...
sources: [[source-1]], [[source-2]]
---

# <Name>

<1-line definition>

## Overview
<short prose, with [[wikilinks]] everywhere relevant>

## Attributes
- ...

## Relations
- relates to [[other-entity]] via ...

## Sources
- [[source-1]] — what this source contributes
- [[source-2]] — ...
```

개념 페이지도 구조 동일, 제목과 바디 내용만 다름.

## Step 5: index.md 재생성

실제 파일시스템을 스캔하여 `index.md`의 4개 섹션(Sources/Entities/Concepts/Syntheses)을 다시 쓴다. 각 라인:
`- [[page-name]] — one-line summary (N sources)`

`N sources`는 `sources:` frontmatter 배열 길이.

## Step 6: log.md 추가

append:
```markdown

## [YYYY-MM-DD] ingest | <Source Title>
- touched: [[source-page]], [[entity-1]], [[concept-x]], ...
- notes: <short description of what was learned / contradicted / newly created>
```

날짜는 today. 여러 번 ingest한 하루에는 항목을 여러 개 만든다(병합 금지).

## Step 7: 사용자 보고

- 터치된 페이지 개수 + 목록
- 핵심 takeaways 3~7개
- 후속 제안 (비어있는 관련 엔티티, lint 추천 등) 0~2개

## Anti-patterns

- 기존 페이지 내용을 통째로 덮어쓰기 → **금지**. 반드시 병합.
- `raw/` 하위 파일 수정 → **금지**. 실수로 touch되었으면 즉시 원복.
- 소스에 없는 사실 창작 → **금지**. 추측은 `> [!note] Unverified:`로 표시.
- index 증분 업데이트(한 줄만 추가) → 권장하지 않음. 항상 전체 재생성하여 drift 방지.
