# Claude Code 스킬 모음

웹 개발 워크플로우를 위한 [Claude Code](https://claude.ai/code) 슬래시 커맨드 스킬 모음입니다.

> **English version:** [README.md](./README.md)

---

## 스킬 목록

### `/lighthouse` — Lighthouse 감사 스위트

로컬 개발 서버에 Google Lighthouse 감사를 실행하고, 문제를 분석하며, 수정할 항목을 선택하면 코드를 자동 수정하고, 재빌드 후 재측정하여 before/after 비교 결과를 `history.md`에 저장합니다.

**보조 설정 커맨드:**

| 커맨드 | 설명 |
|---|---|
| `/lighthouse` | 현재 설정으로 감사 실행 |
| `/lighthouse-config` | 현재 설정 확인 / 기본값으로 초기화 |
| `/lighthouse-device` | 데스크탑 / 모바일 전환 |
| `/lighthouse-mode` | 측정 모드 전환 (navigation / timespan / snapshot) |
| `/lighthouse-categories` | 감사할 카테고리 선택 |
| `/lighthouse-throttling` | 스로틀링 방식 설정 (simulate / devtools / provided) |
| `/lighthouse-output` | 출력 형식, 로케일, 히스토리 언어, 스크린샷 옵션 설정 |

**주요 기능:**
- 컴팩트 결과 파싱 — 토큰 사용량 최소화 (원시 JSON 대신 파이프 구분 텍스트)
- 실패 항목 자동 진단 및 파일·라인 기준 코드 수정 제안
- `AskUserQuestion`을 통한 다중 선택 수정 UI
- 수정 후 개발 서버 자동 재빌드
- CWV diff 포함 Before/After 점수 비교
- URL별 히스토리 디렉터리: `/search` → `.claude/lighthouse/search/history.md`
- **히스토리 이중 언어** — `/lighthouse-output`에서 `en`(영어) 또는 `kr`(한국어) 선택 가능

---

## 설치

### 요구 사항
- [Claude Code](https://claude.ai/code) CLI 또는 데스크탑 앱
- `npx` (Node.js) — 전역 설치 없이 `lighthouse` 실행
- 포트 **4173** (Vite preview) 또는 **3000**에서 실행 중인 로컬 개발 서버

### 설치 단계

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

2. **`.gitignore`에 히스토리 디렉터리 추가** (감사 히스토리를 버전 관리에서 제외):

   ```gitignore
   # Claude Code — Lighthouse 감사 히스토리
   .claude/lighthouse/
   ```

3. **개발 서버를 시작**한 후 Claude Code에서 `/lighthouse`를 실행하세요.

---

## 설정

모든 설정은 `.claude/skills/lighthouse/config.json`에 저장되며 세션 간에 유지됩니다.

| 필드 | 기본값 | 설명 |
|---|---|---|
| `device` | `desktop` | `desktop` 또는 `mobile` |
| `mode` | `navigation` | `navigation`, `timespan`, `snapshot` |
| `categories` | 전체 4개 | `performance`, `accessibility`, `best-practices`, `seo` |
| `throttling` | `simulate` | `simulate`, `devtools`, `provided` |
| `output` | `["json"]` | `json`, `html`, `csv` |
| `outputPath` | `/tmp/lh-run` | Lighthouse JSON 출력용 임시 파일 경로 |
| `locale` | `en` | Lighthouse 리포트 로케일 (`en`, `ko`, `ja`, `zh`) |
| `language` | `en` | history.md 출력 언어: `en`(영어) 또는 `kr`(한국어) |
| `disableStorageReset` | `false` | 실행 간 브라우저 캐시 유지 |
| `disableFullPageScreenshot` | `false` | HTML 리포트에서 스크린샷 생략 |

---

## 히스토리 저장 위치

감사 결과는 `.claude/lighthouse/`에 저장됩니다 (기본적으로 gitignore 처리).

- 루트 URL `/` → `.claude/lighthouse/history.md`
- `/search` → `.claude/lighthouse/search/history.md`
- `/ranking/gyeonggi` → `.claude/lighthouse/ranking/gyeonggi/history.md`

히스토리 파일에는 모든 실행 결과(점수, Core Web Vitals, 접을 수 있는 실패 상세 섹션)가 기록됩니다. 코드 수정이 적용된 경우 Before/After 비교가 자동으로 저장됩니다.

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
