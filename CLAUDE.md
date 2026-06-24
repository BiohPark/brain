# CLAUDE.md — 이 Obsidian Vault 관리 규칙

이 저장소는 사용자의 **개인 Obsidian vault**이자 **개인 RAG**다. Claude Code(이 에이전트)는 이 vault를
관리한다: 노트 생성/편집, 메타데이터 위생, 자동 링킹·MOC 유지, 검색/요약. 동시에 별도 프로그램
**`mes-agent`** 도 이 vault를 읽고 쓰므로, 두 시스템의 규칙은 **절대 충돌하면 안 된다.**

---

## 🚨 가드레일 (위반 금지)

1. **`agent/` 폴더는 `mes-agent`(타 업무 자동화 프로그램)의 업무관리 DB다.**
   → Claude Code는 `agent/`에 **쓰기/수정/삭제를 절대 하지 않는다.** 규칙 정합성 확인용 *읽기*만 허용.
   → 모든 스캔·감사·정리 작업에서 `agent/`는 **제외**한다.
2. **규칙의 단일 출처(Single Source of Truth)는 `[[🏛 Vault Guides]]` 와 `agent/guides/🤖 Agent 노트작성 가이드`.**
   이 CLAUDE.md는 그 규칙을 *복제·강제*할 뿐 **새 규칙을 만들지 않는다.** 규칙을 바꾸기 전 반드시 두 문서를
   먼저 읽고, `mes-agent`와의 divergence가 없는지 확인한다. (충돌은 "몹시 위험")
3. **`private/30. Secrit Notes/`** (심리상담·에세이 등)은 **민감 정보**다. 외부 전송·요약 게시 금지.
   사용자의 직접 요청 없이는 열람을 최소화한다.
4. **비밀키**(REST API apiKey)는 이 파일이나 vault 노트에 평문으로 적지 않는다. (`.mcp.json`에만 보관)
5. Claude Code 자체 산출물(감사 리포트, 작업 로그)은 **`claude/`** 폴더에만 둔다. 실제 지식 노트는 vault
   컨벤션을 그대로 따라 해당 폴더(`private/...` 등)에 만든다.

---

## 폴더 소유권 맵

| 폴더 | 소유/용도 | Claude Code 권한 |
|------|-----------|------------------|
| `agent/` | **mes-agent** 업무관리 DB | **읽기 전용 참조만** (쓰기 절대 금지) |
| `claude/` | **Claude Code** 작업공간 (리포트·플레이북) | 읽기/쓰기 |
| `private/` | 개인 지식 노트 본체 | 컨벤션 준수하에 읽기/쓰기 |
| `private/30. Secrit Notes/` | 민감 정보 | 최소 열람, 외부 반출 금지 |
| `🏛 SecureInfo` (루트) | **비밀정보**(API키·계정) | 🚨 열람·반출 금지 |
| `🏛 Expire•Subscription` (루트) | **개인 금융**(결제·보험·공과금·차량) | 🚨 외부 반출 금지 |
| 루트 `🏛`·`🏷` MOC 노트 | 마스터 인덱스·대시보드 (→ 아래 "홈(루트) 핵심 문서") | 신중히 편집 (링크 추가 등) |
| `public/`, `copilot/`, `private_work/` | 외부 발행/타 도구 | 요청 시에만 |

> `agent/`와 `claude/`의 차이: **`agent/`=다른 프로그램이 쓰는 운영 DB**, **`claude/`=이 에이전트의 작업 흔적.**
> 둘을 혼동하지 말 것. 자세한 구분은 [[claude/README]] 참조.

---

## 홈(루트) 핵심 문서

루트 `.md`는 vault의 **진입점·대시보드·인덱스**다. **시작 진입점은 `🏛 Home`(대시보드, homepage 플러그인 자동 오픈).** 진입 순서: `🏛 Home`(현황) → `🏛 Vault Guides`(규칙) → `🏛 Categories`(분류) → `🏛 Tasks`/`Tasks Kanban`(실행).

