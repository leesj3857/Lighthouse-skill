---
description: Show current Lighthouse configuration or reset to defaults.
argument-hint: "[reset]"
disable-model-invocation: true
---

## Load Config

```!
cat .claude/skills/lighthouse/config.json 2>/dev/null || echo '{"device":"desktop","mode":"navigation","categories":["performance","accessibility","best-practices","seo"],"throttling":"simulate","output":["json"],"outputPath":"/tmp/lh-run","locale":"en","language":"en","disableStorageReset":false,"disableFullPageScreenshot":false,"extraHeaders":{}}'
```

## Procedure

1. If `$ARGUMENTS` contains `reset`, overwrite `.claude/skills/lighthouse/config.json` with the defaults below, print "Configuration reset to defaults." and stop.

   Default config:
   ```json
   {
     "device": "desktop",
     "mode": "navigation",
     "categories": ["performance", "accessibility", "best-practices", "seo"],
     "throttling": "simulate",
     "output": ["json"],
     "outputPath": "/tmp/lh-run",
     "locale": "en",
     "language": "en",
     "disableStorageReset": false,
     "disableFullPageScreenshot": false,
     "extraHeaders": {}
   }
   ```

2. Read values from the loaded JSON and print:

```
Current Lighthouse Configuration
──────────────────────────────────────────
  device                : {value}
  mode                  : {value}
  categories            : {value comma-separated}
  throttling            : {value}
  output                : {value}
  locale                : {value}
  language              : {value}
  disable-storage-reset : {value}
  disable-screenshot    : {value}
  extra-headers         : {value or "(none)"}
──────────────────────────────────────────
Change settings:
  /lighthouse-device       → device (desktop / mobile)
  /lighthouse-mode         → mode (navigation / timespan / snapshot)
  /lighthouse-categories   → categories
  /lighthouse-throttling   → throttling (simulate / devtools / provided)
  /lighthouse-output       → output format, locale, storage, screenshot
  /lighthouse-lang         → history.md language (en / kr)
  /lighthouse-config reset → reset to defaults
  /lighthouse              → run audit with current settings
```
