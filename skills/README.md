# Skills

Claude Code skills bundled with this guide.

| Skill | Purpose |
| --- | --- |
| [`codemap`](codemap/SKILL.md) | Generate a `docs/CODEMAP.md` with bidirectional links between mermaid diagrams and source code |

The skill is a filesystem-walking variant — for repos that use [`code-review-graph`](https://github.com/anthropics/code-review-graph), the same skill description triggers Claude to read the graph as the source of truth instead of doing its own walk.

## Install

Clone or download this repo, then symlink the skill directory into your Claude Code skills folder:

```bash
# User-scope (every project)
ln -s "$PWD/skills/codemap" ~/.claude/skills/codemap

# OR project-scope (just this repo)
mkdir -p .claude/skills
ln -s "$PWD/skills/codemap" .claude/skills/codemap
```

Alternatively, copy the directory rather than symlinking if you don't want updates from this repo.

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