| 파일 | 역할 | 주의 |
|------|------|------|
| `🏛 Home` | **대시보드 홈**(노트·링크 통계, 활동 히트맵, 누적 차트, 지식구조 차트, Tasks). dataviewjs+`dashboard.css` 스니펫. `🔍` 드릴다운 허브 | dataviewjs 블록·`cssclasses: dashboard` 보존 |
| `🏛 Categories` | **마스터 분류 인덱스**(번호→카테고리 트리). 모든 `Categories` 속성의 정본 | 구조 변경 신중. 2026-06-14 정리됨 |
| `🏛 Vault Guides` | vault 규칙 원천(아이콘·속성·태그). 이 CLAUDE.md의 **상위 출처** | 규칙 SoT, 임의 변경 금지 |
| `🏛 Tasks` | 오늘/진행/반복 **Task 대시보드**(Tasks 플러그인 쿼리) | `` ```tasks `` 쿼리 블록 보존 |
| `🏛 Tasks Kanban` | 진행 프로젝트 **칸반**(kanban-plugin) | 플러그인 관리 형식 보존 |
| `🏛 Utils` | 바로가기(API/SBL/PDF)·Obsidian 팁·단축키·검색법·인용 스타일. `![[🏛 Vault Guides]]` 임베드 | — |
| `🏛 SecureInfo` | **비밀정보**(API키·계정) | 🚨 열람·반출 금지 |
| `🏛 Expire•Subscription` | 정기결제·보험·공과금·차량·만료 **개인 금융 대시보드** | 🚨 외부 반출 금지 |
| `🏷 Manual` / `🏷 Career` / `🏷 Health` | 도메인 인덱스 **스텁(거의 비어있음)** | 다듬기/채우기 후보 |
| `🏷 Invest` | 투자·경조사·용돈 기록 | 개인 금융정보, 반출 주의 |
| `🔍 …` (드릴다운) | `🏛 Home` 통계 클릭 시 열리는 **Dataview 목록 노트**: `🔍 고립·미분류 노트`·`🔍 카테고리별 노트`·`🔍 태그별 노트`·`🔍 최근 수정 노트` | Dataview 쿼리 보존. `Indexes: [[🏛 Home]]` |

## Vault 구조·규모

- 규모: `.md` 약 367개. `private/`(≈311) 지식 본체, `agent/`(≈44) = **mes-agent DB(쓰기 금지)**, `claude/` = 본 에이전트 작업공간.
- `private/` 하위: `00. Inbox`(미분류 유입), `10. Calendar Notes`, `30. Secrit Notes`(민감), `80. References`(첨부), `90. Settings`(Templater 템플릿·Periodic Notes).
- 분류 번호 체계: 파일명 `NNN` → 부모 `📚NN0`, `NNN-X` → `NNN`, `📚NN0` → `📖N00`. 상세 트리는 `[[🏛 Categories]]`.
- 첨부 폴더: `private/80. References/81. Attachments` (`.obsidian/app.json` 기준).

---

## 속성(frontmatter) 스키마  — 출처: [[🏛 Vault Guides]]

```yaml
---
tags:                      # 키워드. 형식: #대분류/소분류 (중첩 슬래시, 한글 가능, 공백 없음)
Categories:                # 1차 인덱스. [[🏛 Categories]]와 연계. [[link]] 배열
  - "[[📚630 KnowledgeManagement]]"
Indexes:                   # 2차 인덱스. 🏷 인덱스 노트와 연계. [[link]] 배열
  - "[[🏛 Utils]]"
People:                    # 연관된 사람. [[@이름]] 배열
  - "[[@박비오]]"
