# AGENTS.md Semantic Indexing Strategy (excluding `addons/`)

## Goal
Create one `AGENTS.md` file per folder (except anything under `addons/`) so coding agents can:
- understand ownership and architecture quickly,
- map intent to likely files,
- choose high-signal search commands,
- avoid scanning the whole repository for every task.

## Principles
1. **Directory-local scope only**: each `AGENTS.md` describes only the folder where it lives and points to children.
2. **Semantic vocabulary first**: include domain words users ask for (e.g., "ORM", "registry", "HTTP routing", "migration").
3. **Search recipes, not prose**: provide `rg` command patterns for common intents.
4. **Token-efficient structure**: stable sections with short bullets so agents can parse quickly.
5. **Cross-links as index edges**: every `AGENTS.md` lists "if you need X, jump to Y".

## Rollout approach

### 1) Build a folder inventory excluding `addons/`
Use an automated pass to list directories and group by depth:
- depth 0/1: root architecture map,
- depth 2+: specialized maps.

Recommended command:
```bash
find . -path './addons' -prune -o -type d -print | sort
```

### 2) Define a strict AGENTS.md template
Use one compact template everywhere to keep output predictable:

```md
# AGENTS.md — <path>

## Purpose
What this folder is responsible for.

## Contains
Key files and subfolders with one-line meaning.

## Semantic tags
Comma-separated concepts likely to appear in user prompts.

## Common tasks -> where to look
- Task phrase -> file(s)/dir(s)

## Search recipes
- `rg "<pattern>" <path>` for <intent>

## Boundaries
What is *not* in this folder; where to go instead.

## Child indexes
Relative links to deeper `AGENTS.md` files.
```

### 3) Seed top-level AGENTS.md files first
Prioritize folders that route most requests:
- `odoo/`
- `doc/`
- `debian/`
- `setup/`
- repository root

These should behave as "directory routers" and minimize blind searching.

### 4) Add deeper AGENTS.md files only where entropy is high
Create per-folder indexes in places with many files or mixed concerns. Keep tiny leaf folders minimal (3–6 bullets).

### 5) Add semantic tags with query intent in mind
For each folder, include tags in three classes:
- **Domain tags**: business/framework terms (e.g., "module loading", "RPC", "i18n").
- **Artifact tags**: file-type terms (e.g., "CLI", "manifest", "tests").
- **Action tags**: user verbs (e.g., "debug", "extend", "upgrade", "profile").

### 6) Encode search patterns that avoid repo-wide scans
Every AGENTS should include scoped commands, such as:
- `rg "class .*Controller" odoo/`
- `rg "def .*_rpc" odoo/`
- `rg "migration|upgrade" doc/`

Use positive path scoping and avoid expensive recursive grep variants.

### 7) Add "decision edges" between folders
Example edges:
- "Need packaging? jump to `setup/AGENTS.md`"
- "Need runtime internals? jump to `odoo/AGENTS.md`"
- "Need release/process docs? jump to `doc/AGENTS.md`"

This turns all AGENTS files into a navigable graph.

### 8) Validate quality automatically
Run a lint/check script that verifies:
- each non-`addons/` directory has `AGENTS.md` (or explicit exemption),
- required sections exist,
- links to child AGENTS are valid,
- commands are path-scoped,
- max size threshold (e.g., <= 120 lines).

### 9) Keep maintenance cheap
Add CI check + pre-commit helper:
- if a new directory is introduced, require an `AGENTS.md` or exemption note,
- fail on stale child links,
- encourage incremental edits rather than large rewrites.

## Practical generation workflow
1. Generate directory tree (excluding `addons/`).
2. Auto-create draft `AGENTS.md` with template.
3. Fill top-level and high-traffic folders manually.
4. Auto-backfill low-traffic folders with concise stubs.
5. Run lint and fix gaps.
6. Commit in phases: "root/router", then "deep indexes", then "lint/polish".

## Suggested minimal metadata per folder
- `owner_hint`: team or subsystem owner (if known)
- `stability`: stable | changing | experimental
- `last_reviewed`: date
- `related_tests`: where tests usually live

This metadata helps agents decide confidence and where to validate edits.

## Example root AGENTS.md outline
At repo root, include:
- a one-screen architecture map,
- "start here" triage by user intent,
- explicit exclusion note for `addons/`,
- links to all first-level folder AGENTS files,
- a short command cookbook for common searches.

## What success looks like
- Agents can route to likely folders in one hop.
- Search commands are mostly folder-scoped (not whole-repo).
- Fewer irrelevant files opened per task.
- New contributors can answer "where should this change go?" quickly.
