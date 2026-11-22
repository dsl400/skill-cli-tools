# Codex CLI

## Executable

Run this Bash block once in the shell that will launch Codex. It resolves
known installation locations, including SSH shells with a minimal PATH.

```bash
codex_bin=$(command -v codex || true)
if [[ -z "$codex_bin" && -x "$HOME/.local/bin/codex" ]]; then
  codex_bin="$HOME/.local/bin/codex"
fi
if [[ -z "$codex_bin" ]]; then
  codex_bin=$(printf '%s\n' "$HOME"/.vscode/extensions/openai.chatgpt-*-linux-x64/bin/linux-x86_64/codex | sort -V | tail -n 1)
fi
if [[ ! -x "$codex_bin" ]]; then
  printf 'Codex executable missing from documented locations\n' >&2
  exit 127
fi
```

## Run

Set `repo` to the absolute target directory. Keep the variables and execution
in the same shell. Replace the prompt body with the task, files, constraints
and expected validation.

```bash
repo='/absolute/project'
run_dir=$(mktemp -d /tmp/cli-codex.XXXXXX)
cat > "$run_dir/prompt.md" <<'PROMPT'
Review the requested files. Return findings with file locations and evidence.
PROMPT
if "$codex_bin" -a never exec -s read-only -C "$repo" \
  -o "$run_dir/result.md" - < "$run_dir/prompt.md" \
  > "$run_dir/events.log" 2> "$run_dir/stderr.log"; then
  run_status=0
else
  run_status=$?
fi
printf 'exit=%s artifacts=%s\n' "$run_status" "$run_dir"
```

For an authorized edit, use `-s workspace-write` and an implementation prompt.
When another worker is editing the repository, create an isolated checkout
and use its absolute path as `repo`:

```bash
git -C "$repo" worktree add -b 'codex/<task>' '<absolute-worktree-path>' HEAD
```

The configured model and effort apply. Override only when specified by the
user or task: `-m '<model>' -c 'model_reasoning_effort="<effort>"'`, after
`exec`. A task outside Git also needs `--skip-git-repo-check`.

| Option | Use |
|---|---|
| `-` | Read the prompt from stdin; the file supplies EOF. |
| `-o <file>` | Save the final answer. |
| `--json` | Write JSONL events to stdout; retain them in `events.log`. |
| `-a never` before `exec` | Return approval failures without waiting for interaction. |

For an argument prompt, close stdin with `</dev/null`. Retain stdout and
stderr: the final-answer file can be absent when startup fails.

## Checks

| Check | Filler | Direct |
|---|---|---|
| Input completion | Start Codex in the background. | Supply the prompt file as stdin, or close stdin for an argument prompt. |
| Failure evidence | Discard command output. | Retain exit status, `events.log`, `stderr.log` and `result.md` when present. |
| Task isolation | Always use the source project's worktree path. | Create a task-specific worktree when workers would overlap. |

## Flow

```lnl
START:
  DO    resolve the executable with the block above and prepare the task directory and prompt
  DO    select read-only for review or workspace-write for authorized edits
  DO    run the documented command and wait for that process to exit; if the execution tool returns a session handle, continue waiting on that handle
  ASK   did the process fail or omit its final answer?
  STOP  report the exit status and saved diagnostics IF yes
REVIEW:
  CHECK the answer agrees with the saved output and requested validation
  CHECK edits are inspected with git diff and git status --short in the target directory, including new files
REPLY:
  SAY   the result, artifact directory and any unverified claim
  END
```
