---
name: cli-tools
description: Use when running CLI tools documented in this index, including Claude Code and Codex.
metadata:
  source: https://github.com/dsl400/skill-cli-tools
  version: 0.1.0
---

# CLI tools

| Tool | Reference | Use when |
|---|---|---|
| Claude Code | [claude.md](claude.md) | Running Claude CLI or handing it a task. |
| Codex | [codex.md](codex.md) | Running Codex CLI or splitting work between Claude and Codex. |

```lnl
START:
  DO    select the tool matching the user's request from the index
  ASK   is the tool listed?
  STOP  report that the index has no reference for the requested tool IF no
  DO    read the linked file next to this skill and follow its instructions
REVIEW:
  CHECK the reference matches the requested tool
REPLY:
  SAY   the result and any open diagnostic
  END
```
