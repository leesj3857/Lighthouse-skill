---
description: Set Lighthouse audit categories (performance/accessibility/best-practices/seo).
argument-hint: "[p|a|bp|s|all]"
disable-model-invocation: true
---

## Load Config

```!
cat .claude/skills/lighthouse/config.json 2>/dev/null || echo '{"device":"desktop","mode":"navigation","categories":["performance","accessibility","best-practices","seo"],"throttling":"simulate","output":["json"],"outputPath":"/tmp/lh-run","locale":"en","language":"en","disableStorageReset":false,"disableFullPageScreenshot":false,"extraHeaders":{}}'
```

## Procedure

1. Read the current `categories` array from the JSON above.

2. If `$ARGUMENTS` contains category keywords (`p`, `performance`, `a`, `accessibility`, `bp`, `best-practices`, `s`, `seo`, `all`), save those values directly and skip step 3. `all` means all four categories.

3. Show AskUserQuestion:
   - question: "Select audit categories (current: {current values}, multi-select)"
   - header: "Categories"
   - multiSelect: true
   - options:
     - label: "Performance", description: "FCP · LCP · TBT · CLS · SI — Core Web Vitals and loading performance"
     - label: "Accessibility", description: "Color contrast · ARIA · keyboard navigation · focus management"
     - label: "Best Practices", description: "HTTPS · CSP · console errors · modern web standards"
     - label: "SEO", description: "Meta tags · structured data · crawlability"

4. Map selections to Lighthouse CLI category names:
   `Performance` → `performance` / `Accessibility` → `accessibility` / `Best Practices` → `best-practices` / `SEO` → `seo`
   If nothing selected, use all four.

5. Update the `categories` field in `.claude/skills/lighthouse/config.json`.

6. Print:
   `Saved → device: {value} | mode: {value} | categories: {value}`
