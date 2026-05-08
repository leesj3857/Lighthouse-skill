---
description: Set Lighthouse measurement device (desktop/mobile).
argument-hint: "[desktop|mobile]"
disable-model-invocation: true
---

## Load Config

```!
cat .claude/skills/lighthouse/config.json 2>/dev/null || echo '{"device":"desktop","mode":"navigation","categories":["performance","accessibility","best-practices","seo"],"throttling":"simulate","output":["json"],"outputPath":"/tmp/lh-run","locale":"en","language":"en","disableStorageReset":false,"disableFullPageScreenshot":false,"extraHeaders":{}}'
```

## Procedure

1. Read the current `device` value from the JSON above.

2. If `$ARGUMENTS` contains `desktop` or `mobile`, save that value directly and skip step 3.

3. Show AskUserQuestion:
   - question: "Select measurement device (current: {current value})"
   - header: "Device"
   - multiSelect: false
   - options:
     - label: "Desktop", description: "1350×940 viewport, CPU 1x — uses --preset=desktop"
     - label: "Mobile", description: "360×640 viewport, CPU 4x slowdown — Lighthouse default"

4. Save the selection (lowercase) to the `device` field in `.claude/skills/lighthouse/config.json`.

5. Print:
   `Saved → device: {value} | mode: {value} | categories: {value}`
