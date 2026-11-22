# Claude CLI

## Executable

Run this Bash block once in the shell that will launch Claude. The NVM
fallback also exposes the Node executable needed by npm installations.

```bash
claude_bin=$(command -v claude || true)
if [[ -z "$claude_bin" && -x "$HOME/.local/bin/claude" ]]; then
  claude_bin="$HOME/.local/bin/claude"
fi
if [[ -z "$claude_bin" ]]; then
  claude_bin=$(printf '%s\n' "$HOME"/.nvm/versions/node/*/bin/claude | sort -V | tail -n 1)
  export PATH="$(dirname "$claude_bin"):$PATH"
fi
if [[ ! -x "$claude_bin" ]]; then
  printf 'Claude executable missing from documented locations\n' >&2
  exit 127
fi
```

## Run

Set `repo` to the absolute target directory. Keep the variables and execution
in the same shell. Replace the prompt body with the task and relevant files.

```bash
repo='/absolute/project'
run_dir=$(mktemp -d /tmp/cli-claude.XXXXXX)
cat > "$run_dir/prompt.md" <<'PROMPT'
Review the requested files. Return findings with file locations and evidence.
PROMPT
if (cd "$repo" && "$claude_bin" --print --permission-mode dontAsk \
  --tools 'Read,Glob,Grep' --allowedTools 'Read,Glob,Grep' \
  < "$run_dir/prompt.md" > "$run_dir/result.md" 2> "$run_dir/stderr.log"); then
  run_status=0
else
  run_status=$?
fi
printf 'exit=%s artifacts=%s\n' "$run_status" "$run_dir"
```

For an authorized edit, write an implementation prompt with its validation
command. Replace both tool lists with:

```text
Read,Glob,Grep,Write,Edit,Bash
```

In `--allowedTools` only, replace `Bash` with the actual authorized command
rule, for example `Bash(npm test *)`. `--tools` selects available built-ins;
`--allowedTools` pre-approves calls. `dontAsk` denies calls needing a prompt.
These tool lists govern built-ins; configured MCP tools have separate permissions.

The configured model applies. Add `--model '<model>'` when the task specifies
one. The process uses `repo` as its working directory; `--add-dir` adds access
without changing that directory.

| Option | Use |
|---|---|
| `--output-format json` | Save the result and session ID as JSON. |
| `--output-format stream-json --verbose` | Save events as JSON lines. |
| `--resume '<session-id>' --print` | Continue that session with the next prompt. |

The default output is text. Read stdout even on failure: authentication errors
can appear there. A prompt file supplies EOF; use `</dev/null` when passing
the prompt as an argument.

## Checks

| Check | Filler | Direct |
|---|---|---|
| Non-interactive execution | Run Claude in automation. | Use `--print` with finite stdin. |
| Tool permissions | Give Claude the tools it needs. | Select built-ins with `--tools` and authorize task commands with `--allowedTools`. |
| Result evidence | Claude says the task succeeded. | Read exit status, saved output and the edited files. |

## Flow

```lnl
START:
  DO    resolve the executable with the block above and prepare the task directory and prompt
  DO    select the review tool lists or the edit tool lists and authorized command rules
  DO    run the documented command and wait for that process to exit; if the execution tool returns a session handle, continue waiting on that handle
  ASK   did the run fail or report a blocked action?
  STOP  report the exit status, saved diagnostics and blocked action IF yes
REVIEW:
  CHECK the answer agrees with the saved output and requested validation
  CHECK edits are inspected with git diff and git status --short in the target directory, including new files
REPLY:
  SAY   the result, artifact directory and any unverified claim
  END
```