date_created: 2026-06-14T00:00:00+09:00   # ISO 8601
date_modified:
status:                    # (업무 노트) wait | 진행 | 완료 | 보류
---
```

- **업무/회의 노트**는 추가로 `title`, `date_requested/due/done`, `status`를 가진다.
  템플릿 구조: `private/90. Settings/91. Templates/(Templater) 업무`, `(Templater) 회의`.

## 아이콘·분류 레벨

| 아이콘 | 의미 | 번호 체계 |
|--------|------|-----------|
| `🏛` | 메인/홈/가이드 | — |
| `📖` | 1차 분류 (Category Lv.1) | 100–900 (1자리 백번대) |
| `📚` | 2차 분류 (Category Lv.2) | N01–N99 (2자리) |
| (무아이콘) | 3차 분류 (상세 주제) | — |
| `🏷` | 인덱스 (Index/MOC) | — |
| `🔍` | 대시보드 드릴다운 쿼리 노트 (`🏛 Home` 하위) | — |

## 파일명 규칙

| 유형 | 형식 | 예 |
|------|------|----|
| 분석/개발 노트 | `YYYY-MM-DD-kebab-제목` | `2026-06-14-배포-결과-분석` |
| 업무 태스크 | `YYYY-MM-DD-업무-제목` | `2026-06-14-Syncade-배포` |
| 회의 노트 | `YYYY-MM-DD-회의-주제` | `2026-06-14-스프린트-리뷰` |
| 가이드/MOC | 아이콘 + 제목 | `🏛 Utils`, `🏷 Career` |

## Wikilink 규칙
- 관련 노트는 `[[노트명]]`, 별칭은 `[[노트명|표시텍스트]]`, 섹션은 `[[노트명#섹션]]`.
- 백링크가 많을수록 노트의 RAG 가치가 높다. 적극 연결한다. (단, 인지 가능한 범위 내에서)
- **노트 제목 변경은 옵시디언 앱에서** 수행(앱이 백링크 자동 갱신). 파일을 직접 rename하면 들어오는 링크가 깨질 수 있으니, 직접 편집으로 이름을 바꿀 땐 **그 노트를 가리키는 모든 링크도 함께 치환**한다.

---

## Vault 접근 방식 (하이브리드)

| 작업 | 방식 | 도구 |
|------|------|------|
| 대량 읽기·편집·검색(grep), 백그라운드 정리, 오프라인 작업 | **직접 파일 (기본)** | `Read`/`Write`/`Edit`/`Grep`/`Glob` |
| 활성 노트 열기·포커스, **Templater 등 명령 실행**, 태그 목록, 인앱 검색 | **REST API (MCP, 보조)** | 아래 MCP 도구 |

**전제**: REST API/MCP는 **Obsidian이 실행 중일 때만** 동작한다. 플러그인 "Local REST API & MCP Server v4.1".
MCP 서버: `http://127.0.0.1:27123/mcp` (설정은 `.mcp.json`). 주요 MCP 도구:

- 읽기/쓰기: `vault_read` `vault_write` `vault_append` `vault_patch` `vault_delete` `vault_move`
- 목록/구조: `vault_list` `vault_get_document_map` `tag_list`
- 검색: `search_simple` `search_query`
- 명령(Templater): `command_list` `command_execute` · 활성파일: `active_file_get_path` `open_file`

> 의미기반(임베딩) 검색은 Obsidian의 **smart-connections / smart-composer** 플러그인이 vault 내부에서
> 담당한다. Claude Code의 메타데이터 위생·링킹 작업이 이 임베딩 RAG와 mes-agent의 검색 품질을 직접 끌어올린다.

---

## 4대 관리 작업 (슬래시 커맨드)

| 커맨드 | 용도 |
|--------|------|
| `/vault-note` | 컨벤션대로 노트 생성/편집 (frontmatter·아이콘·파일명·폴더·구조) |
| `/vault-audit` | 메타데이터 위생 점검 → `claude/reports/`에 리포트 (수정은 승인 후, `agent/` 제외) |
| `/vault-moc` | 관련 노트 자동 링킹 + `🏛 Categories`·`🏷` 인덱스 갱신 |
| `/vault-search` | Categories/Indexes/tags/People 기준 검색 + 다중 노트 교차 요약 |

상세 플레이북: `claude/workflows/`.

---

## 🏛 Home 대시보드 & Vault 건강도 개선 활동

**대시보드 구성 파일** (2026-06-14 신설, SoT `🏛 Vault Guides`에도 등록됨):
- `🏛 Home` (root) — dataviewjs로 실시간 계산. 퀵스탯(총노트·총링크·고립·카테고리·Inbox)·활동 히트맵·누적 성장 차트(obsidian-charts)·지식구조 차트(카테고리/태그/폴더/건강도)·Tasks·발견.
- `🔍 고립·미분류 노트`·`🔍 카테고리별 노트`·`🔍 태그별 노트`·`🔍 최근 수정 노트` (root) — 통계 클릭 시 열리는 Dataview 목록.
- `.obsidian/snippets/dashboard.css` — `cssclasses: dashboard` 전용 스타일(테마 변수 기반). 비활성 시 설정→모양→CSS 스니펫에서 `dashboard` 토글.
- `homepage` 플러그인 — 시작 시 `🏛 Home` 자동 오픈.
- 모든 통계는 `"private"` 범위. 제목이 노출되는 목록(최근수정·랜덤·히트맵 상세)에서는 `30. Secrit Notes` 제외. `agent/` 미포함.

**Vault 건강도 개선 활동 (정기 루틴)** — RAG 품질을 끌어올리는 핵심 작업:
1. `🏛 Home`의 **고립/미분류/미태그 수치**(🩺 Vault 건강도)와 **Inbox 대기 수**를 확인한다.
2. `🔍 고립·미분류 노트`·`🔍 태그별 노트`를 열어, **거기 나열된 노트들의 메타데이터를 개선**한다:
   - `Categories` 추가(→ `[[🏛 Categories]]` 트리에서 적절한 `📚`/`📖` 선택)
   - `Indexes` 추가(→ 관련 `🏷` 인덱스 연결)
   - `tags` 보강(`#대분류/소분류` 형식) 및 관련 노트 `[[wikilink]]` 백링크
3. 수정은 **vault 컨벤션 준수 + 사용자 승인 후** 진행한다(대량 변경은 `/vault-audit`→리포트→승인 흐름).
4. 목표: 고립/미분류/미태그 수치를 점진적으로 낮추고 Inbox를 비운다. 숫자가 줄면 임베딩 RAG와 `mes-agent` 검색 정확도가 함께 오른다.

> 이 활동은 `🏛 Home` 통계를 "보기"에서 "고치기"로 잇는 루프다. 한 번에 다 하지 말고 인지 가능한 분량씩.

## RAG 유지 원칙
이 vault는 RAG로 쓰인다. **컨벤션 일관성(정확한 frontmatter·태그·링크) = 검색 품질**이다.
Claude Code가 하는 모든 정리·링킹은 사용자와 `mes-agent` 양쪽의 검색 정확도를 높이기 위한 것이다.
규칙을 임의로 바꿔 일관성을 깨뜨리지 않는다 — 의심되면 `[[🏛 Vault Guides]]`를 먼저 확인한다.
