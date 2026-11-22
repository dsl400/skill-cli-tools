# cli-tools

An index of CLI know-how. `skills/cli-tools/SKILL.md` links to a file beside
it for each tool.

| Tool | Reference |
|---|---|
| Codex | [codex.md](skills/cli-tools/codex.md) |

## Checks

The index checks whether a tool has a reference so it can report missing
coverage. Tool-specific instructions live in the linked file and load when
that tool is needed.

`codex.md` is copied from the `codex/SKILL.md` file on `ryz2`. Its version
observation and model preference belong to that source environment.

## Install

```sh
gh release download --repo dsl400/skill-cli-tools --pattern cli-tools.tar.gz -O - | tar -xz -C ~/.claude/skills
```

Or as part of [dsl400/skills](https://github.com/dsl400/skills).
