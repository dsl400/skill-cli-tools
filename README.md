# cli-tools

An index of CLI know-how. `skills/cli-tools/SKILL.md` links to a file beside
it for each tool. The skill is user-invocable.

| Tool | Reference |
|---|---|
| Claude Code | [claude.md](skills/cli-tools/claude.md) |
| Codex | [codex.md](skills/cli-tools/codex.md) |

## Checks

- **Documented recipes.** Routine runs use the reference directly. Help and
  external documentation are for uncovered operations or compatibility errors.
- **Executable resolution.** SSH shells can omit user installations from PATH.
  The recipes cover the installation locations found locally and on
  `gogu.lan`, `racheta.lan` and `ryz2`.
- **Finite input.** A prompt file reaches EOF, allowing an unattended run to proceed.
- **Failure evidence.** Exit status and retained output expose startup failures
  even when the tool produces no final answer.
- **Task isolation.** A separate worktree prevents overlapping workers from
  editing the same checkout. The target path belongs to the task.
- **Tool permissions.** Claude's available built-ins and pre-approved calls
  are separate settings; a review list needs changes before an edit can run.
- **Result evidence.** The diff, new files and requested validation establish
  what the delegated run accomplished.

## Maintenance sources

Codex flags were checked against local `codex exec --help` and `codex --help`
for version 0.165.0. The Codex reference originated from the skill on `ryz2`.
Claude flags were checked against local version 2.1.310 and the official
[CLI reference](https://code.claude.com/docs/en/cli-reference) and
[programmatic execution guide](https://code.claude.com/docs/en/headless).
Review date: 2026-11-22. These are maintenance sources, not runtime steps.

## Install

```sh
gh release download --repo dsl400/skill-cli-tools --pattern cli-tools.tar.gz -O - | tar -xz -C ~/.claude/skills
```

Or as part of [dsl400/skills](https://github.com/dsl400/skills).
