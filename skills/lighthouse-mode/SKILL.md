---
description: Set Lighthouse measurement mode (navigation/timespan/snapshot).
argument-hint: "[navigation|timespan|snapshot]"
disable-model-invocation: true
---

## Load Config

```!
cat .claude/skills/lighthouse/config.json 2>/dev/null || echo '{"device":"desktop","mode":"navigation","categories":["performance","accessibility","best-practices","seo"],"throttling":"simulate","output":["json"],"outputPath":"/tmp/lh-run","locale":"en","language":"en","disableStorageReset":false,"disableFullPageScreenshot":false,"extraHeaders":{}}'
```

## Procedure

1. Read the current `mode` value from the JSON above.

2. If `$ARGUMENTS` contains `navigation`, `timespan`, or `snapshot`, save that value directly and skip step 3.

3. Show AskUserQuestion:
   - question: "Select measurement mode (current: {current value})"
   - header: "Mode"
   - multiSelect: false
   - options:
     - label: "Navigation", description: "Measures initial page load. Supports Performance & CWV. Most common mode."
     - label: "Timespan", description: "Measures interactions over a time period. Best for INP · TBT runtime performance."
     - label: "Snapshot", description: "Snapshot of current page state. Quick accessibility/SEO check on an already-loaded page."

4. Save the selection (lowercase) to the `mode` field in `.claude/skills/lighthouse/config.json`.

5. Print:
   `Saved → device: {value} | mode: {value} | categories: {value}`
