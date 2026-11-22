---
name: codex
description: Use when the user asks to hand work to Codex, to run the Codex CLI, or to split a task between Claude and Codex.
---

# codex

The Codex CLI ships inside the OpenAI VSCode extension; call it by its path:

```bash
C=$(ls -d ~/.vscode/extensions/openai.chatgpt-*-linux-x64 | sort -V | tail -1)/bin/linux-x86_64/codex
$C --version        # codex-cli 0.171.0-alpha.13 on 2026-11-20
```

The model the user picked is `gpt-6.2-sol` at `medium` effort. `~/.codex/config.toml`
holds another default model, so each run names both:

```bash
$C exec -m gpt-6.2-sol -c model_reasoning_effort=medium \
  -s <sandbox> -C <dir> -o <scratchpad>/codex-<task>.md "<prompt>" </dev/null >/dev/null 2>&1
```

`</dev/null` closes stdin: `codex exec` reads extra input from an open stdin,
and a background shell keeps it open, so the run waits forever at
`Reading additional input from stdin...`.

| Flag | Use |
|---|---|
| `-s read-only` | a question or a review; Codex changes no file |
| `-s workspace-write` | a change; run it in a git worktree of its own |
| `-C <dir>` | the directory Codex works in |
| `-o <file>` | Codex writes its last message there; read it after the run |
| `--json` | the event stream as JSON lines on stdout, when the steps matter |

Codex starts with the prompt and the `AGENTS.md` of the directory it works in.
The prompt names the issue, the skill to follow and the files.

```lnl
START:
  ASK   does the task change files?
  JUMP  WORKTREE IF yes
  DO    run codex exec with -s read-only -C <repo root>
  JUMP  RESULT
WORKTREE:
  DO    create a git worktree under ../ryzolviam-codex-<task> on a branch codex/<task>
  DO    run codex exec with -s workspace-write -C <that worktree>, in the background when the run can take minutes
RESULT:
  DO    read the -o file and the git diff of the worktree
  ASK   does the result hold a claim the diff or a command cannot confirm?
  STOP  report the claim and what it lacks IF yes
REVIEW:
  CHECK each change of Codex is read before it reaches main
  CHECK the merge or the commit of the work of Codex waits for the user
REPLY:
  SAY   what Codex did, the -o file, the diff, and what is still to check
  END
```
