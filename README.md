# llm-wiki-harness

[Andrej Karpathy의 LLM Wiki 패턴](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)을 Claude Code 하네스로 구현한 플러그인.

RAG처럼 매 질문마다 원본을 다시 뒤지는 대신, LLM이 **영속적인 wiki(Obsidian vault)** 를 incremental하게 빌드·유지한다. 소스를 한 번 ingest하면 요약·엔티티·개념·상호참조가 계속 누적된다.

## 설치

```bash
# 1) 마켓플레이스 등록
/plugin marketplace add ryanchoi6/llm-wiki-harness

# 2) 플러그인 설치
/plugin install llm-wiki@llm-wiki-harness
```

## 빠른 시작

1. 프로젝트 루트에서 Claude Code를 연다.
2. 첫 요청 예: `"이 URL을 위키에 추가해줘 https://example.com/article"`
3. 오케스트레이터가 vault를 못 찾으면 **onboarding 대화**로 경로를 묻는다:
   - `(1)` 기존 vault가 있다 → 절대경로 입력
   - `(2)` 현재 디렉토리를 vault로 사용 (**권장** — 프로젝트 루트 자체가 Obsidian vault)
   - `(3)` 하위 디렉토리에 새로 만들기
4. 선택이 끝나면 플러그인 template이 복사되고, 경로가 프로젝트 `CLAUDE.md`에 자동 기록된다. 다음 세션부터는 묻지 않음.
5. Obsidian에서 해당 경로를 vault로 열면 graph view로 진행 상황을 볼 수 있다.

### 경로 수동 지정

onboarding 건너뛰고 싶으면 미리 프로젝트 `CLAUDE.md` 에:

```markdown
**Vault 경로**: .
```
(또는 절대/상대 경로). 환경변수 `LLM_WIKI_VAULT` 도 작동.

## 3가지 Operation

| Op | 트리거 예시 | 담당 에이전트 |
|----|----------|-------------|
| **ingest** | "이 PDF 위키에 넣어줘", "raw/article.md 정리" | `wiki-ingestor` |
| **query**  | "X와 Y 비교해줘", "이 개념 설명" | `wiki-querier` |
| **lint**   | "위키 점검", "모순 확인", "고아 페이지" | `wiki-linter` |

모든 요청은 `wiki-orchestrator` 스킬을 거쳐 자동 라우팅된다. 수동으로 에이전트를 호출할 필요 없음.

## Vault 구조

Vault는 그냥 마크다운 디렉토리. 프로젝트 루트를 그대로 vault로 써도 되고, 하위 폴더로 분리해도 됨.

```
<vault>/
├── schema.md        # 위키 규약 (LLM이 따르는 단일 기준)
├── index.md         # 전체 페이지 카탈로그 (매 ingest마다 재생성)
├── log.md           # 연대기 로그 (append-only)
├── raw/             # 원본 소스 (immutable — LLM은 읽기만)
│   └── assets/
├── summaries/       # 소스별 요약 페이지 (type: summary)
├── entities/        # 사람/조직/장소/제품 (type: entity)
├── concepts/        # 개념·기법·아이디어 (type: concept)
├── comparisons/     # X vs Y, tradeoff 분석 (type: comparison)
├── overview/        # 도메인 조망·진입점 맵 (type: overview)
└── syntheses/       # 다중 소스 종합·분석 (type: synthesis)
```

- 모든 페이지에 YAML frontmatter(`type`, `created`, `updated`, `sources`).
- Obsidian 위키링크(`[[page-name]]`)로 전 페이지 상호 연결.
- 모순은 삭제하지 않고 `## Contradictions` 블록으로 보존.

## 에이전트·스킬

```
agents/
├── wiki-ingestor.md   # 소스 → 요약 + 엔티티/개념 업데이트 + index/log
├── wiki-querier.md    # 위키 기반 답변 + syntheses 저장
└── wiki-linter.md     # 모순/orphan/dangling/frontmatter 점검

skills/
├── wiki-orchestrator/ # 진입점 (라우터)
├── wiki-ingest/       # ingest 워크플로우 상세
├── wiki-query/        # query 워크플로우 상세
└── wiki-lint/         # lint 워크플로우 상세

template/
└── vault/llm-wiki/    # 첫 초기화 시 복사되는 스캐폴드
```

## 설계 원칙

- **raw/는 불변**. LLM이 절대 수정하지 않는다.
- **병합·보존 우선**. 기존 페이지 덮어쓰기 금지. 모순은 양측 기록.
- **Citation 필수**. 모든 주장에 `[[source]]` 링크.
- **삭제 금지**. lint도 orphan을 삭제하지 않고 리포트만.

## Credits

- Pattern: [Andrej Karpathy](https://github.com/karpathy) — [gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- Harness scaffold: [revfactory/harness](https://github.com/revfactory/harness)

## License

MIT
