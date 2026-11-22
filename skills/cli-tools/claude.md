# Claude CLI

Check the installed interface with `claude --version` and `claude --help`.
Run from the target project's directory; `--add-dir` grants access to an
additional directory.

## Invocation

For an unattended run, read the prompt from a file and retain both outputs:

```sh
claude --print --model '<model>' --permission-mode dontAsk \
  --allowedTools 'Read,Glob,Grep' \
  < '<prompt.md>' > '<result.md>' 2> '<stderr.log>'
```

Replace `<model>` with the requested model or omit `--model` to use the
configured default. Include the task, relevant files, constraints and
expected checks in the prompt.

## Checks

| Check | Filler | Direct |
|---|---|---|
| Non-interactive execution | Run Claude in automation. | Use `--print`; supply stdin from a file or close it with `</dev/null` when the prompt is an argument. |
| Tool permissions | Give Claude the tools it needs. | Use `--allowedTools 'Read,Edit,Bash(npm test *)'` for an edit requiring that test command. |
| Result evidence | Claude says the task succeeded. | Check the exit status, saved output, diff and requested validation. |

`--allowedTools` pre-approves matching calls. `--tools` selects the available
built-in tools. `--permission-mode dontAsk` denies calls that would otherwise
require a prompt; report blocked actions with the requested permission.

| Option | Use |
|---|---|
| `--output-format json` | Capture a structured result and session ID. |
| `--output-format stream-json --verbose` | Capture events as JSON lines. |
| `--resume '<session-id>'` | Continue a specific session. |

## Flow

```lnl
START:
  DO    check the installed CLI and select the project directory
  DO    prepare the prompt, output paths and permissions for the authorized task
  DO    run claude --print and retain its exit status, stdout and stderr
  ASK   did the run fail or report a blocked action?
  STOP  report the diagnostic and any permission needed IF yes
REVIEW:
  CHECK the result is supported by the output and, for edits, the diff and requested checks
REPLY:
  SAY   the result, artifact paths and any unverified claim
  END
```

## Sources

[Programmatic execution](https://code.claude.com/docs/en/headless) and
[CLI reference](https://code.claude.com/docs/en/cli-reference).
Options checked against local Claude Code 2.1.310 on 2026-11-22.
