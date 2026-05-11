---
description: Run Lighthouse audit → analyze issues → auto-fix code → re-measure → before/after comparison. Audits performance, accessibility, best-practices, and SEO.
argument-hint: "[/path] [desktop|mobile] [all|p|a|bp|s] [html]"
allowed-tools: Bash, Read, Edit, Write
---

## Load Config

```!
cat .claude/skills/lighthouse/config.json 2>/dev/null || echo '{"device":"desktop","mode":"navigation","categories":["performance","accessibility","best-practices","seo"],"throttling":"simulate","output":["json"],"outputPath":"/tmp/lh-run","language":"en","disableStorageReset":false,"disableFullPageScreenshot":false,"extraHeaders":{}}'
```

## 1. Parse Settings & Arguments

Read device, mode, categories, throttling, outputPath, language from the JSON above.

Token overrides from `$ARGUMENTS` (this run only):
- `/path` or `http(s)://` → URL
- `desktop` / `mobile` → device
- `navigation` / `timespan` / `snapshot` → mode
- `p`/`performance`, `a`/`accessibility`, `bp`/`best-practices`, `s`/`seo`, `all` → categories
- `simulate` / `devtools` / `provided` → throttling
- `html` → add html to output formats

If no URL specified, use root `/`.

**Determine HISTORY_DIR** from the URL path:
- URL path `/` or empty → `.claude/lighthouse/`
- URL path `/search` → `.claude/lighthouse/search/`
- URL path `/ranking/gyeonggi` → `.claude/lighthouse/ranking/gyeonggi/`

Rule: strip leading/trailing `/` from the path and append to `.claude/lighthouse/`.

```bash
mkdir -p <HISTORY_DIR>
```

All subsequent `history.md` references use `<HISTORY_DIR>/history.md`.

## 2. Server Check

Check ports 4173 → 3000 in order:
```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:4173/
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/
```
If neither responds, print build/serve instructions and **stop**. Remember the active port as BASE_URL.

## 3. Run Lighthouse (BEFORE)

Print `Measuring [BEFORE]: <URL> | <device> | <categories>` then run:

```bash
npx lighthouse <URL> \
  --output=json --output-path=<outputPath> \
  --chrome-flags="--headless --no-sandbox --disable-dev-shm-usage" \
  --only-categories=<categories comma-separated> \
  --throttling-method=<throttling> \
  --locale=<language> --quiet \
  [--preset=desktop  if device=desktop]
```

## 4. Parse BEFORE Results

Run the Python below via Bash. **Keep output under 3 KB**:

```python
import json,datetime,os
cfg_path='<outputPath>'
p=cfg_path if os.path.exists(cfg_path) else cfg_path+'.report.json'
d=json.load(open(p))
cfg=d.get('configSettings',{})
ft=d.get('fetchTime','')
try:
  kst=datetime.datetime.fromisoformat(ft.replace('Z','+00:00'))+datetime.timedelta(hours=9)
  print('TIME|'+kst.strftime('%Y-%m-%d %H:%M'))
except: print('TIME|'+ft)
print(f"ENV|{cfg.get('formFactor')}|{d.get('gatherMode')}|{cfg.get('throttlingMethod')}")
cats=d.get('categories',{})
audits=d.get('audits',{})
for cid,c in cats.items():
  print(f"SCORE|{cid}|{int(round(c.get('score',0)*100))}")
for k,lbl in [('first-contentful-paint','FCP'),('largest-contentful-paint','LCP'),
              ('total-blocking-time','TBT'),('cumulative-layout-shift','CLS'),
              ('speed-index','SI'),('interactive','TTI')]:
  print(f"CWV|{lbl}|{audits.get(k,{}).get('displayValue','—')}")
for cid,c in cats.items():
  for ref in c.get('auditRefs',[]):
    aid=ref.get('id'); a=audits.get(aid,{}); sc=a.get('score')
    if sc is not None and sc<0.9:
      dv=(a.get('displayValue','') or '').replace('\n',' ')[:60]
      print(f"FAIL|{cid}|{aid}|{int(round(sc*100))}|{dv}")
```

Store parsed output as **BEFORE data**.

## 5. Compare with Previous History

Read the first `## ` section score table from `<HISTORY_DIR>/history.md` and calculate diffs.
If the file doesn't exist, treat all diffs as `—`.

Change notation: rise `+N ✅` / drop `-N ❌` / same `→` / no prior `—`

## 6. Diagnose Issues & List Fixes

Analyze FAIL lines and split into **code-fixable** and **non-code** items.

**Severity:**
- 🔴 CLS > 0.1, LCP > 4s
- 🟡 color-contrast, tabindex, aria, target-size
- 🟢 fetchpriority, image attributes

**Item format:**
```
N. 🔴/🟡/🟢 [audit-id] Title
   Current: <value>  Cause: <one line>  Fix: <file> — <change>
```

