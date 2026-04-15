---
name: wiki-lint
description: 위키 전체의 건강 상태를 점검한다. 모순, stale, orphan, dangling link, missing entity, frontmatter 결함, schema drift, index 동기화를 체크하고, 자동 수정 가능한 것은 고치며 판단 필요한 것은 리포트한다. '위키 점검', '건강 체크', '모순 확인', '고아 페이지', '링크 검사', 'lint', '위키 정리', '위키 상태' 등의 요청에 반드시 사용.
---

# Wiki Lint Workflow

`wiki-linter` 에이전트가 따르는 절차. 삭제는 하지 않는다. 자동 수정은 안전한 것만.

## Step 0: 이전 lint 결과 조회

`log.md`의 직전 lint 엔트리를 찾아 이전에 보고된 이슈 목록을 메모. 해결 여부를 이번 점검에 표시한다.

## Step 1: 인덱스 동기화

1. 파일시스템을 스캔하여 6개 폴더(`summaries/`, `entities/`, `concepts/`, `comparisons/`, `overview/`, `syntheses/`)의 파일 목록을 수집.
2. `index.md`와 비교.
3. 불일치면 `index.md`를 실제 상태로 재생성 (자동 수정).

## Step 2: Frontmatter 검증

각 페이지 순회:
- `type`, `created`, `updated` 필수. 누락 시:
  - `updated` → 오늘 날짜로 자동 보충 (자동 수정)
  - `type` → 폴더 기반 추론하여 자동 보충 (자동 수정)
  - `created` → 누락 시 파일 mtime 근사값으로 보충 (자동 수정)
- `sources`/`source_ref` 누락: 추측 금지, 리포트만.
- frontmatter 자체가 없음: 리포트.

## Step 3: Dangling links

모든 `[[target]]`을 추출 → 해당 파일이 vault에 존재하는지 확인.
- 존재하지 않음 + 유사 이름 페이지가 있음 → 자동 교정(자동 수정). 수정 내역을 log에 기록.
- 유사 없음 → 리포트 ("missing page: [[target]] referenced by [[a]], [[b]]").

## Step 4: Orphans

어느 페이지에서도 링크되지 않는 페이지 목록 (index.md 제외).
→ 삭제 금지. 리포트만. 링크 후보 위치 2~3개 제안.

## Step 5: Missing entities/concepts

`summaries/` 전체를 훑어 고유명사/기술용어 중 3회 이상 등장하지만 자체 페이지가 없는 항목을 수집 → 리포트.

## Step 6: Contradictions & stale

- 각 엔티티/개념 페이지의 본문에서 서로 충돌하는 주장 여부 확인 (`## Contradictions` 블록이 없는데 내용이 충돌하는 경우).
- `created`/`updated`가 오래된 주장이 이후 소스에 의해 반박당한 경우 리포트.
- 자동 수정 금지 — 사용자 판단 영역.

## Step 7: Schema drift

`schema.md`의 규칙과 다른 패턴이 여러 페이지에 반복 등장하면 리포트. 규칙을 갱신할지 사용자와 협의.

## Step 8: 리포트 작성

사용자에게 전달:

```markdown
## Lint report <today>

### 자동 수정됨 (N건)
- index.md 재생성 (sources 2건, entities 1건 반영)
- [[foo]]: frontmatter updated 보충
- [[bar]]: dangling `[[old-name]]` → `[[new-name]]` 교정

### 확인 필요 (유형별)
**Contradictions**: ...
**Missing entities**: ...
**Orphans**: ...
**Stale candidates**: ...

### 이전 리포트 대비
- 해결됨: ...
- 남음: ...

### 제안
- ingest할 만한 소스: ...
- 다음 lint 권장 시점: 소스 N개 이상 추가 후
```

## Step 9: log.md 추가

```markdown

## [YYYY-MM-DD] lint | routine
- touched: index.md, <자동수정된 파일들>
- notes: auto-fixed N; reported M (contradictions, missing, orphans, stale)
```

## 안전 원칙

- **삭제 금지**. orphan도 보존.
- **내용 재작성 금지**. frontmatter 보충과 dangling link 교정만 허용.
- 모호하면 리포트로 돌리고 사용자 판단을 기다린다.
