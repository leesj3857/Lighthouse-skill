---
description: Set the language used when writing history.md (en or kr).
argument-hint: "[en|kr]"
disable-model-invocation: true
---

## Load Config

```!
cat .claude/skills/lighthouse/config.json 2>/dev/null || echo '{"device":"desktop","mode":"navigation","categories":["performance","accessibility","best-practices","seo"],"throttling":"simulate","output":["json"],"outputPath":"/tmp/lh-run","locale":"en","language":"en","disableStorageReset":false,"disableFullPageScreenshot":false,"extraHeaders":{}}'
```

## Procedure

1. Read the current `language` value from the JSON above.

2. If `$ARGUMENTS` contains `en` or `kr`, save that value directly and skip step 3.

3. Show AskUserQuestion:
   - question: "Select history.md output language (current: {current value})"
   - header: "History Lang"
   - multiSelect: false
   - options:
     - label: "en", description: "Write history.md in English — Category, Score, Applied fixes, failures, etc."
     - label: "kr", description: "Write history.md in Korean — 카테고리, 점수, 수정 적용, 미통과 상세, etc."

4. Save the selection to the `language` field in `.claude/skills/lighthouse/config.json`.

5. Print:
   `Saved → language: {value}`