**Known fix patterns for this project:**
- `lcp-discovery-insight` → PlaceCardCover.svelte `fetchpriority="high"` (check if already applied)
- `cumulative-layout-shift` / `layout-shifts` → fixed header min-height, font-display:optional
- `tabindex` → Search.svelte remove `tabindex="1"`
- `target-size` → Search.svelte input height ≥ 24px
- `label-content-name-mismatch` → Footer.svelte include visible text in aria-label
- `color-contrast` → Footer.svelte darken textBtn-gray70, fix write-post button contrast
- `image-aspect-ratio` → add aspect-ratio or width/height to img
- `unsized-images` → add width/height to image component

Non-code items:
```
● [non-code] cache-insight: Set Cache-Control on CDN
● [non-code] image-delivery-insight: Enable WebP/AVIF on CDN
```

## 7. Select Fix Items

Use AskUserQuestion:
- question: "Select issues to fix (multi-select)"
- header: "Fix Items"
- multiSelect: true
- options: code-fixable items from step 6 (up to 4, highest severity first)
  - label: "N. [audit-id] Title"
  - description: "filename — change description"

**If nothing selected → jump to step 10 (save history). Skip re-measurement.**

## 8. Apply Code Fixes

For each selected item in order:
1. Read the file
2. Edit the fix
3. Print `[Done] filename: change description`

If the file structure differs from expectation, skip and print a one-line reason.

## 9. Rebuild & AFTER Measurement

Run the following only if **at least one fix was applied**.

**9-1. Kill existing server & rebuild:**
```bash
lsof -ti:<BASE_URL port> | xargs kill -9 2>/dev/null; true
pnpm run build 2>&1 | tail -3
```

**9-2. Restart preview server & wait:**
```bash
pnpm run preview --port <BASE_URL port> &
sleep 4
curl -s -o /dev/null -w "%{http_code}" <BASE_URL>
```
If not 200, retry after `sleep 3` (max 2 times).

**9-3. Run Lighthouse AFTER:**
Print `Measuring [AFTER]: <URL>` then run:
```bash
npx lighthouse <URL> \
  --output=json --output-path=<outputPath>-after \
  --chrome-flags="--headless --no-sandbox --disable-dev-shm-usage" \
  --only-categories=<categories> \
  --throttling-method=<throttling> \
  --locale=<language> --quiet \
  [--preset=desktop]
```

**9-4. Parse AFTER results:**
Run the same Python script from step 4 against `<outputPath>-after`.
Store as **AFTER data**.

## 10. Write history.md

Save to `<HISTORY_DIR>/history.md`. If the file doesn't exist, create it with this header:

```markdown
# Lighthouse History — /<url-path>

> Auto-generated by `/lighthouse /<url-path>`. Latest result at the top.

---
```

Use **language** from config to determine all labels (`ko` → Korean, anything else → English):

| Element | `en` | `ko` |
|---|---|---|
| Category column | Category | 카테고리 |
| Score column | Score | 점수 |
| Previous column | Previous | 이전 |
| Change column | Change | 변화 |
| Before column | Before | Before |
| After column | After | After |
| Applied fixes | Applied fixes | 수정 적용 |
| Failures summary | failures | 미통과 상세 |
| No failures | No failures ✅ | 미통과: 없음 ✅ |

If fixes were applied, use **Before/After format**; otherwise use **single-run format**.

### With fixes (Before/After format):

```markdown
## {TIME_BEFORE} → {TIME_AFTER} | {device} | {mode} | {throttling} | {categories}

| Category | Before | After | Change |
|---|---|---|---|
| Performance | 86 | 91 | +5 ✅ |
| SEO | 100 | 100 | → |

**Before CWV:** FCP 0.5s · LCP 2.6s · TBT 0ms · CLS 0.029 · SI 0.5s · TTI 2.6s
**After CWV:**  FCP 0.5s · LCP 2.1s · TBT 0ms · CLS 0.012 · SI 0.5s · TTI 2.1s

Applied fixes: `item1`, `item2`

<details><summary>AFTER failures (Performance 5)</summary>

- `largest-contentful-paint` [55] — 2.1s
- ...

</details>

---
```

### Without fixes (single-run format):

```markdown
## {TIME} | {device} | {mode} | {throttling} | {categories}

| Category | Score | Previous | Change |
|---|---|---|---|
| Performance | 86 | 57 | +29 ✅ |

CWV: FCP 0.5s · LCP 2.6s · TBT 0ms · CLS 0.029 · SI 0.5s · TTI 2.6s

<details><summary>Failures (Performance 7)</summary>

- `largest-contentful-paint` [45] — 2.6s
- ...

</details>

---
```

**Common rules:**
- Bold any CWV value where LCP > 2.5s or CLS > 0.1
- Omit `<details>` block and write `No failures ✅` when there are none
- When language is `ko`, replace all English labels with their Korean equivalents from the table above

## 11. Print Summary

Without fixes:
```
✅ Lighthouse done — saved to <HISTORY_DIR>/history.md
Score: Performance 86(+29) · SEO 100(→)
CWV:   FCP 0.5s · LCP 2.6s · CLS 0.029
Fixes: none (N non-code improvements suggested)
```

With fixes:
```
✅ Lighthouse done (Before → After) — saved to <HISTORY_DIR>/history.md
Performance: 86 → 91 (+5 ✅)   SEO: 100 → 100 (→)
LCP: 2.6s → 2.1s   CLS: 0.029 → 0.012
Fixes applied: N
```
