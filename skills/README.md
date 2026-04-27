# Skills

Claude Code skills bundled with this guide.

| Skill | Purpose |
| --- | --- |
| [`codemap`](codemap/SKILL.md) | Generate a `docs/CODEMAP.md` with bidirectional links between mermaid diagrams and source code |
| [`codemap-gr`](codemap-gr/SKILL.md) | Same output as `codemap`, but backed by the [`code-review-graph`](https://github.com/anthropics/code-review-graph) knowledge graph for accurate symbol locations and call edges |

Both skills produce identical output formats — pick `codemap-gr` when the repo has `code-review-graph` available, otherwise use `codemap`.

## Install

Clone or download this repo, then symlink the skill directories into your Claude Code skills folder:

```bash
# User-scope (every project)
ln -s "$PWD/skills/codemap" ~/.claude/skills/codemap
ln -s "$PWD/skills/codemap-gr" ~/.claude/skills/codemap-gr

# OR project-scope (just this repo)
mkdir -p .claude/skills
ln -s "$PWD/skills/codemap" .claude/skills/codemap
ln -s "$PWD/skills/codemap-gr" .claude/skills/codemap-gr
```

Alternatively, copy the directories rather than symlinking if you don't want updates from this repo.

## Verify

In Claude Code, run:

```text
/skill codemap
```

The skill content will load if the file is correctly placed.

## Usage

Once installed, ask Claude:

- *"Generate a code map for this repo"*
- *"Refresh the CODEMAP"*
- *"Add a code map to docs/"*

Claude will invoke the skill automatically when the task matches its description.
