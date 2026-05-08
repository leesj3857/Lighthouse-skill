---
description: Set Lighthouse network/CPU throttling method (simulate/devtools/provided).
argument-hint: "[simulate|devtools|provided]"
disable-model-invocation: true
---

## Load Config

```!
cat .claude/skills/lighthouse/config.json 2>/dev/null || echo '{"device":"desktop","mode":"navigation","categories":["performance","accessibility","best-practices","seo"],"throttling":"simulate","output":["json"],"outputPath":"/tmp/lh-run","locale":"en","language":"en","disableStorageReset":false,"disableFullPageScreenshot":false,"extraHeaders":{}}'
```

## Procedure

1. Read the current `throttling` value from the JSON above.

2. If `$ARGUMENTS` contains `simulate`, `devtools`, or `provided`, save that value directly and skip step 3.

3. Show AskUserQuestion:
   - question: "Select throttling method (current: {current value})"
   - header: "Throttling"
   - multiSelect: false
   - options:
     - label: "simulate", description: "Lighthouse software-simulates network & CPU. Stable, reproducible results. Recommended default. (--throttling-method=simulate)"
     - label: "devtools", description: "Chrome DevTools applies real network & CPU limits. Closer to real conditions but with variance. (--throttling-method=devtools)"
     - label: "provided", description: "No throttling. Uses current network & CPU as-is. Fastest but environment-dependent. (--throttling-method=provided)"

4. Save the selection to the `throttling` field in `.claude/skills/lighthouse/config.json`.

5. Print:
   `Saved → throttling: {value} | device: {value} | mode: {value}`
