---
name: wiki-query
description: 위키 내용을 근거로 사용자 질문에 답변한다. index.md에서 관련 페이지를 찾아 읽고, citation을 포함한 답변을 만들며, 재사용 가치가 있으면 syntheses/에 저장한다. '위키에서 찾아줘', '정리해줘', '비교해줘', '분석해줘', '무엇인지 설명' 등 위키 기반 지식 질문에 반드시 사용. 이전 답변 보완/재검색 요청 시에도 사용.
---

# Wiki Query Workflow

사용자 질문에 위키로 답하는 절차. `wiki-querier` 에이전트가 따른다.

## Step 1: index.md 스캔

`index.md`를 읽고 질문과 관련될 법한 페이지 목록을 만든다. 키워드 매칭 + 엔티티/개념 이름 유사도.

## Step 2: 페이지 로드

관련 페이지들을 실제로 읽는다. 각 페이지의 `sources:` frontmatter를 따라가 필요하면 원 `summaries/` 페이지까지 읽어 근거 강도를 확인한다.

근거가 부족하면 Grep으로 vault 전체를 보조 검색. 그래도 부족하면 "위키에 근거 없음"을 답변에 명시.

## Step 3: 답변 작성

포맷은 질문에 맞게:
- 단순 질의 → 짧은 프로즈
- 비교 → 표
- 정리 → bullet 또는 구조화된 섹션
- 타임라인 → 연대기

**citation 규칙**:
- 모든 핵심 주장 뒤에 `[[source-page]]` 또는 `[[entity]]` 링크.
- 같은 주장에 여러 출처면 모두 나열.
- `## Contradictions` 블록이 있던 페이지는 양측을 "X는 A라고 하지만 Y는 B라고 한다"로 드러낸다.

**금지**:
- citation 없는 사실 주장
- 위키에 없는 사실을 "아마도" 수준으로 채우기

## Step 4: 저장 판단

답변이 아래에 해당하면 `syntheses/<slug>.md`로 저장:
- 비교/대조 (`X vs Y`)
- 다중 소스 합성
- 새로운 연결/통찰
- 사용자가 저장을 지시

저장 페이지 프런트매터:
```yaml
---
type: synthesis
tags: [...]
created: <today>
updated: <today>
sources: [[...]]
question: "사용자 원 질문"
---
```

본문에 **질문 → 답변 → 근거 목록** 순서. 그 후:
1. `index.md` 재생성
2. `log.md`에 `## [YYYY-MM-DD] query | <slug>` 항목 추가 (touched에 synthesis 페이지 + 참조 페이지들)

단순 trivia성 답변은 저장하지 않음 (로그에도 남기지 않음).

## Step 5: 사용자 보고

- 답변 본문
- 근거 페이지 목록 (`Based on: [[a]], [[b]]`)
- 저장 여부 또는 제안
- (선택) 위키 gap 제안 1개까지

## 기존 synthesis 재활용

질문이 기존 `syntheses/` 페이지와 중복되면:
- 그 페이지를 먼저 읽고 답변에 사용
- 업데이트가 필요하면 해당 페이지를 수정하고 `updated` 갱신 (새 synthesis 생성 금지)
- log에 `query` 대신 `synthesis-updated` 메모로 기록
