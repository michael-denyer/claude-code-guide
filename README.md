# Claude Code — User Guide

[![Topic: Claude Code](https://img.shields.io/badge/topic-Claude%20Code-D97757?logo=anthropic&logoColor=white)](https://claude.com/claude-code)
[![Topic: Anthropic](https://img.shields.io/badge/topic-Anthropic-D97757?logo=anthropic&logoColor=white)](https://www.anthropic.com)
[![Topic: LLMs](https://img.shields.io/badge/topic-LLMs-6E56CF)](https://en.wikipedia.org/wiki/Large_language_model)
[![Topic: Agentic Coding](https://img.shields.io/badge/topic-Agentic%20Coding-2D7A4F)](https://www.anthropic.com/engineering)
[![Topic: AI Developer Tools](https://img.shields.io/badge/topic-AI%20Developer%20Tools-1F6FEB)](https://github.com/topics/ai-coding)
[![Topic: MCP](https://img.shields.io/badge/topic-MCP-8957E5)](https://modelcontextprotocol.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Opinionated guide to working effectively with Claude Code in VS Code. Started life as a fresh-setup checklist and grew into a user guide covering setup, file layout, operating principles, model strategy, and prompt practice. Bias: minimal surface area, high signal, fail-loud over fail-silent, mechanism over memory.

**How to read this**:

- **New to Claude Code?** Read [Onboarding ladder](#onboarding-ladder) first, then come back for depth as you hit it.
- **Setting up a fresh machine or repo?** [TL;DR](#tldr--what-to-install-on-day-one), then [Install instructions](#install-instructions).
- **Want to work better with Claude?** [Operating principles](#principle-doc-as-rule-needs-doc-as-gate), [Model selection strategy](#model-selection-strategy), and [Prompt tips](#prompt-tips--getting-more-from-claude).
- **Reference**: [CLAUDE.md split](#claudemd-split--what-goes-where), [Settings split](#settings-split), [MCP server config](#mcp-server-config).

This document is itself an example of the principles inside it: imperatives at the top, depth on demand, every rule paired (where possible) with the gate that enforces it.

---

## TL;DR — What to install on day one

| Layer | Install | Skip |
|-------|---------|------|
| Editor integration | LSP servers (incl. `pyrefly` plugin for Python), Claude Code VS Code extension | — |
| Skills | `superpowers` (selective), `pr-review-toolkit` | — |
| Planning | `GSD` for large projects requiring detailed planning and codebase mapping | GSD for small/exploratory work |
| Repo hygiene | pre-commit hooks (`ruff`, `markdownlint-cli2`, `maid`, `mmdc`) | — |
| Tracking | `bd` (beads) | Markdown TODOs scattered across files |
| Python | `uv`, `pytest-randomly`, `pytest-xdist`, `pytest-timeout` | `pip`, `setup.py`, `requirements.txt` |
| Graphs | `code-review-graph` only on >50k LOC repos | Don't pre-install |
| CI | SHA-pinned actions, Dependabot, `osv-scanner` | Tag-pinned actions (`@v6`) |

---

## Onboarding ladder

A staged path from "fresh install" to "productive in a real project". Don't try to internalise everything below in one sitting — work through the ladder, and revisit each rung when you hit a real example of why it matters.

### Day 1 — get something working

Goal: a working Claude Code that can read, edit, and run code in one repo.

- [ ] Install Claude Code (CLI + VS Code extension) and sign in
- [ ] Install a language LSP for your primary language (for Python: the **`pyrefly`** Claude plugin)
- [ ] Pick **one** real repo and clone it locally
- [ ] Have Claude do something concrete: explain a function, fix a tiny bug, add a one-line test
- [ ] Notice when Claude asks for permission — start saying "yes" deliberately, not reflexively
- [ ] Read your first auto-generated CLAUDE.md (if the repo has one)

**Skip on day 1**: skills, plugins, MCP servers, GSD, `bd`, anything that isn't the core read/edit/run loop. Earn complexity later.

### Day 2 to 3 — basic file layout

Goal: understand where rules and settings live, so you can shape Claude's behaviour.

- [ ] Create `~/.claude/CLAUDE.md` with your communication preferences (rigorous, terse, no AI attribution in commits)
- [ ] Create `<repo>/CLAUDE.md` with one paragraph of project purpose + any one gotcha you've already hit
- [ ] Create `~/.claude/settings.json` with a small `allow` list (`git status`, `git diff`, `ls`, `rg`) — see the [Settings split](#settings-split) section
- [ ] Add a `deny` rule for one obviously dangerous command (`rm -rf:*`, `git push --force:*`)
- [ ] Run a task and watch how the allow list reduces permission prompts — that's the feedback loop

### Week 1 — repo hygiene and tracking

Goal: the repo enforces its own rules so you don't have to remember them.

- [ ] Install `prek` and add a minimal `.pre-commit-config.yaml` (`ruff`, `markdownlint-cli2`)
- [ ] Add `mmdc` to the pre-commit if you use mermaid diagrams (it catches what `maid` alone won't)
- [ ] Install `bd` (beads) and `bd init` in the repo — start tracking work as issues, not as scattered TODOs
- [ ] Add `~/.claude/CLAUDE.md` rules for the conventions you keep correcting Claude on (e.g. "always commit `uv.lock` after a `pyproject.toml` version bump")
- [ ] Run `/fewer-permission-prompts` once to mine your transcript for safe-to-allow commands

### Week 2 — skills and process discipline

Goal: turn repeated work into reusable patterns.

- [ ] Install `superpowers` and use **one** skill in anger: `brainstorming` before a feature, or `systematic-debugging` on the next bug
- [ ] Install `pr-review-toolkit` and run `silent-failure-hunter` before your next PR
- [ ] When you find yourself prompting Claude the same way three times, promote that to a saved prompt or a skill (see [Principle: prompts → skills](#principle-prompts--skills-promote-repeated-work))
- [ ] Read the [Operating principles](#principle-doc-as-rule-needs-doc-as-gate) — even just the headlines

### Week 2 onwards — model strategy and cost awareness

Goal: stop paying Opus rates for Sonnet-shape work.

- [ ] Read [Model selection strategy](#model-selection-strategy) — internalise plan-in-Opus / execute-in-Sonnet / dispatch-in-Haiku
- [ ] Use `/clear` between unrelated tasks to bound context and cost
- [ ] Try one parallel-subagent pattern: spawn a Sonnet research subagent while you keep the main thread on Opus planning
- [ ] Check session cost (e.g. `/gsd-session-report` if you have GSD; or your provider's billing dashboard) — most users overspend by 3–5× before they look

### Month 1 — MCP servers (only when needed)

Goal: connect external systems Claude can read from, with read-only as the default.

- [ ] Add a project `.mcp.json` only when a real task hits an external system
- [ ] Default every database / warehouse server to **read-only** at three layers: server flag (`--readonly`), database role (SELECT-only `GRANT`s), settings deny rules
- [ ] See the worked Databricks and `dbhub` PostgreSQL recipes in [MCP server config](#mcp-server-config)

### Month 1+ — heavier tools, only if you feel the gap

Add these when you have a concrete pain point, not preemptively:

- **GSD** — only on a real multi-week project that needs phase-gated planning and codebase mapping
- **`code-review-graph`** — only when grep stops scaling (>50k LOC, lots of cross-module coupling)
- **CI hardening** (Dependabot, `osv-scanner`, SHA-pinned actions) — once you publish anything externally
- **`claude-md-management`** plugin — once any of your CLAUDE.md files cross ~150 lines or feel stale

### Signals to revisit the ladder

- You're correcting Claude on the same convention for the third time → write a CLAUDE.md rule + (where possible) a gate
- You're prompting Claude the same way for the third time → promote to a skill or command
- You're running the same review checklist every PR → add a `pr-review-toolkit` invocation or a hook
- Your CLAUDE.md is over 300 lines → split into `docs/<TOPIC>.md` files with a tight index (see [AI signposting](#principle-ai-signposting-in-documentation))
- Your monthly Claude bill surprises you → re-read [Model selection strategy](#model-selection-strategy)

The ladder is not linear. Skip rungs that don't fit your work; come back to them when a real task surfaces the need. The point is **earn complexity, don't pre-pay for it**.

---

## Tooling tiers

### Tier 1 — install first (table stakes)

**LSP servers.** Single biggest quality multiplier. Claude uses `findReferences` / `goToDefinition` / `workspaceSymbol` instead of grep-guessing. Configure for every language in your stack (Python, TS, Go, etc.).

For Python specifically, install **[`pyrefly-lsp-cc-plugin`](https://github.com/michael-denyer/pyrefly-lsp-cc-plugin)** — wires Meta's `pyrefly` type checker into Claude as an LSP. Faster and stricter than `pyright`/`mypy` for the kinds of checks Claude actually needs (find references, go-to-def, type-at-cursor) and catches real type errors before they hit CI. See install instructions below.

**Pre-commit hooks (run via `prek`).** Catch trash before CI. Use **[`prek`](https://github.com/j178/prek)** as the runner — a Rust rewrite of `pre-commit` that's drop-in compatible with `.pre-commit-config.yaml`, dramatically faster (10–100× on cold runs), single-binary install, and handles language environments without polluting the repo. Keep the standard `.pre-commit-config.yaml` filename and schema; just invoke `prek` instead of `pre-commit`.

```yaml
# .pre-commit-config.yaml — recommended baseline (consumed by prek)
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.15.0  # pin exact version
    hooks:
      - id: ruff
      - id: ruff-format
  - repo: https://github.com/DavidAnson/markdownlint-cli2
    rev: v0.18.1
    hooks:
      - id: markdownlint-cli2
        exclude: ^(\.venv|\.planning)/
  - repo: local
    hooks:
      - id: maid
        name: maid (mermaid syntax)
        entry: npx -p @probelabs/maid maid
        language: node
        files: \.(md|mmd)$
        exclude: ^(\.venv|\.planning)/
      - id: mmdc
        name: mmdc (mermaid render parity)
        entry: npx -p @mermaid-js/mermaid-cli mmdc -i
        language: node
        files: \.mmd$
```

`maid` alone is insufficient — its parser accepts constructs the real mermaid renderer rejects (e.g., `;` in sequence-diagram loop labels). `mmdc` catches them.

**`superpowers` skills (selective).** The skills you'll actually use:

- `brainstorming` — before any feature work
- `systematic-debugging` — before any bug fix
- `test-driven-development` — for non-trivial implementation
- `verification-before-completion` — before claiming work is done
- `writing-plans` / `executing-plans` — for multi-step work

Skip the heavyweight orchestration if it doesn't fit your workflow.

### Tier 2 — install when you feel the pain

**`pr-review-toolkit`.** Useful right before opening a PR. Best agents:

- `silent-failure-hunter` — catches `try/except: pass`, swallowed errors, fallbacks that mask bugs
- `comment-analyzer` — flags stale or wrong comments (real source of tech debt)
- `code-reviewer` — checks against project guidelines
- `type-design-analyzer` — for new types in typed languages

Don't run all five on every diff — that's noise. Run them selectively.

**`code-review-graph`.** Knowledge graph over code with semantic search via embeddings. Worth it when:

- Codebase >50k LOC
- Lots of cross-module coupling
- You're doing architectural refactors

Skip on fresh/small repos. Add later when grep stops scaling.

**`codemap` / `codemap-gr` (bundled in this repo).** Generates a `docs/CODEMAP.md` with mermaid architecture diagrams and **bidirectional links** between diagram nodes and source code — clicking a node jumps to the file/function, and source files link back to where they appear on the map. Numbered architectural layers (`[1]` entry points → `[5]` background) and a stable colour palette make code maps comparable across repos. `codemap-gr` is the variant backed by the `code-review-graph` knowledge graph (use it when you have CRG installed; the output is identical). Worth it when:

- A new contributor needs a 5-minute orientation to a non-trivial repo
- You're refactoring and want a single visual to sanity-check what touches what
- A phase plan or PR description benefits from a visual summary

See [`skills/`](skills/) in this repo for the skill files and install instructions.

### Tier 3 — install on the right projects

**GSD (Getting Shit Done).** Massively useful on the right projects: large, multi-week efforts that require detailed planning, structured spec-to-execute discipline, and deep codebase mapping. The phase-gated workflow (spec → discuss → plan → execute → verify → secure → ship), milestone/phase/plan hierarchy under `.planning/`, parallel mapper agents, and persistent thread/context tooling are exactly what you want when:

- The project will run for weeks or months
- Requirements are unclear and need Socratic refinement
- You're onboarding to or refactoring a large codebase and need detailed mapping (`/gsd-map-codebase`, `/gsd-scan`, `/gsd-graphify`)
- You want forced discipline so nothing slips between specification and execution
- Multiple workstreams or parallel phases need coordination

**Bad fit for**: bug fixes, small features, exploratory throwaway work, scripts, or repos where the planning shape is already handled by Linear/Jira and another system. On those, the `.planning/` artifact tree and ~70 `gsd-*` commands are pure overhead.

**Recommendation**: install per-project on large efforts where you'd benefit from detailed planning and codebase mapping. Don't install globally by default.

---

## CLAUDE.md split — what goes where

Three locations, three different lifetimes and audiences.

```text
~/.claude/CLAUDE.md           # Global — your preferences, every project
<repo>/CLAUDE.md              # Project — committed, every contributor
<repo>/CLAUDE.local.md        # Project local — gitignored, your machine
```

### `~/.claude/CLAUDE.md` (global)

**Audience**: you, across every project. Committed to nothing.

**Belongs here**:

- Communication style (rigorous vs sycophantic, no AI attribution in commits)
- Universal coding philosophy (Pragmatic Programmer principles, language-agnostic)
- Default tooling preferences (`uv` over `pip`, `ruff` over `black`+`flake8`, `loguru` over `logging`)
- Test discipline defaults (randomization, parallel, timeouts, observable-output asserts)
- CI/CD hardening rules that apply everywhere (SHA-pin actions, no `--quiet`, `--index-url` over `--extra-index-url`)
- Diagram conventions (mermaid only, `<br/>` not `\n`)
- Documentation style (Google docstrings, README link discipline, strip generator tags)
- Git commit rules that apply to all your repos
- LSP-over-grep preference
- Cross-project tools (`bd`, `code-review-graph` install recipe pointer)

**Does NOT belong here**:

- Project-specific architecture
- Repo-specific commands (`make test`, `pnpm build`)
- Domain knowledge for one codebase
- Anything contributors should also follow (that goes in project CLAUDE.md)

### `<repo>/CLAUDE.md` (project, committed)

**Audience**: every contributor (human and AI). Committed to git.

**Belongs here**:

- One-paragraph project purpose
- Architecture overview + mermaid diagram
- How to run things: setup, test, lint, build commands
- Where the dev environment lives (`uv sync`, devcontainer, etc.)
- Domain gotchas — non-obvious behavior that surprised someone once
- Known bugs and their workarounds (with link to the issue)
- Testing patterns specific to this repo (fixtures, factories, integration vs unit conventions)
- Module/package layout map (what lives where)
- Links to `docs/` (ARCHITECTURE.md, USER_GUIDE.md)
- External system references (Linear project name, Grafana dashboard URL)

**Does NOT belong here**:

- Your personal preferences (those are global)
- Anything machine-specific (paths, credentials)
- Stale TODO lists (use `bd`)
- Generated provenance tags

**Keep tight**. If it gets >300 lines, split into `docs/` and link from CLAUDE.md.

### `<repo>/CLAUDE.local.md` (project, gitignored)

**Audience**: just you on this machine. `.gitignore`d.

**Belongs here**:

- Local paths (`/Users/<you>/data/...`)
- Personal scratchpad notes about this repo
- Temporary "while I'm working on X, remember Y" context
- Local-only env nudges ("on this machine, use port 8081 not 8080 because $reason")
- Personal experiments not yet ready for the project CLAUDE.md
- Memory pointers for ongoing work (link to `.planning/` or beads issue IDs)

**Add to `.gitignore`**:

```gitignore
CLAUDE.local.md
.claude/settings.local.json
```

### Maintaining CLAUDE.md files — the `claude-md-management` plugin

CLAUDE.md files rot fast: they accumulate stale rules, contradict the current code, and grow past the point where Claude actually reads them. Two skills in the `claude-md-management` plugin keep them honest:

| Skill | When to use |
| --- | --- |
| `claude-md-management:revise-claude-md` | **End of a session** that produced new conventions, surprises, or guardrails. Captures session learnings (what worked, what didn't, what to avoid next time) and folds them into the relevant CLAUDE.md (global, project, or local). The "I just learned X — remember it" path. |
| `claude-md-management:claude-md-improver` | **Periodic audit / new contributor onboarding**. Scans every `CLAUDE.md` in the repo, evaluates each against template best practices, produces a quality report (length, link discipline, stale rules, missing sections, redundancy with `docs/`), then applies targeted fixes. The "spring cleaning" path. |

**Recommended cadence**:

- Run `revise-claude-md` ad-hoc whenever a session surfaces a non-obvious gotcha you don't want to relearn
- Run `claude-md-improver` quarterly, or whenever a CLAUDE.md crosses ~300 lines, or before onboarding a new contributor

**Install**:

```bash
# In Claude Code:
/plugin add claude-md-management
# Then invoke either skill via the Skill tool or `/revise-claude-md` / `/claude-md-improver`
```

Both skills respect the global/project/local split — they update the right file based on whether the rule is universal (global), repo-wide (project), or machine-specific (local).

---

## Settings split

Three tiers, mirroring the CLAUDE.md split.

```text
~/.claude/settings.json                # Global — your defaults across every repo
<repo>/.claude/settings.json           # Project — committed, every contributor
<repo>/.claude/settings.local.json     # Project local — gitignored, your machine
```

### `~/.claude/settings.json` (global)

Universal allowlist for read-only operations and your trusted tools.

```jsonc
{
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(git show:*)",
      "Bash(ls:*)",
      "Bash(cat:*)",
      "Bash(rg:*)",
      "Bash(uv run:*)",
      "Bash(uv sync)",
      "Bash(uv lock)",
      "Bash(bd ready)",
      "Bash(bd show:*)",
      "Bash(bd stats)",
      "Bash(gh pr view:*)",
      "Bash(gh pr list:*)",
      "Bash(gh issue view:*)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(git push --force:*)",
      "Bash(git reset --hard:*)"
    ]
  },
  "env": {
    "PYTHONDONTWRITEBYTECODE": "1"
  }
}
```

Use `/fewer-permission-prompts` periodically to scan transcripts and grow this list.

### `<repo>/.claude/settings.json` (project, committed)

Project-specific commands every contributor should be allowed to run without prompts.

```jsonc
{
  "permissions": {
    "allow": [
      "Bash(make test)",
      "Bash(make lint)",
      "Bash(pytest:*)",
      "Bash(npm run build)",
      "Bash(npm run test)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/post-edit-lint.sh" }
        ]
      }
    ]
  }
}
```

### `<repo>/.claude/settings.local.json` (project, gitignored)

Your personal overrides for this repo.

```jsonc
{
  "permissions": {
    "allow": [
      "Bash(docker compose up:*)",
      "Bash(./scripts/local-dev.sh:*)"
    ]
  },
  "env": {
    "DEBUG": "1",
    "LOCAL_DB_URL": "postgres://localhost:5432/dev"
  }
}
```

---

## MCP server config

```text
~/.claude.json                         # Global MCP servers (user scope)
<repo>/.mcp.json                       # Project MCP servers (committed)
```

### Global (`~/.claude.json` user-scope MCP entries)

Servers you want everywhere:

- `code-review-graph` — code intelligence (if installed)

### Project (`<repo>/.mcp.json`, committed)

Servers tied to this project's external systems. **Default to read-only connections** for any data store that can mutate state — never give an LLM `write` or `execute` access to a SQL warehouse or production database without an explicit, audited reason.

```jsonc
{
  "mcpServers": {
    "databricks-sql": {
      "command": "uvx",
      "args": ["databricks-mcp-server"],
      "env": {
        "DATABRICKS_HOST": "${DATABRICKS_HOST}",
        "DATABRICKS_TOKEN": "${DATABRICKS_READONLY_TOKEN}",
        "DATABRICKS_WAREHOUSE_ID": "${DATABRICKS_READONLY_WAREHOUSE_ID}",
        "DATABRICKS_READ_ONLY": "true"
      }
    },
    "postgres-readonly": {
      "command": "npx",
      "args": [
        "-y",
        "@bytebase/dbhub",
        "--transport", "stdio",
        "--dsn", "${POSTGRES_READONLY_URL}",
        "--readonly"
      ]
    }
  }
}
```

**Read-only setup — Databricks**:

- Use a **dedicated SQL warehouse** with a service principal that has `USE CATALOG` / `SELECT` only — no `MODIFY`, no `CREATE`, no `EXECUTE` on Unity Catalog
- Token used (`DATABRICKS_READONLY_TOKEN`) is generated for that service principal, never your personal PAT
- The `databricks-sql` MCP server exposes both `execute_sql` and `execute_sql_read_only` — bind only the read-only variant via the warehouse permissions even though both tools exist (defence in depth: server allows both, ACL allows neither write)
- Optional belt-and-braces: set `DATABRICKS_READ_ONLY=true` if the server build supports it, so it refuses non-`SELECT` statements before hitting Unity Catalog

**Read-only setup — PostgreSQL**:

- Build the connection string from a **read-only role**, not the app role:

  ```bash
  POSTGRES_READONLY_URL=postgresql://readonly_user:${PASSWORD}@host:5432/db?sslmode=require
  ```

- Create the role with `SELECT`-only grants:

  ```sql
  CREATE ROLE readonly_user LOGIN PASSWORD '...';
  GRANT CONNECT ON DATABASE mydb TO readonly_user;
  GRANT USAGE ON SCHEMA public TO readonly_user;
  GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;
  ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO readonly_user;
  REVOKE INSERT, UPDATE, DELETE, TRUNCATE ON ALL TABLES IN SCHEMA public FROM readonly_user;
  ```

- Use **[`@bytebase/dbhub`](https://github.com/bytebase/dbhub)** as the MCP server. Pass `--readonly` so the server itself rejects any non-`SELECT` statement before it hits the database; pair this with the read-only role above for two layers of defence (server-level guard + database-level `GRANT`s)
- `dbhub` works across PostgreSQL, MySQL, MariaDB, SQL Server, and SQLite, so the same server config + guardrail pattern transfers to other databases without learning a new tool
- Connection string is passed via `--dsn` (env var substitution still works) — keep `sslmode=require` and prefer a replica DSN over a primary that accepts writes
- Exposed tool surface in dbhub is intentionally narrow (schema introspection + parametrised queries) — combined with `--readonly` mode there is no DDL/DML path

### Settings guardrails — refuse execute without read-only confirmation

The MCP config is the first line of defence; settings policy is the second. Add `deny` rules to **both global and project** `settings.json` so the tools that *can* mutate state require explicit case-by-case approval, even if a misconfigured server exposes them.

```jsonc
// ~/.claude/settings.json (global)
{
  "permissions": {
    "deny": [
      // Databricks: deny the unrestricted execute, allow only the read-only variant
      "mcp__databricks-sql__execute_sql",
      // dbhub: deny any postgres MCP server name that does NOT have the
      // -readonly suffix — forces every contributor to register the server
      // under postgres-readonly with --readonly, or hit a permission prompt
      "mcp__postgres__*",
      "mcp__postgres-write__*"
    ],
    "allow": [
      "mcp__databricks-sql__execute_sql_read_only",
      "mcp__databricks-sql__poll_sql_result",
      "mcp__postgres-readonly__*"
    ]
  }
}
```

The pattern: **deny the unrestricted variant globally**, **allow the read-only-named variant**, and let permission prompts surface any attempt to execute mutating SQL. With `dbhub`, the `--readonly` flag enforces this server-side; the settings deny rule is the second layer that catches the case where someone registers the server without `--readonly` or under a non-conforming name. This works even if the underlying MCP server is misconfigured to expose write paths — the harness blocks the call before the server sees it.

For project-level lock-down, mirror the deny rules into `<repo>/.claude/settings.json` so contributors inherit the guardrail without per-machine setup.

**Rules**:

- Use env var substitution (`${VAR}`) — never inline secrets
- Document required env vars in project CLAUDE.md
- Add `.env` and `.env.local` to `.gitignore`
- Prefer read-only MCP scopes where the server supports them — and back them with database-level `GRANT`s, not just trust in the tool name
- Name servers with `-readonly` suffix so the intent is visible in every prompt and audit log
- If you genuinely need write access (rare), spin up a *second* MCP entry (`postgres-write`) gated by a project-local settings override, and require a pull request to enable it — never let the read-only entry get write privileges

---

## What to add that the question didn't mention

- **`bd` (beads) — issue tracker designed for AI-coding agents.** Local-first, git-synced task database that survives session compaction. Replaces scattered markdown TODOs with persistent issues that have IDs, dependencies, priorities, and types (task/bug/feature/epic/chore). Critical commands: `bd ready` (unblocked work), `bd create`, `bd show <id>`, `bd dep` (dependencies), `bd sync` (push/pull via git). Use it for any work that spans more than one session.
- **`uv`** as the Python package manager — speed and lockfile reproducibility
- **`pytest-randomly` + `pytest-xdist -n 3` + `pytest-timeout=30`** as default dev deps in every Python project
- **GitHub Actions pinned to commit SHAs** + Dependabot on the `github-actions` ecosystem
- **`osv-scanner`** as a CI step on lockfiles for CVE detection
- **Sigstore build provenance** (`actions/attest-build-provenance`) for any package you publish to PyPI
- **YAML form issue templates** (not markdown) for public repos — harder for spam bots

---

## Principle: doc-as-rule needs doc-as-gate

**Generic form**: any rule stated only in prose, without a mechanical check, will decay. Documentation describes intent; **enforcement defines reality**. The two must travel together — written together, changed together, deleted together.

> "If it isn't automated, it isn't done." — Jez Humble & David Farley, *Continuous Delivery* (2010)

**Every prose rule needs a CI or hook counterpart, or it will drift.**

A rule that lives only in `CLAUDE.md`, `TESTING.md`, `CONTRIBUTING.md`, or a README is a **doc-as-rule**: a stated norm with no enforcement. It will be followed perfectly the day it's written and erode from there. Concrete failure mode: a `TESTING.md` that says "always tier-mark new tests" with no check — within weeks new tests land unmarked, and the rule is fiction. The fix is a `conftest.py` hook (or pre-commit, or CI step) that **fails loudly** when an unmarked test shows up. That's a **doc-as-gate**: the rule is also code that runs.

The principle generalises beyond tests:

- A **policy** without a gate is an aspiration
- A **convention** without a linter is folklore
- A **threshold** without a check is a target nobody hits
- A **boundary** without an enforcement mechanism is a suggestion
- A **deprecation notice** without a build-time warning is invisible

When you write a rule into a doc, in the same PR add the gate that enforces it:

| Prose rule (doc-as-rule) | Enforcement (doc-as-gate) |
| --- | --- |
| "All tests must be tier-marked" | `conftest.py` hook that errors on unmarked tests |
| "No `--quiet` flags in CI" | `ruff` custom rule or grep-based pre-commit hook |
| "Pin GitHub Actions to SHAs" | `pinact` or grep hook in pre-commit |
| "No mocks in integration tests" | pytest marker + import-linter rule |
| "All public functions have docstrings" | `ruff` `D1` rules enabled |
| "Use `<br/>` not `\n` in mermaid labels" | `mmdc` render parity hook (catches it via render failure) |
| "Strip generator tags from committed docs" | grep hook for `generated-by:` / `generated by claude` |
| "Commit `uv.lock` after `pyproject.toml` version bump" | pre-commit hook comparing lock vs `pyproject` mtimes |
| "No AI attribution in commit messages" | `commit-msg` hook rejecting `Co-Authored-By: Claude` etc. |

**Boundary catalogue addition.** When cataloguing system boundaries, add a column for *enforcement mechanism* alongside *rule statement*. A boundary without an enforcement mechanism is not a boundary — it's an aspiration. Items missing enforcement go on a remediation list with a deadline.

**Tolerance updates.** When you tighten or loosen a rule (raise a coverage threshold, change a strictness level, add a new lint), update **both** the prose statement and the gate in the same commit. If they live apart, they will diverge. If they must live apart (e.g., a docs-only repo describes a CI in another repo), add a cross-reference test or a scheduled drift-detection check.

**Rule of thumb**: if you can't grep for a violation or fail a build on it, the rule isn't real — it's documentation theatre.

---

## Principle: AI signposting in documentation

Claude reads docs the same way humans do — in the order they appear, prioritising what's near the top, skimming what's nested deep. Treat your CLAUDE.md, README, and module docs as **signposted for an AI reader**, not just a human one.

**Concrete signposting rules**:

- **Lead with the unmissable.** Domain gotchas, irreversible operations, and "do not do X" rules go in the first 50 lines of CLAUDE.md, not buried in section 7. Claude truncates / skims; the top of the file gets the most attention.
- **State rules as imperatives.** "Always commit `uv.lock` after a version bump" reads more reliably than "It can be helpful to commit lockfiles."
- **Mark danger zones explicitly.** Use a `## Gotchas` or `## Do not` section. Claude weights headings highly — a heading called `## Known bugs` will be checked; the same content under `## Notes` won't.
- **Link, don't restate.** If something is in `docs/ARCHITECTURE.md`, link it from CLAUDE.md and trust Claude to follow the link when relevant. Restating creates two sources that drift.
- **Add reasoning to non-obvious rules.** "Use `<br/>` not `\n` in mermaid labels — `\n` renders literally" gives Claude enough context to apply the rule to edge cases. Bare rules without rationale get over- or under-applied.
- **Use callout patterns Claude recognises.** `**IMPORTANT:**`, `**Do not:**`, `**Why:**` are signal-rich for an LLM. Plain prose is not.
- **Date-stamp project memory.** "As of 2026-04, we use `prek` not `pre-commit`" beats "We use `prek`" — the date tells Claude when the claim was true and whether to verify.

**Anti-pattern**: a 600-line CLAUDE.md with everything in flat prose. Claude will read it, but the high-priority signals get lost in noise. Split into a tight top-of-file rules block (the things Claude must never miss) and **link to depth in topic-specific docs**. The CLAUDE.md becomes an index; the depth lives where it can be read on demand.

**Recommended split** for a typical repo:

| Topic | Lives in | CLAUDE.md role |
| --- | --- | --- |
| Hard rules / gotchas / "do not do X" | `CLAUDE.md` itself (top of file) | Stated in full |
| Architecture overview + module map | `docs/ARCHITECTURE.md` | One-line summary + link |
| Setup, install, dev environment | `docs/SETUP.md` or `README.md` | One-line summary + link |
| CI pipelines, build matrix, release process | `docs/CI.md` | One-line summary + link |
| Databricks notebook conventions, cluster IDs, common SQL | `docs/DATABRICKS.md` | One-line summary + link |
| Database schema, query patterns, migration playbook | `docs/DATABASE.md` | One-line summary + link |
| Testing strategy, fixtures, tier-marking rules | `docs/TESTING.md` | One-line rule + link |
| Deployment runbook, on-call, rollback procedures | `docs/RUNBOOK.md` | One-line summary + link |
| Domain glossary, business rules, external integrations | `docs/DOMAIN.md` | One-line summary + link |
| API contracts, schema versioning, breaking-change policy | `docs/API.md` | One-line summary + link |

**Why this works for an AI reader**:

- The top-of-file rules block stays under ~150 lines so Claude weights it heavily on every turn
- Each link is a load-on-demand pointer — Claude reads `docs/DATABRICKS.md` only when the task touches Databricks, instead of carrying it as ambient context for unrelated work
- A linked doc that drifts is easier to spot in review than a buried paragraph in CLAUDE.md
- Contributors editing one topic (e.g. CI) touch one file — fewer merge conflicts, clearer git blame

**The CLAUDE.md index pattern**:

```markdown
## Topic-specific guides

For depth, see:

- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) — module layout and data flow
- [docs/CI.md](docs/CI.md) — pipeline, required checks, release tags
- [docs/DATABRICKS.md](docs/DATABRICKS.md) — workspace IDs, cluster policies, SQL warehouse names
- [docs/TESTING.md](docs/TESTING.md) — fixtures, tier marks, integration vs unit
- [docs/RUNBOOK.md](docs/RUNBOOK.md) — incident playbooks, rollback steps
```

If a section in CLAUDE.md grows past ~30 lines, that's the signal to extract it into `docs/<TOPIC>.md` and replace it with a one-line pointer.

---

## Principle: prompts → skills (promote repeated work)

If you've prompted Claude the same way twice, the third time it should be a skill, command, or plugin — not a re-typed prompt.

**Promotion ladder** (cheapest to most durable):

| Tier | When | Lives where |
| --- | --- | --- |
| Ad-hoc prompt | One-off task | Chat |
| Saved prompt / scratchpad | You'll do this 2–3 more times | `~/.claude/prompts/` or `<repo>/.claude/prompts/` |
| Skill | Repeated workflow with judgment calls | `~/.claude/skills/` (global) or `<repo>/.claude/skills/` (project) |
| Slash command | Repeated workflow with deterministic shape | `~/.claude/commands/` or plugin command |
| Plugin | Bundle of related skills/commands/hooks for distribution | `~/.claude/plugins/` |

**Promotion criteria**:

- **3+ uses** of the same prompt → write a skill or command
- **Cross-project** applicability → goes global (`~/.claude/`)
- **Project-specific** workflow → goes in repo (`<repo>/.claude/`)
- **Multiple related workflows** → bundle as a plugin

**What goes in a skill vs a command**:

- **Skill** = soft reasoning, judgment, multi-step decisions, "here's how to think about X"
- **Command** = deterministic execution, "run these steps", argument-driven

When a session keeps invoking the same investigation pattern (e.g., "look at git history for this file, find the original PR, summarise the design decisions"), that's a skill candidate. Use the `plugin-dev:skill-development` and `plugin-dev:command-development` skills to scaffold properly.

**Don't over-promote**. A prompt used twice is not yet a skill — premature promotion creates skill sprawl that dilutes attention. Wait for the third repeat.

---

## Principle: correct errant behaviour with mechanism, not memory

When Claude does the wrong thing — drifts off-spec, misses a convention, takes a shortcut you didn't want — the fix is **a mechanism that prevents recurrence**, not a memory you hope it retains.

**Correction toolkit, in order of preference**:

| Mechanism | When to use | Permanence |
| --- | --- | --- |
| **Hook** (PreToolUse, PostToolUse, Stop) | Block / validate specific tool calls or outputs at runtime | Hardest — fires every time, no opt-out |
| **CI gate** | Catch the issue at PR time even if the hook is bypassed | Hard — blocks merge |
| **Policy in `settings.json`** (`deny` rules) | Prevent dangerous bash commands or tool patterns globally | Hard — system-enforced |
| **Skill** | Encode the right *thinking* for a class of task | Medium — invoked when relevant |
| **Slash command** | Encode the right *steps* for a deterministic task | Medium — explicitly invoked |
| **Plugin update** | Fix at source if the errant behaviour came from a plugin's skill/command | Hard — fixes for everyone |
| **CLAUDE.md update** | Document the rule for human + AI readers | Soft — drift-prone unless paired with a gate |
| **In-session correction** | One-off, non-recurring | Softest — evaporates next session |

**The escalation rule**: if Claude does X wrong **once**, correct in-session and (if non-trivial) write a memory or CLAUDE.md note. If Claude does X wrong **twice across sessions**, escalate to a hook, gate, or skill — memory alone clearly isn't holding.

**Updating existing tooling beats adding new tooling.** If a behaviour leaks past a skill you already have, edit that skill — don't write a second one that overlaps. If a hook isn't firing on a case you care about, broaden the matcher rather than adding a parallel hook. Skill / command / plugin sprawl is a real cost; consolidate before you add.

**Worked examples**:

| Errant behaviour | Wrong fix | Right fix |
| --- | --- | --- |
| Claude commits with `Co-Authored-By: Claude` | Tell it not to in CLAUDE.md | Add `commit-msg` hook rejecting AI attribution + update global CLAUDE.md |
| Claude runs `rm -rf` without confirming | Memory: "always confirm destructive ops" | `deny` rule in `~/.claude/settings.json` for `Bash(rm -rf:*)` |
| Claude skips writing tests for new functions | Tell it to do TDD next time | Update / re-invoke the `superpowers:test-driven-development` skill; add coverage gate in CI |
| Claude uses `\n` in mermaid labels | Note in CLAUDE.md | `mmdc` render-parity pre-commit hook (catches via render failure) |
| Claude forgets to commit `uv.lock` after version bump | Verbal correction | `prek` hook comparing `uv.lock` mtime vs `pyproject.toml` mtime |
| Claude proposes a refactor mid-bugfix | "Stay focused next time" | Add to global CLAUDE.md (it's already a stated rule) — escalate to a skill if it recurs |
| Claude writes shallow docstrings | Reminder in chat | `ruff` `D1` rules in `pyproject.toml` |

**This is the same principle as doc-as-rule needs doc-as-gate**, applied to AI behaviour: a stated correction without a mechanical anchor is just hope.

---

## Principle: testing — fakes over mocks, outcomes over internals

**LLMs left to their own devices over-rely on mocks and preferentially test internals.** This is not a Claude-specific failure — every LLM has been trained on a public corpus dominated by mock-heavy unit-test examples (Stack Overflow, blog posts, tutorial code) where `mock.patch(...)` and `assert_called_once_with(...)` are the dominant patterns. The result: without explicit guidance, an LLM will reach for `mock.patch` against an internal function as the *first* tool, assert call counts and argument shapes as the *primary* check, and treat "the test passes" as proof the code works — even when the test is asserting on the implementation it just wrote rather than on observable behaviour. Tests that pass while the code is broken are the predictable outcome.

The fix is to **state the preference explicitly** in CLAUDE.md, **back it with mechanism** (import-linter, conftest hook, code review skill — see [Doc-as-gate](#doc-as-gate) below), and **call it out when reviewing AI-generated tests**: the first question on any new test is "what does this test fail on?" If the answer is "an internal refactor", the test is wrong even if it's currently green.

Two strong preferences keep tests honest:

### Strong preference for fakes over mocks

A **fake** is a working alternative implementation (in-memory database, a real HTTP server on `localhost`, a stub queue that actually queues). A **mock** is a recording of expected calls — `mock.foo.return_value = 42` and `assert mock.foo.called_once_with(...)`.

**Why fakes win**:

- Fakes exercise the same call paths as production — only the storage / network is swapped
- Fakes break when the contract changes; mocks pass with stale assumptions
- Fakes don't tie tests to implementation details, so refactors don't break them
- Fakes catch wiring bugs (wrong serialisation, missing fields, off-by-one ordering); mocks paper over them

**Defaults**:

- Database → in-memory SQLite or `pytest-postgresql` / `testcontainers`, never `mock.patch('db.session')`
- HTTP → `responses`, `httpx.MockTransport`, or `pytest-httpserver`, never `mock.patch('requests.get')`
- Queue → in-memory channel or `fakeredis`, never `mock.patch('queue.publish')`
- Filesystem → `tmp_path` fixture, never `mock_open`
- Time → `freezegun` or injected clock, never `mock.patch('time.time')`

**When mocks are acceptable**:

- At **system boundaries** you can't fake (third-party paid APIs, hardware, irreplaceable side-effects)
- For **dispatch / router** functions where the delegation IS the observable behaviour
- For **side-effect verification on outbound boundaries** (sent email, published metric) where there's nothing to check downstream
- **Internal mocks with a documented reason** — this is a *preference*, not a ban. Real codebases (e.g. [`jamma`](https://github.com/michael-denyer/jamma) — a fast Python/C reimplementation of GEMMA for large-scale GWAS, where some internal seams wrap C extensions or BLAS calls that aren't worth faking) have legitimate cases where an internal function is the pragmatic seam: the dependency is too expensive to fake, the integration cost outweighs the test value, the function wraps an external call that's already mocked deeper, or the test would otherwise duplicate an integration test elsewhere. The rule is "don't mock internal code by default" — when you do, leave a comment explaining why and what the boundary alternative would have been. If the reason ages badly, the comment makes the test easy to fix.

When you reach for an internal mock, ask: *would a fake be feasible if I refactored?* If yes, prefer the refactor. If no, document why and move on. The goal is fewer brittle implementation-coupled tests, not zero of them.

### Test outcomes and boundaries, not internals

Assert on what the system **produces**, not how it produces it.

**Test outcomes**:

- Return values, exit codes, raised exceptions
- Files written (existence, contents, schema)
- DataFrame shape, columns, expected values
- HTTP responses (status code, headers, body)
- Database state after the operation (`SELECT` and assert)
- Stdout / stderr for CLI tools

**Test boundaries** (where YOUR code meets something else):

- Is the right SQL emitted? (capture and inspect, don't mock the cursor)
- Is the right HTTP request sent? (use `responses` to record, then assert on the recorded request)
- Is the right MLflow run logged? (assert against the local tracking server)
- Is the right Spark plan generated? (`.explain()` and check structure)

**Anti-patterns to refuse**:

- `mock.assert_called_once_with(...)` for an internal helper. The function existing isn't behaviour — what it produces is.
- `assert mock.call_count == 3`. Implementation churn — refactor to one well-named call and the test breaks for no reason.
- Testing private methods directly. If it's worth testing, it's worth a public API. Otherwise it's tested transitively via the public path.
- Asserting log messages as a primary check. Logs are observability, not contract.
- `setUp` heavy in mocks across an entire test class. The mock setup is now the test surface; the real code is barely exercised.
- "Round-trip mock" tests that mock `foo()` to return X then assert that `foo()` was called and returned X. These tautologies pass forever.

**Rule of thumb**: if you can refactor the code under test without changing any test assertion, the test is testing the right thing. If a renaming or extracting refactor breaks the tests, the tests are coupled to internals — rewrite them to assert on outcomes.

### Doc-as-gate

State this in `<repo>/CLAUDE.md` AND back it with mechanism wherever possible:

- `import-linter` rule: tests in `tests/integration/` cannot import `unittest.mock`
- pytest plugin or `conftest.py` hook that fails if a test patches a path inside the project's own package (vs at a boundary)
- Test-tier marks (`@pytest.mark.unit` / `@pytest.mark.integration`) with a CI step that runs integration tests against real fakes, not mocks
- Code review checklist item enforced via `pr-review-toolkit` or a `silent-failure-hunter` invocation

This principle is doc-as-rule needs doc-as-gate, applied to test discipline: a `TESTING.md` saying "prefer fakes" without a check is fiction the day after it's written.

---

## Model selection strategy

The single biggest cost lever in a Claude Code setup is **using the right model for the task**. Defaulting everything to Opus burns budget on work Sonnet would do equally well; defaulting everything to Haiku produces shallow output on work that needs reasoning. The tradeoff is **quality × latency × cost**, and the right point on that curve depends on what the task actually requires.

### The three-model spectrum

| Model | Strengths | Cost / latency | Use for |
| --- | --- | --- | --- |
| **Opus 4.7** | Deepest reasoning, best at architectural tradeoffs, multi-step planning, debugging tangled state, novel design, large-context synthesis (1M token window) | Highest cost, slowest first-token | Hard thinking work — design, refactors that span modules, debugging that requires holding many hypotheses, anything where being wrong is expensive |
| **Sonnet 4.6** | Strong everyday coding, follows specs reliably, good tool use, balanced reasoning depth | Mid-tier on both axes | Default for execution work — implementing a spec'd feature, writing tests against a clear contract, code review of bounded diffs, routine refactors |
| **Haiku 4.5** | Fast, cheap, excellent for narrow / well-defined tasks, tool dispatch, classification, formatting, summarisation | Cheapest, fastest | Mechanical work — log triage, doc lint, classification, "find me X in this file", summarising a long output, generating boilerplate from a template |

### Decision rules

**Pick Opus when**:

- The task requires holding multiple hypotheses simultaneously (debugging, design exploration)
- You're synthesising across many files (>10) or a large context window
- A wrong answer is expensive to correct (architecture decisions, schema design, security-sensitive code)
- The task is open-ended ("what's the right way to structure this?")
- You're spending the *user's* time waiting on the answer — quality dominates latency

**Pick Sonnet when**:

- The shape of the work is clear (spec, plan, or PLAN.md exists)
- You're executing — implementing, testing, reviewing a bounded diff
- The task fits comfortably in a normal context window
- You want a good answer fast, not the best possible answer slow
- This is the right default for ~70% of coding work

**Pick Haiku when**:

- The task is well-defined and narrow (one file, one function, one classification)
- You'll run it many times (batch processing, CI checks, log triage)
- Tool dispatch / orchestration where the *thinking* is in the prompt, not the model
- Latency matters more than depth (interactive chat with quick turns)
- Cost matters because you're running at scale

### Subagent model overrides

Subagents can run on a different model than the parent. Use this aggressively to control cost:

```text
Main agent (Opus, doing planning)
  ├─ Subagent (Sonnet, executing the plan)
  ├─ Subagent (Haiku, classifying log lines in parallel)
  └─ Subagent (Sonnet, running tests and reporting)
```

Patterns that pay off:

- **Plan in Opus, execute in Sonnet** — design once, implement many. The planning context is small but high-stakes; execution is large but routine.
- **Dispatch parallel Haiku subagents** for embarrassingly-parallel work (e.g. "classify each of these 200 issues") — main agent stays Sonnet/Opus to synthesise results
- **Research in Sonnet, summarise in Haiku** — let Sonnet do the open-ended search, hand the raw output to Haiku to compress

The Agent tool's `model` parameter takes precedence over the subagent definition's frontmatter, which takes precedence over the parent's model. Set it explicitly when the cost differential matters.

### Cost / latency intuition

Rough relative cost per token (subject to change — verify on Anthropic's pricing page):

- **Opus** ≈ 5× Sonnet ≈ 25× Haiku on input
- **Opus** ≈ 5× Sonnet ≈ 12× Haiku on output

A long Opus session at 1M context can be 100× the cost of the same task split into Haiku-classified + Sonnet-executed work. The savings compound when you `/clear` between unrelated tasks (smaller context → smaller bill on every subsequent turn).

### Anti-patterns

- **Defaulting to Opus for everything** — the most common mistake; pays 5× for Sonnet-quality output on Sonnet-shaped work
- **Using Haiku for open-ended reasoning** — you'll get fluent shallow answers that look right and are wrong; the savings evaporate when you have to redo the work
- **Switching models mid-task without `/clear`** — the new model inherits the previous transcript; you pay for the long context without the original model's reasoning
- **Not using subagent overrides** — running batch / parallel work on the parent model is the fastest way to overspend

### Rule of thumb

> **Plan in Opus, execute in Sonnet, dispatch in Haiku.**

Three-line decision tree for a fresh task:

1. Is the *shape* of the work unclear? → **Opus** (design / debug / refactor planning)
2. Is the shape clear and you're implementing? → **Sonnet** (default execution)
3. Is the work narrow and repetitive? → **Haiku** (or a Haiku subagent)

If you're not sure, start in Sonnet. Escalate to Opus when you hit a reasoning wall; drop to Haiku when you find yourself doing the same thing repeatedly.

---

## Prompt tips — getting more from Claude

Patterns that consistently produce better output. Cheap to adopt; compound across sessions.

### Phrasing patterns

- **"I asked you this yesterday — what else would you recommend / do / improve?"** Goes deeper than a fresh "what should I do about X?" because Claude treats it as iterative refinement, surfaces the second-tier ideas it skipped the first time, and stays consistent with prior context. Use when you suspect the first answer was the obvious 80% and you want the long tail.
- **"Ideas / improvements for version 2"** is richer than "what would you improve?" The framing implies a real next iteration with room for ambition, so Claude proposes structural changes rather than surface tweaks. "What would you improve" gets you typo-fixes; "ideas for v2" gets you architectural critique.
- **Constraints are often more effective than goals.** "Implement X" is open-ended; "Implement X without adding any new dependencies, in under 100 lines, using only the stdlib" forces a narrower, sharper solution. Constraints crystallise tradeoffs the model would otherwise paper over. State what's *off-limits*, not just what you *want*.
- **State the audience.** "Explain this for a senior engineer reviewing the PR" yields a different, better answer than "explain this." Claude calibrates depth, jargon, and what to omit based on who's reading.
- **Ask for the worst case first.** "What's the most likely way this goes wrong?" or "What would a hostile code reviewer say?" pulls out the criticism Claude would otherwise soften. Pair with the eventual recommendation.
- **Specify the form, not just the content.** "Give me a 3-row comparison table" or "respond in under 200 words, no headings" beats letting Claude default to verbose markdown. Form constraints save tokens and force prioritisation.

### Multimodal and tool patterns

- **Paste images.** Claude is genuinely good at interpreting screenshots — UI bug reports, error dialogues, chart annotations, whiteboard photos, architecture diagrams from a slide deck, even hand-drawn sketches. Faster than transcribing. Particularly strong on: reading log output from a terminal screenshot, pointing at a specific element ("the spinner in the top right"), explaining a chart, and reverse-engineering a UI layout into code.
- **Spawn subagents explicitly.** Ask Claude to "use a subagent to research X while you work on Y" and it will dispatch the right Agent in parallel — keeps the main context lean, runs independent work concurrently. Skills like `superpowers:dispatching-parallel-agents` formalise this; in plain prompts, just say "in parallel" and "independently" and Claude will reach for the Agent tool. Best for: web research, broad code searches, generating multiple drafts, and any "two unrelated questions" turn.
- **Clear context on new tasks.** When switching to an unrelated task, run `/clear` (or start a new session). The previous transcript is otherwise loaded into every prompt — that's drift risk (the model anchoring on stale assumptions) and token cost. The signal is: if the next task wouldn't benefit from the previous task's context, clear. Don't be precious about losing chat history; the work is in the files, not the chat.
- **Parallelise independent steps.** When you hand Claude a plan with steps that don't depend on each other ("update the README, regenerate the diagrams, bump the version"), explicitly say *"do these in parallel"*. Claude will batch tool calls into a single message instead of serialising them — wall-clock time drops noticeably on multi-file edits. The harness already supports parallel tool calls; you just need to authorise them.
- **Use the `Monitor` tool for long-running watches.** When you need to wait on something that streams output over time — a CI run, a long test suite, a deploy, a build, `tail -f` on a log, a `kubectl rollout status` — ask Claude to *monitor* it rather than polling with sleeps. Each new line of stdout becomes a notification, so Claude wakes only when something happens, not on a fixed cadence. Common uses:
  - **CI / GitHub Actions**: `gh run watch <run-id>` under Monitor — Claude reports failures the moment they appear, then can dispatch a fix
  - **Test suite watches**: `pytest --looponfail` or `cargo watch` — Claude sees each test result as it streams
  - **Deploy / rollout**: `kubectl rollout status deploy/foo` — pause until ready, fail fast on rollback
  - **Polling for state**: `until <check>; do sleep 2; done` under Monitor — wakes Claude exactly when the loop exits, instead of you guessing the right delay
  Don't combine `Monitor` with manual `sleep` loops or `ScheduleWakeup` retries — pick one mechanism. If you only need a one-shot "wait until done", `Bash` with `run_in_background` plus a final read is simpler; reach for `Monitor` when the *event stream* itself matters.

### Iteration patterns

- **Feed back on the answer, not the topic.** "That's too generic — give me three specific examples from the codebase" iterates faster than re-asking the original question. Treat each turn as editing the previous response.
- **Ask for the inverse.** After Claude proposes an approach, ask "what would the case for *not* doing this look like?" — surfaces tradeoffs and alternatives without restarting the conversation.
- **Force a confidence tag.** "Rate your confidence on each of these 1–5 and explain anything below 3." Cheap way to surface the parts of an answer that are guesses dressed as facts.
- **Pre-seed the format.** If you know you want a markdown table, paste the empty table headers in your prompt. Claude fills the cells; you skip the back-and-forth on formatting.

### Verification and source references — avoiding BS

LLMs hallucinate fluently — confident, well-formatted, plausibly-named, and wrong. The fix is not "trust less"; it's to **demand evidence as part of the prompt**, so verification is built into the answer rather than tacked on afterwards.

- **Demand citations from the start.** "Answer this and quote the exact line(s) of code or docs that support each claim. If you can't quote a source, say so explicitly." Bakes verification into the response shape.
- **Require file paths and line numbers.** "Cite every claim as `path/to/file.py:42-51`." With them, a "the auth flow does X" claim is verifiable in seconds — click the link, read the lines, confirm. Without them, you have to take it on trust.
- **Force unknowns into a separate section.** "Split your answer into **Verified** (you can point to source) and **Inferred** (your reasoning). Never mix them." Stops fabricated detail from hiding inside correct context.
- **Verify before recommending.** For *actionable* answers ("use the `--frob` flag", "call `client.do_x()`"), require Claude to *run the help command*, *grep for the symbol*, or *fetch the official docs* before recommending. The Claude Agent SDK and Anthropic SDK have the tools to verify; lazy prompts let it skip them.
- **Cross-check with a second tool.** When Claude cites an API, ask it to verify by calling the function with `--help`, reading the source, or fetching the documentation page. Two independent confirmations beat one confident assertion.
- **Watch for these BS tells**:
  - **Plausibly-named functions / flags** that don't exist (e.g. `client.batch_async()` when only `client.batch()` exists). Always grep before trusting.
  - **Round-numbered "facts"** ("studies show 80% of teams…") with no link.
  - **Versions and dates without a source** ("as of v3.2…" — verify the version exists and the claim is in the changelog).
  - **Documentation paraphrase that sounds like the docs but isn't** — ask for a direct quote.
  - **Library APIs from training data** that have since changed. Always verify against the installed version.
- **Use stale-claim detection on memory.** A memory that says "function `X` exists" was true when written; it may not be true now. Before recommending from memory, grep / read to confirm. The `auto memory` instructions in the system prompt cover this — extend the discipline to anything Claude tells you that it claims to "remember".
- **Date-stamp everything time-sensitive.** "As of 2026-04, the recommended pattern is…" gives you a verification target ("is this still true in 2026-09?") instead of an indefinite claim.
- **Reject undocumented confidence.** If Claude says "I'm confident", "this should work", "this is the standard way" without an underlying source, push back: "What's the source? Verify before answering."

**Prompt template for high-stakes questions**:

```text
Question: <your question>

Constraints:
1. Cite every factual claim with a file path + line range, a docs URL, or a `<command> --help` invocation
2. Split your answer into "Verified" (sourced) and "Inferred" (reasoning) — never mix
3. If you can't verify a claim, say "I can't verify this" — do not guess
4. Before recommending any API / flag / library function, run a check that proves it exists in the version we're using
5. Rate confidence 1–5 on each Verified claim; anything below 4 explain why
```

This template is verbose for routine work — but for any answer where being wrong is expensive (security, schema design, production behaviour, infra config), the cost of the extra tokens is dwarfed by the cost of acting on a hallucination.

### Token economy — reduce spend without losing quality

Tokens cost money and dilute attention. Every paragraph of preamble, every redundant recap, every "let me think about this…" line is paid for and adds noise. Two levers: shrink **output verbosity** and shrink the **long tail of agent work** (test → review → re-verify → re-test cycles).

**Shrink output verbosity**:

- **Set a verbosity contract up-front.** "Reply in under 100 words, no preamble, no recap, no closing summary." Saves 30–50% on routine turns.
- **Suppress the recap.** "Don't restate what you just did — I read the diff." Claude's default closer ("I've now done X, Y, Z…") is wasted on most users.
- **Ban acknowledgement openers.** "Skip phrases like 'Great question', 'I'll help you', 'Let me…' — start with the answer." Often 50–100 tokens per turn, compounding across long sessions.
- **Demand structured output for structured tasks.** "Return only a JSON object with keys X, Y, Z." Eliminates prose framing entirely.
- **Cap thinking-out-loud.** When a task doesn't need extended reasoning, say so: "Don't deliberate — give the answer directly." For genuinely hard problems, the inverse is also true (think more, output less).
- **Match response length to question length.** A one-sentence question rarely needs a multi-paragraph answer. Tell Claude this once in CLAUDE.md and stop paying for it on every turn.
- **Use code references, not code paste.** "Point to the relevant lines as `file.py:42-51` instead of pasting the function back at me." Saves output tokens and improves clickability.

**Shrink the long tail of agent work**:

The expensive part of agent loops is rarely the first useful action — it's the test/review/re-verify/re-test/clarify/re-plan tail that follows. Aim it carefully.

- **Verify once, not after every step.** Default agent loops re-verify between each subtask ("I've done X, let me check it works… now I'll do Y, let me check that…"). For low-risk work, batch verification: "do all three tasks, then run the tests once at the end."
- **Don't ask for re-confirmation on completed work.** Once a step is verified, it's done — don't include it in the next verification pass. State explicitly: "tests for steps 1–3 passed earlier; only re-run tests touching step 4."
- **Cap the review depth.** "Don't add a code review pass; the pre-commit hook handles it." If `prek` and CI already enforce style and types, an inline review pass is duplication.
- **Stop the test-fix loop at N iterations.** "If the test fails twice, stop and ask me — don't loop." Prevents Claude from churning through 10 failed attempts at a flaky test.
- **Decide once, execute many.** Avoid "should I do A or B?" for every subtask once you've made the architectural call. Encode the decision in the plan ("we're going with A throughout — don't re-litigate") so subagents don't each re-derive it.
- **Skip optional gates on draft work.** Code review, security audit, and verification phases (e.g. GSD's `/gsd-secure-phase`, `/gsd-code-review`) are valuable on PR-ready work and waste on prototypes. Use `/gsd-quick` or skip them explicitly when you're spiking.
- **Trust the type checker and linter.** "If `pyrefly` and `ruff` pass, don't separately re-read the diff for type/style." Mechanical checks already enforced should not be re-done by the model.
- **Don't ask Claude to summarise its own work.** The diff is the summary. End-of-turn recaps re-narrate work you can already see.
- **Reduce subagent reporting overhead.** When dispatching subagents, cap the report size: "report in under 200 words" or "return a single line + verdict." Long subagent transcripts re-enter the parent context and recompound.

**Session-level levers**:

- **`/clear` aggressively** between unrelated tasks. The previous transcript is loaded into every prompt — that's drift risk *and* token cost.
- **Right-size the model.** See [Model selection strategy](#model-selection-strategy) — running Haiku-shape work on Opus is the largest single source of overspend.
- **Enable prompt caching** where you can. CLAUDE.md is cached automatically; structure long stable preambles to maximise hit rate.
- **Use the 1M context window deliberately, not by default.** Claude Code Opus has a 1M window, but a long-context turn costs roughly proportional to its context. Trim what's loaded.
- **Watch your bill weekly.** Most users overspend by 3–5× before they notice. A monthly check is too late; weekly catches drift.

**Anti-pattern**: micro-managing token spend on a small task. The five seconds spent shaping a "be terse" instruction can outweigh the savings on a 200-token answer. Apply these rules at the **session and CLAUDE.md level** so they cost nothing per turn — set the verbosity contract once globally, then forget about it.

### Prompt anti-patterns

- **"Are you sure?"** triggers Claude to second-guess correct answers as readily as wrong ones. Ask for *evidence* instead: "show me the line that confirms that" or "verify by running the tests".
- **Vague praise mid-task** ("good job, keep going") burns tokens without redirecting. Use turns for steering, not encouragement.
- **Asking the same question across many sessions** when the answer should be persistent — promote it to memory, CLAUDE.md, or a skill (see the prompts → skills principle).

---

## Install instructions

Run these in order on a fresh machine. All commands are macOS / zsh.

### 1. Claude Code itself

```bash
# Claude Code CLI + VS Code extension
# Install via the official installer at https://claude.com/code (sign in with Anthropic account)
# Then in VS Code: install the "Claude Code" extension from the marketplace
```

### 2. LSP servers (Python: `pyrefly-lsp-cc-plugin`)

[`pyrefly-lsp-cc-plugin`](https://github.com/michael-denyer/pyrefly-lsp-cc-plugin) wires Meta's `pyrefly` type checker into Claude as an LSP — fast, strict, and the LSP Claude actually uses for `findReferences` / `goToDefinition` / `hover`.

```bash
# Install pyrefly itself
uv tool install pyrefly

# Install the Claude Code plugin from GitHub
# (see https://github.com/michael-denyer/pyrefly-lsp-cc-plugin for the latest install command)
# In Claude Code, add the plugin source then:
#   /plugin add pyrefly-lsp-cc-plugin
# Verify Claude can call LSP tools (findReferences, goToDefinition, hover) against a Python file in your repo.
```

For other languages, install the matching LSP (`typescript-language-server`, `gopls`, `rust-analyzer`, etc.) and ensure VS Code picks it up — Claude Code's LSP integration delegates to whatever the editor has running.

### 3. `bd` (beads)

```bash
# Install
brew install steveasleep/beads/bd
# or via Go
go install github.com/steveasleep/beads/cmd/bd@latest

# Initialize in a repo
cd <repo>
bd init

# Daily workflow
bd ready            # what can I work on?
bd create -t bug -p 1 -- "description"
bd show <id>
bd update <id> --status in_progress
bd close <id>
bd sync             # commit + push beads database
```

`bd` stores its database in `.beads/` inside the repo and syncs via git, so the issue list travels with the code.

### 4. Pre-commit (via `prek`)

```bash
# Install prek — Rust rewrite of pre-commit, drop-in compatible
brew install prek
# or
cargo install prek
# or
uv tool install prek

# In your repo: write .pre-commit-config.yaml (see Tier 1 above), then:
prek install                          # installs git hook (same hook contract as pre-commit)
prek run --all-files                  # one-time full sweep

# Node tools (maid, mmdc) install on first run via `language: node` — no manual step
# Existing .pre-commit-config.yaml files work unchanged — no migration needed
```

If you have a CI step that runs hooks, swap `pre-commit run` for `prek run` there too. The hook config schema and `repo:` / `rev:` / `hooks:` shape are identical.

### 5. `superpowers` skills

```bash
# In Claude Code:
/plugin add superpowers
# Verify with `/skill` to see brainstorming, systematic-debugging, etc. listed
```

### 6. `pr-review-toolkit`

```bash
# In Claude Code:
/plugin add pr-review-toolkit
# Invoke per-PR via the Agent tool with subagent_type=silent-failure-hunter (etc.)
```

### 7. `code-review-graph` (large repos only)

```bash
# Install as uv tool with extras for graph algorithms and embeddings
uv tool install code-review-graph --with python-igraph --with sentence-transformers

# Build the graph for a repo
cd <repo>
crg build

# In Claude Code: /plugin add code-review-graph
# Use the embed_graph MCP function for semantic search across the codebase
```

See `~/.claude/code-review-graph.md` for full recipe.

### 8. `codemap` / `codemap-gr` skills (bundled with this repo)

```bash
# Clone or have this repo locally, then symlink into your skills dir:
ln -s "$PWD/skills/codemap" ~/.claude/skills/codemap
ln -s "$PWD/skills/codemap-gr" ~/.claude/skills/codemap-gr

# Verify in Claude Code:
#   /skill codemap
# Then ask: "Generate a code map for this repo" → produces docs/CODEMAP.md
```

`codemap-gr` requires `code-review-graph` to be installed and a graph built for the repo. Without CRG, use plain `codemap` — same output format, different source of truth (filesystem walk vs knowledge graph).

### 9. `GSD` (per-project, on large efforts)

```bash
# In Claude Code, inside the target repo:
/plugin add gsd
/gsd-new-project        # bootstrap .planning/ for a fresh project
# or
/gsd-ingest-docs        # ingest existing ADRs/PRDs/SPECs into .planning/
```

### 10. Python project bootstrap

```bash
# In a new repo:
uv init
uv add --dev pytest pytest-randomly pytest-xdist pytest-timeout pytest-cov ruff
# Add to pyproject.toml:
#   [tool.pytest.ini_options]
#   addopts = "-n 3 --timeout=30 --cov=mypackage --cov-report=term-missing"
```

### 11. CI hardening (one-time per repo)

```bash
# Add Dependabot for actions
mkdir -p .github
cat > .github/dependabot.yml <<'EOF'
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
EOF

# Add osv-scanner workflow
# See https://google.github.io/osv-scanner/github-action/ for the pinned-SHA snippet
```

---

## The meta-recommendation

The biggest mistake with a fresh setup is installing everything that *sounds* useful, then drowning in slash commands and skills you never remember to invoke. Tooling sprawl has a real cost: it dilutes your attention and Claude's context window.

**Sequence**:

1. Day one: LSP + pre-commit + `superpowers` skills + `bd` + Tier 1 settings
2. First PR: add `pr-review-toolkit`
3. First time grep stops scaling: add `code-review-graph`
4. First greenfield multi-week project with unclear requirements: consider GSD

Add tools when you feel a specific gap, not preemptively.
