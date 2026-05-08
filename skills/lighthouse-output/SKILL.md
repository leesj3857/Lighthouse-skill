---
description: Set Lighthouse output format, Lighthouse locale, history language, storage reset, and screenshot options.
disable-model-invocation: true
---

## Load Config

```!
cat .claude/skills/lighthouse/config.json 2>/dev/null || echo '{"device":"desktop","mode":"navigation","categories":["performance","accessibility","best-practices","seo"],"throttling":"simulate","output":["json"],"outputPath":"/tmp/lh-run","locale":"en","language":"en","disableStorageReset":false,"disableFullPageScreenshot":false,"extraHeaders":{}}'
```

## Procedure

1. Read the current `output`, `locale`, `disableStorageReset`, and `disableFullPageScreenshot` values from the JSON above.

2. **Output format** — show AskUserQuestion:
   - question: "Select output format(s) (current: {current value}, multi-select)"
   - header: "Output Format"
   - multiSelect: true
   - options:
     - label: "json", description: "Save JSON report. Required for history.md recording. Always recommended."
     - label: "html", description: "Save HTML report viewable in a browser. (--output=html)"
     - label: "csv", description: "Save CSV report for spreadsheet analysis. (--output=csv)"

3. **Lighthouse locale** — show AskUserQuestion:
   - question: "Select Lighthouse report locale (current: {current value})"
   - header: "Locale"
   - multiSelect: false
   - options:
     - label: "en", description: "English (--locale=en)"
     - label: "ko", description: "Korean (--locale=ko)"
     - label: "ja", description: "Japanese (--locale=ja)"
     - label: "zh", description: "Simplified Chinese (--locale=zh)"

4. **Storage reset** — show AskUserQuestion:
   - question: "Reset browser cache & storage before each measurement? (current: {current value})"
   - header: "Storage Reset"
   - multiSelect: false
   - options:
     - label: "Reset (recommended)", description: "Start each run with no cache. Produces consistent results."
     - label: "Keep cache", description: "Preserve existing cache. Useful for measuring return-visit performance. (--disable-storage-reset)"

5. **Full-page screenshot** — show AskUserQuestion:
   - question: "Include full-page screenshot? (current: {current value})"
   - header: "Screenshot"
   - multiSelect: false
   - options:
     - label: "Include (recommended)", description: "Embed full-page screenshot in HTML report. Increases JSON file size."
     - label: "Exclude", description: "Skip screenshot. Reduces JSON file size. (--disable-full-page-screenshot)"

6. Save all selections to `.claude/skills/lighthouse/config.json`:
   - `output`: array of selected formats. If empty, keep `["json"]`.
   - `locale`: selected locale
   - `disableStorageReset`: `true` if "Keep cache" selected, otherwise `false`
   - `disableFullPageScreenshot`: `true` if "Exclude" selected, otherwise `false`

7. Print:
   `Saved → output: {value} | locale: {value} | storage-reset: {value} | screenshot: {value}`
