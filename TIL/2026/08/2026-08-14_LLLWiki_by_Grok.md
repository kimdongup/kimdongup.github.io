# LLLWiki by Grok

> 날짜: 2026-08-14
> 원본 노션: [링크](https://app.notion.com/p/LLLWiki-by-Grok-3bceaead73348075ad42dbfef57866c1)

---

```markdown
# Claude Code ↔ Grok 병행 사용 매뉴얼

한 폴더(`~/my-wiki`)를 **같은 위키**로 두고, Claude Code와 Grok Build를 **번갈아** 쓴다. 스킬·에이전트·콘텐츠를 도구별로 복제하지 않는다.

원본 하네스: [llmwiki-harness](https://skillmaru.hell0world.net/space/global/llmwiki-harness) / [GitHub](https://github.com/cookyman74/llmwiki-harness).  
이 폴더의 2026-08-14 패치는 그 위에 **Grok이 같은 규약을 읽게 하는 브릿지**만 얹은 것이다.

---

## 0. 무엇이 바뀌었나

| 파일 | 역할 |
|------|------|
| `AGENTS.md` **(신규)** | Grok이 세션마다 읽는 런타임 브릿지. 디스패치 이름·trust·핸드오프 |
| `CLAUDE.md` | 스키마 정본. 서문에 “Grok도 이 파일을 따른다” + 멀티 런타임 절 |
| `.claude/skills/wiki-ops/SKILL.md` | 전문가 호출을 Claude `Agent` / Grok `spawn_subagent` / 인라인 폴백으로 분기 |
| `.gitignore` | `AGENTS.local.md`, `.grok/settings.local.*` 무시 |

**바꾸지 않은 것:** `raw/`·`wiki/` 콘텐츠, frontmatter 스키마, `.claude/skills/*` 절차, 전문가 역할 파일.  
**하지 말 것:** `.grok/skills/wiki-*` 같은 Grok 전용 복제본. 이중 정본이 되면 두 도구가 다른 규칙으로 위키를 고친다.

규약이 충돌하면 **`CLAUDE.md`가 우선**이다. `AGENTS.md`는 도구 차이만 적는다.

---

## 1. 한 줄 그림

```
당신은 자연어로 말한다
        │
        ▼
 Claude Code          또는          Grok Build (CLI / VS Code Community)
        │                              │
        └──────────┬───────────────────┘
                   ▼
     CLAUDE.md + AGENTS.md + .claude/skills + .claude/agents
                   ▼
     같은 raw/ · wiki/ · index.md · log.md
                   ▼
              Obsidian 볼트
```

위키 작업이 들어오면 메인이 먼저 **wiki-ops**를 따른다. 오퍼레이션마다 전문가 **1명**. 두 도구가 동시에 같은 페이지를 쓰는 팀플레이가 아니다.

---

## 2. 준비물

| 항목 | 용도 |
|------|------|
| **이 폴더를 연 워크스페이스** | 세션·훅·스킬이 전부 cwd에 묶인다. 빈 창에서 시작하면 세션이 다른 버킷에 쌓인다 |
| **python3** | 훅·lint·신뢰도 스크립트. 표준 라이브러리만 사용 |
| **[Claude Code](https://claude.ai/code)** | 원래 하네스 런타임 |
| **Grok Build CLI** (`grok`) | 엔진. VS Code Community 확장도 이 CLI를 띄운다 |
| **로그인** | Claude는 Anthropic 계정. Grok은 SuperGrok / X Premium+ 또는 xAI API 키 (`grok login`) |
| **(권장) Obsidian** | `~/my-wiki`를 vault로 연다. 읽기·그래프용. 도구와 무관 |
| **(선택) Grok Build for VS Code (Community)** | 사이드바 채팅. 공식 xAI 확장이 아님 |

---

## 3. 이 볼트에서 최초 1회

이미 하네스를 받아 두고 위키가 있는 상태라면 시드 복사는 건너뛴다.

### 3-1. 폴더를 연다

VS Code / Cursor / Claude Code 어디서든 **File → Open Folder → `~/my-wiki`**.  
파일 하나만 연 빈 창, 또는 폴더 없는 창에서 Grok을 켜지 않는다. 세션이 `~/my-wiki`가 아닌 cwd에 저장된다.

### 3-2. 개인 프로필

없으면 `CLAUDE.local.md`를 만든다(gitignore). Claude와 Grok이 **둘 다** 로드한다.

```markdown
# CLAUDE.local.md
## 목적 / 사용자 프로필
- 역할:
- 스택:
- 관심/학습:
```

도구 전용 메모가 필요하면 `AGENTS.local.md`에 적는다(역시 gitignore). 스키마는 여기 넣지 않는다.

### 3-3. Grok 훅 trust

프로젝트 훅(`.claude/settings.json`)은 **trust 하기 전에는 조용히 스킵**된다.

```bash
cd ~/my-wiki
grok --trust
```

이미 세션이 열려 있으면 채팅에서 `/hooks-trust`.  
확인: `/hooks`에 SessionStart(`wiki-status-check.py`), PostToolUse(`stamp-updated.py`)가 보여야 한다.

### 3-4. Claude Code 훅

Claude Code에서 이 폴더를 연 뒤 **`/hooks`를 한 번** 연다. 세션 도중에 생긴 훅은 watcher가 바로 못 잡는 경우가 있다.

### 3-5. 동작 확인

```bash
cd ~/my-wiki
python3 .claude/skills/wiki-lint/scripts/wiki-status-check.py .
python3 .claude/skills/wiki-lint/scripts/ingest-status.py .
```

에러 없이 상태 한 줄이 나오면 스크립트 경로는 정상이다.

---

## 4. 매일 시작

말할 문장은 도구와 같다. 진입점만 다르다.

### 4-1. Grok — VS Code Community 확장

1. **반드시** `~/my-wiki` 폴더를 연다.
2. `Cmd+;` (Windows `Ctrl+;`) 또는 Command Palette → **Grok: Open**.
3. 자연어로 요청한다. 예: `raw/meetings/오늘.md 인제스트해줘`.
4. 같은 프로젝트 대화를 이어가려면 시계 아이콘에서 고른다.

확장 없이 쓰려면 터미널:

```bash
cd ~/my-wiki
grok
```

가장 최근 대화를 이어서: `grok --resume` 또는 `grok -c`.

### 4-2. Claude Code

```bash
cd ~/my-wiki
claude
```

IDE 확장을 쓰면 역시 이 폴더가 워크스페이스인지 확인한다.  
슬래시 예: `/wiki-ops`, `/wiki-ingest`, `/wiki-query`, `/wiki-lint`. 자연어도 동일하다.

### 4-3. Obsidian

Obsidian에서 `~/my-wiki` vault를 연다. 에이전트가 파일을 쓰면 그래프·미리보기가 바로 갱신된다. Obsidian은 편집 주체가 아니다.

---

## 5. 무엇을 말하면 되나

메인이 **wiki-ops**로 라우팅한다. 도구 이름을 말할 필요 없다.

| 하고 싶은 일 | 이렇게 말한다 |
|--------------|----------------|
| 소스 넣기 | `raw/….md 인제스트해줘` |
| 질문·비교 | `위키에서 X에 대해 알려줘` / `A와 B 비교해줘` |
| 건강 검진 | `위키 점검해줘` / `린트` |
| 지도 | `MoC 정리해줘` |
| 세션 끝 | `세션 마무리` / `L1 압축` |
| 주간 | `주간 리뷰` |
| 녹음 | `이 녹음 회의록 만들어줘` (민감 회의는 로컬 전사) |

전문가 이름(`wiki-ingestor` 등)은 에이전트 내부 디스패치용이다. 사용자가 외울 필요 없다.

| 의도 | 전문가 | 스킬 |
|------|--------|------|
| 인제스트 | `wiki-ingestor` | wiki-ingest |
| 질의 | `wiki-synthesizer` | wiki-query |
| 린트 | `wiki-linter` | wiki-lint |
| MoC | `wiki-cartographer` | wiki-moc |
| 압축·승격 | `wiki-consolidator` | wiki-consolidate |
| 회의록(전사 **후**) | `meeting-scribe` | meeting-minutes |

녹음 전사는 사람이 백엔드(로컬 vs OpenAI)를 고른 뒤에만 돌아간다. 심사·인사·계약은 로컬 강제.

---

## 6. 도구를 바꿔 이어받기

콘텐츠는 git에 안 올라간다. 같은 머신(또는 동기화된 볼트 폴더)을 공유하는 것이 전제다.

1. 작업을 마친 쪽이 `log.md`에 op를 남긴다 (`## [YYYY-MM-DD] <op> | <제목>`).
2. 다른 도구로 열기 전에:

   ```bash
   grep "^## \[" log.md | tail -5
   ```

3. 하네스(스킬·스키마)만 최신으로 맞출 때: `git pull`. `raw/`·`wiki/`·`index.md`·`log.md`는 로컬에 남는다.
4. 하네스를 고칠 때: Claude든 Grok이든 **정본 경로만** 고친다 (`.claude/`, `CLAUDE.md`, `AGENTS.md`). PR에도 그 경로만 올린다.

---

## 7. 동시에 쓰면 안 되는 것

- **같은 소스를 양쪽에서 동시에 인제스트**하지 않는다. 페이지가 두 벌로 생긴다.
- **같은 파일을 양쪽에서 동시에 편집**하지 않는다. 나중 쓰기가 이긴다.
- 한쪽이 인제스트·린트 중이면 다른 쪽은 **질의만** 하거나 기다린다.
- 빈 세션(사이드바만 열고 한 줄도 안 보냄)은 저장되지 않는다. 보낸 대화만 디스크에 남는다.

역할 나누기 예:

- 낮: Grok 사이드바에서 짧은 질의·초안
- 정리 타임: Claude 또는 Grok 중 **하나만** 인제스트·린트

어느 쪽이든 산출물 스키마는 같다.

---

## 8. 세션이 “안 남아요”

Grok 세션은 `~/.grok/sessions/<인코딩된-cwd>/<id>/`에 자동 저장된다. 시계 아이콘은 **지금 연 폴더**의 세션만 보여 준다.

- 폴더 없이 대화 → 다른 cwd 버킷에 저장 → `~/my-wiki`를 열면 목록에 없음
- 찾기: 사이드바 **Projects** 레일, 또는 그 cwd에서 `grok sessions list`
- 위키 파일 자체는 세션과 별개다. 인제스트가 끝난 페이지는 도구를 바꿔도 Obsidian에서 그대로 보인다

자세한 저장 규칙은 Grok 세션 문서와 같다. 이 볼트에서는 **항상 `~/my-wiki`를 연 뒤** 대화를 시작하면 된다.

---

## 9. 훅이 안 돌 때

SessionStart가 스킵되면 첫 위키 작업 전에 직접 돌린다.

```bash
python3 .claude/skills/wiki-lint/scripts/wiki-status-check.py .
python3 .claude/skills/wiki-lint/scripts/lint-due.py . 3
python3 .claude/skills/wiki-lint/scripts/ingest-status.py .
```

`updated:`가 안 찍히면 방금 고친 페이지 frontmatter의 `updated:`를 오늘 날짜로 맞춘다.

Grok에서 훅이 비어 있으면: 이 폴더를 연 상태인지 → `grok --trust` 또는 `/hooks-trust` → 세션 재시작.

---

## 10. 모델 티어

에이전트 파일의 `model: sonnet` / `opus`는 **선호 티어**다.

| 표기 | 의도 | Claude Code | Grok |
|------|------|-------------|------|
| `sonnet` | 잦은 추출·쓰기 (인제스트) | Sonnet 계열 | 세션 기본 모델 |
| `opus` | 병합·종합·감사 | Opus 계열 | 세션 기본, 더 강한 모델이 있으면 그쪽 |

해당 ID가 없으면 세션 기본 모델로 진행한다. 페이지 형식은 바꾸지 않는다.

---

## 11. 문제 해결

| 증상 | 원인 | 해결 |
|------|------|------|
| Grok이 위키 규칙을 모르는 것 같다 | 폴더를 안 열었거나 `AGENTS.md`/`CLAUDE.md` 미로드 | `~/my-wiki`를 Open Folder. `/session-info`로 cwd 확인 |
| 어제 Grok 대화가 시계에 없다 | cwd가 달랐음 | Projects 레일. 빈 창에서 하지 말 것 |
| SessionStart 알림 없음 | 훅 미trust / 미로드 | Grok: `--trust`. Claude: `/hooks` |
| `updated:` 안 바뀜 | 위와 동일 | 훅 로드 또는 수동 스탬프 |
| 같은 개념 페이지가 두 개 | 동시 인제스트 또는 재인제스트 실수 | 다음 린트에서 병합. 같은 소스는 한 도구만 |
| Grok이 `.grok/skills`를 만들자고 함 | 이중 정본 | 거절. `.claude/skills`만 수정 |
| 서브에이전트가 안 뜸 | 런타임 제한 | 메인이 `.claude/agents/<이름>.md` + 스킬을 읽고 인라인 실행. 산출물 경로는 같음 |
| 스크립트 실패 | python3 없음 / 다른 디렉터리 | `cd ~/my-wiki`, `python3 --version` |

---

## 12. 관련 문서

| 문서 | 언제 보나 |
|------|-----------|
| [INTRO.md](INTRO.md) | 위키가 뭔지 3분 |
| [README.md](README.md) | 재현·오퍼레이션·스키마 전체 |
| [CLAUDE.md](CLAUDE.md) | 페이지 규약 정본 |
| [AGENTS.md](AGENTS.md) | 에이전트용 브릿지 (이 매뉴얼의 기계 버전) |
| [README §6](README.md#6-사용법--5가지-오퍼레이션) | ingest / query / lint / moc / consolidate 상세 |

슬래시 명령은 설치한 CLI·Claude 버전에 따라 목록이 조금 다를 수 있다. 자연어로 wiki-ops가 받는 문장이면 충분하다.

```



