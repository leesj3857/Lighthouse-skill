# Claude Code 스킬 모음

웹 개발에 바로 쓸 수 있는 [Claude Code](https://claude.ai/code) 슬래시 커맨드 스킬 모음입니다.

> **English version:** [README.md](./README.md)

---

## 스킬 목록

### `/lighthouse` — Lighthouse 검사 도구

로컬 개발 서버에 Google Lighthouse 검사를 실행하고, 문제를 분석한 뒤 수정할 항목을 고르면 코드를 자동으로 수정합니다. 수정 후 재빌드 및 재측정까지 진행하며, Before/After 비교 결과를 `history.md`에 저장합니다.

**설정 커맨드:**

| 커맨드 | 설명 |
|---|---|
| `/lighthouse` | 현재 설정으로 검사 실행 |
| `/lighthouse-config` | 현재 설정 확인 / 기본값으로 초기화 |
| `/lighthouse-device` | 데스크탑 / 모바일 전환 |
| `/lighthouse-mode` | 측정 모드 전환 (navigation / timespan / snapshot) |
| `/lighthouse-categories` | 검사할 카테고리 선택 |
| `/lighthouse-throttling` | 스로틀링 방식 설정 (simulate / devtools / provided) |
| `/lighthouse-output` | 출력 형식, 로케일, 히스토리 언어, 스크린샷 옵션 설정 |

**주요 기능:**
- 결과를 압축 파싱해 토큰 사용량을 최소화 (원시 JSON 대신 파이프 구분 텍스트)
- 실패 항목 자동 진단 및 파일·라인 단위 수정 방향 제안
- `AskUserQuestion`을 통한 수정 항목 다중 선택 UI
- 수정 완료 후 개발 서버 자동 재빌드
- Core Web Vitals 포함 Before/After 점수 비교
- URL별 히스토리 저장: `/search` → `.claude/lighthouse/search/history.md`
- **히스토리 이중 언어 지원** — `/lighthouse-output`에서 `en`(영어) 또는 `kr`(한국어) 선택 가능

---

## 설치

### 한 줄 설치 (Mac / Linux)

터미널에 아래 명령어를 붙여넣으면 모든 스킬 파일이 `~/.claude/skills/`에 바로 설치됩니다:

```bash
BASE="https://raw.githubusercontent.com/leesj3857/Lighthouse-skill/main/skills" && for s in lighthouse lighthouse-categories lighthouse-config lighthouse-device lighthouse-lang lighthouse-mode lighthouse-output lighthouse-throttling; do mkdir -p ~/.claude/skills/$s && curl -fsSL "$BASE/$s/SKILL.md" -o ~/.claude/skills/$s/SKILL.md; done && curl -fsSL "$BASE/lighthouse/config.json" -o ~/.claude/skills/lighthouse/config.json && echo "✅ Lighthouse 스킬 설치 완료 → ~/.claude/skills/"
```

> 설치 후 개발 서버를 켜고 Claude Code에서 `/lighthouse`를 실행하면 바로 사용할 수 있습니다.

### 수동 설치

### 요구사항
- [Claude Code](https://claude.ai/code) CLI 또는 데스크탑 앱
- `npx` (Node.js) — 전역 설치 없이 `lighthouse` 실행 가능
- 포트 **4173** (Vite preview) 또는 **3000**에서 실행 중인 로컬 개발 서버

### 설치 방법

1. **`skills/` 폴더**를 프로젝트의 `.claude/` 디렉터리에 복사합니다:

   ```bash
   cp -r skills/* .claude/skills/
   ```

   복사 후 프로젝트 구조:
   ```
   your-project/
   └── .claude/
       └── skills/
           ├── lighthouse/
           │   ├── SKILL.md
           │   └── config.json    ← 설정이 여기에 저장됨
           ├── lighthouse-config/
           │   └── SKILL.md
           ├── lighthouse-device/
           │   └── SKILL.md
           ├── lighthouse-categories/
           │   └── SKILL.md
           ├── lighthouse-mode/
           │   └── SKILL.md
           ├── lighthouse-output/
           │   └── SKILL.md
           └── lighthouse-throttling/
               └── SKILL.md
   ```

2. **`.gitignore`에 히스토리 디렉터리 추가** (검사 히스토리를 버전 관리에서 제외):

   ```gitignore
   # Claude Code — Lighthouse 검사 히스토리
   .claude/lighthouse/
   ```

3. **개발 서버를 시작**한 후 Claude Code에서 `/lighthouse`를 실행하세요.

---

## 설정

모든 설정은 `.claude/skills/lighthouse/config.json`에 저장되며 세션이 끊겨도 유지됩니다.

| 필드 | 기본값 | 설명 |
|---|---|---|
| `device` | `desktop` | `desktop` 또는 `mobile` |
| `mode` | `navigation` | `navigation`, `timespan`, `snapshot` |
| `categories` | 전체 4개 | `performance`, `accessibility`, `best-practices`, `seo` |
| `throttling` | `simulate` | `simulate`, `devtools`, `provided` |
| `output` | `["json"]` | `json`, `html`, `csv` |
| `outputPath` | `/tmp/lh-run` | Lighthouse JSON 결과 임시 파일 경로 |
| `locale` | `en` | Lighthouse 리포트 언어 (`en`, `ko`, `ja`, `zh`) |
| `language` | `en` | history.md 출력 언어: `en`(영어) 또는 `kr`(한국어) |
| `disableStorageReset` | `false` | 실행 사이 브라우저 캐시 유지 |
| `disableFullPageScreenshot` | `false` | HTML 리포트에서 전체 페이지 스크린샷 생략 |

---

## 히스토리 저장 위치

검사 결과는 `.claude/lighthouse/`에 저장됩니다 (기본적으로 gitignore 처리).

- 루트 URL `/` → `.claude/lighthouse/history.md`
- `/search` → `.claude/lighthouse/search/history.md`
- `/ranking/gyeonggi` → `.claude/lighthouse/ranking/gyeonggi/history.md`

히스토리 파일에는 매 실행의 점수, Core Web Vitals, 접을 수 있는 실패 상세 내역이 기록됩니다. 코드 수정이 적용된 경우 Before/After 비교가 자동으로 함께 저장됩니다.

---

## 출력 예시

```
✅ Lighthouse 완료 (Before → After) — .claude/lighthouse/history.md 저장됨
Performance: 57 → 86 (+29 ✅)   SEO: 100 → 100 (→)
LCP: 3.3s → 2.6s   CLS: 0.891 → 0
수정: 3건 적용
```

---

## 라이선스

MIT
