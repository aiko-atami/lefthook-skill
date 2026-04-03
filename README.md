# lefthook

Skill for working with Lefthook in Git repositories. It helps set up or reshape `lefthook.yml`, choose the right config model for a hook, scope commands to the right files and directories, and debug why a hook behaves differently from what the repository expects.

## Install

Install command:

```bash
npx skills add aiko-atami/lefthook-skill --skill lefthook
```

## What This Skill Handles

- Bootstrapping Lefthook in a repository that does not yet have a working hook setup.
- Authoring or refactoring `lefthook.yml`, `.lefthook.yml`, or related local override files.
- Choosing between `commands`, `jobs`, and `scripts` based on the actual workflow complexity.
- Designing `pre-commit`, `commit-msg`, `prepare-commit-msg`, `pre-push`, `post-merge`, and custom `lefthook run <task>` flows.
- Scoping hook commands with `glob`, `exclude`, `file_types`, `root`, and file placeholders such as `{staged_files}` or `{push_files}`.
- Handling local-only behavior through `lefthook-local.*`, wrappers, templates, and machine-specific automation.
- Reviewing existing Lefthook configs for broken filters, unnecessary complexity, and missing `stage_fixed: true`.
- Reproducing and debugging hook behavior with `lefthook validate`, `lefthook dump`, and `lefthook run`.

## Supported Task Types

The skill is organized around 8 practical task types:

- `setup` - install or bootstrap Lefthook in a repository and choose the right config file format.
- `author` - write or update hook configuration for the current toolchain.
- `model` - decide when to use `commands`, `jobs`, `scripts`, helper groups, or templates.
- `hooks` - add or refine hook-specific behavior for commit, push, merge, and local automation.
- `scope` - align placeholders, globs, roots, and file selection with the intended hook semantics.
- `local` - move per-developer wrappers, skips, or Docker-specific behavior into `lefthook-local.*`.
- `review` - inspect an existing config for mistakes, regressions, and maintainability issues.
- `debug` - trace misbehavior with validation, merged-config inspection, and direct hook execution.

## Covered Lefthook Topics

- Config file discovery and format choices: `lefthook.yml`, `.lefthook.yml`, `.config/lefthook.yml`, YAML/TOML/JSON/JSONC variants, and matching `lefthook-local.*`.
- Hook structure: top-level hook sections, custom task groups, and the operational meaning of `parallel`, `piped`, `follow`, `files`, `setup`, `commands`, `jobs`, and `scripts`.
- Command modeling: `run`, `script`, `group`, named jobs, priorities, interactive commands, wrappers, and templates.
- File targeting: `glob`, `exclude`, `file_types`, `root`, custom `files:` commands, and placeholders like `{files}`, `{staged_files}`, `{push_files}`, `{all_files}`, `{cmd}`, and `{1}`.
- Common automations: staged linting and formatting, commit message validation, docs hygiene, monorepo hooks, and cautious post-merge or post-checkout workflows.
- Merge and override behavior: main config, `extends`, `remotes`, and `lefthook-local.*` precedence.
- Day-2 operations: `lefthook install`, `lefthook validate`, `lefthook dump`, and `lefthook run`.

## Included Templates And Recipes

The skill ships with reusable starter assets:

- `assets/minimal-pre-commit.yml` - minimal staged-file linting baseline.
- `assets/js-ts-pre-commit.yml` - JS/TS pre-commit setup with ESLint and Prettier autofix.
- `assets/commit-msg.yml` - Commitizen plus Commitlint flow.
- `assets/monorepo-root.yml` - `root:`-based example for frontend/backend monorepos.
- `assets/lefthook-local.yml` - local override pattern for wrappers and machine-specific automation.

## Reference Base

The skill is backed by these reference files:

- `references/overview.md` - what Lefthook is, supported config files, install flow, and core CLI commands.
- `references/config-model.md` - top-level keys, hook structure, placeholders, merge order, and when to choose `commands` vs `jobs` vs `scripts`.
- `references/patterns.md` - ready-to-adapt patterns for pre-commit, commit-msg, monorepos, docs, local overrides, and debugging.
- `references/best-practices.md` - battle-tested guidance for fast hooks, autofixers, monorepos, local automation, and remote configs.
- `references/schema-cheatsheet.md` - exact field names and valid nesting without opening the full upstream schema.

## Default Assumptions

- Prefer preserving the repository's existing config filename and format instead of introducing a new one.
- Prefer YAML for new setups unless the repository already standardizes on TOML, JSON, or JSONC.
- Prefer `commands` for simple named tasks and escalate to `jobs` only when ordering, grouping, or mixed behaviors actually require it.
- Prefer repo-local toolchain commands over custom wrappers invented just for hooks.
- Prefer narrow file filters and the correct placeholder for the hook type to keep hooks fast and predictable.
- Prefer `lefthook-local.*` for local skips, Docker wrappers, or private tooling that should not become team-wide policy.
- Prefer `stage_fixed: true` only for commands that really rewrite tracked files.

## Boundaries And Guardrails

- Do not add a second main Lefthook config format to the same project.
- Do not use `{staged_files}` for tools that must analyze the entire repository.
- Do not keep Docker-only or machine-specific behavior in the shared config when `lefthook-local.*` is the right boundary.
- Do not treat `prepare-commit-msg` interactivity as a universal team default when GUI commits or non-TTY environments are common.
- Do not assume hooks need reinstall after every config edit; validate and rerun them first.
- Do not guess at exact option names when the schema or bundled references can confirm them.
