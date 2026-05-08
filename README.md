# Claude Code Skills

A collection of [Claude Code](https://claude.ai/code) slash-command skills for web development workflows.

> **Korean version:** [README.ko.md](./README.ko.md)

---

## Skills

### `/lighthouse` — Lighthouse Audit Suite

Runs a Google Lighthouse audit against your local dev server, analyzes failures, lets you select issues to fix, auto-applies code changes, rebuilds, re-measures, and saves a before/after comparison to `history.md`.

**Companion configuration commands:**

| Command | Description |
|---|---|
| `/lighthouse` | Run audit with current settings |
| `/lighthouse-config` | Show current settings / reset to defaults |
| `/lighthouse-device` | Switch between desktop and mobile |
| `/lighthouse-mode` | Switch between navigation / timespan / snapshot |
| `/lighthouse-categories` | Select which categories to audit |
| `/lighthouse-throttling` | Set throttling method (simulate / devtools / provided) |
| `/lighthouse-output` | Set output format, locale, history language, and screenshot options |

**Features:**
- Compact result parsing — keeps token usage low (pipe-separated text instead of raw JSON)
- Auto-diagnoses failures and suggests code-level fixes with file + line guidance
- Multi-select fix picker via `AskUserQuestion`
- Rebuilds dev server automatically after applying fixes
- Before/After score comparison with CWV diff
- Per-URL history directories: `/search` → `.claude/lighthouse/search/history.md`
- **Bilingual history.md** — choose `en` (English) or `kr` (Korean) via `/lighthouse-output`

---

## Installation

### Quick Install (Mac / Linux)

Paste the following into your terminal to install all skill files directly into `~/.claude/skills/`:

```bash
BASE="https://raw.githubusercontent.com/leesj3857/Lighthouse-skill/main/skills" && for s in lighthouse lighthouse-categories lighthouse-config lighthouse-device lighthouse-lang lighthouse-mode lighthouse-output lighthouse-throttling; do mkdir -p ~/.claude/skills/$s && curl -fsSL "$BASE/$s/SKILL.md" -o ~/.claude/skills/$s/SKILL.md; done && curl -fsSL "$BASE/lighthouse/config.json" -o ~/.claude/skills/lighthouse/config.json && echo "✅ Lighthouse skills installed → ~/.claude/skills/"
```

> After installing, start your dev server and run `/lighthouse` in Claude Code.

### Manual Install

### Requirements
- [Claude Code](https://claude.ai/code) CLI or desktop app
- `npx` (Node.js) — used to run `lighthouse` without a global install
- A local dev server running on port **4173** (Vite preview) or **3000**

### Steps

1. **Copy the `skills/` folder** into your project's `.claude/` directory:

   ```bash
   cp -r skills/* .claude/skills/
   ```

   Your project structure should look like:
   ```
   your-project/
   └── .claude/
       └── skills/
           ├── lighthouse/
           │   ├── SKILL.md
           │   └── config.json    ← persists your settings
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

2. **Add history directory to `.gitignore`** (keeps audit history out of version control):

   ```gitignore
   # Claude Code — Lighthouse audit history
   .claude/lighthouse/
   ```

3. **Start your dev server**, then run `/lighthouse` in Claude Code.

---

## Configuration

All settings are stored in `.claude/skills/lighthouse/config.json` and persist between sessions.

| Field | Default | Description |
|---|---|---|
| `device` | `desktop` | `desktop` or `mobile` |
| `mode` | `navigation` | `navigation`, `timespan`, or `snapshot` |
| `categories` | all 4 | `performance`, `accessibility`, `best-practices`, `seo` |
| `throttling` | `simulate` | `simulate`, `devtools`, or `provided` |
| `output` | `["json"]` | `json`, `html`, `csv` |
| `outputPath` | `/tmp/lh-run` | Temporary file path for Lighthouse JSON output |
| `locale` | `en` | Lighthouse report locale (`en`, `ko`, `ja`, `zh`) |
| `language` | `en` | history.md output language: `en` (English) or `kr` (Korean) |
| `disableStorageReset` | `false` | Keep browser cache between runs |
| `disableFullPageScreenshot` | `false` | Skip screenshot in HTML report |

---

## History Storage

Audit results are saved to `.claude/lighthouse/` (gitignored by default).

- Root URL `/` → `.claude/lighthouse/history.md`
- `/search` → `.claude/lighthouse/search/history.md`
- `/ranking/gyeonggi` → `.claude/lighthouse/ranking/gyeonggi/history.md`

History files record every run with scores, Core Web Vitals, and a collapsible failures section. Before/After comparisons are stored automatically when code fixes are applied.

---

## Example Output

```
✅ Lighthouse done (Before → After) — saved to .claude/lighthouse/history.md
Performance: 57 → 86 (+29 ✅)   SEO: 100 → 100 (→)
LCP: 3.3s → 2.6s   CLS: 0.891 → 0
Fixes applied: 3
```

---

## License

MIT
