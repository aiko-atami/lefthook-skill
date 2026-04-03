# Best Practices From Guides And Articles

Use this reference when a user explicitly asks for best practices, battle-tested setups, or rationale beyond raw option definitions.

Keep two rules straight:

- exact option names and merge behavior come from the bundled Lefthook docs and schema;
- the practices below come from official guide pages plus real-world articles linked from the upstream README.

Legend:

- `[Official]` means the recommendation is directly supported by the bundled Lefthook docs, schema, or official guide pages.
- `[Article-derived]` means it comes from README-linked articles and should be treated as workflow guidance, not a hard product rule.
- `[Mixed]` means the option exists in official docs, but the recommendation about when to use it comes from article examples and repeated real-world usage.

## Recommended Defaults

- `[Mixed]` Keep `pre-commit` fast. Prefer file-scoped linters, formatters, spell checks, and link checks against `{staged_files}`. Reserve whole-repo checks such as full test suites or full type checks for cases where incremental execution is unsafe.
- `[Official]` Run independent checks in parallel. Use `parallel: true` when tasks do not depend on each other.
- `[Mixed]` Use `stage_fixed: true` only for commands that actually rewrite tracked files in `pre-commit`.
- `[Article-derived]` Prefer `commit-msg` validation as the shared team baseline. Treat interactive `prepare-commit-msg` flows as optional because they depend on a terminal TTY.
- `[Mixed]` In monorepos, combine `root` and `glob` so each tool runs only where its manifest and config exist.
- `[Mixed]` Use custom groups plus `lefthook run <task>` when a hook needs to compose helper tasks.
- `[Mixed]` Keep environment-specific automation local. If a hook depends on Docker, `dip`, or private tooling, prefer `lefthook-local.yml` or a guarded script under `.lefthook/`.

## Patterns That Show Up Repeatedly

### 1. [Mixed] Fast parallel pre-commit hooks

Good shared configs usually put the cheapest, highest-signal checks in `pre-commit` and parallelize them. Common examples:

- ESLint, Prettier, RuboCop, typos, or style checkers on staged files
- link or Markdown checks only for touched docs files
- a full-project task only when partial execution would be misleading

Use this when you want fast feedback without making each commit feel like CI.

### 2. [Mixed] Autofixers should restage their changes

If a formatter or linter rewrites files, pair it with `stage_fixed: true`. This avoids the common failure mode where the tool fixes files but the commit still contains the pre-fix content.

Do not add `stage_fixed: true` to read-only validators.

### 3. [Mixed] Monorepo commands should be physically close to their toolchain

Article examples consistently run commands from the subdirectory that owns the toolchain. That means:

- `root: "client/"` for frontend package-manager tasks
- `root: "server/"` or another backend directory for language-specific tooling
- matching `glob` filters so jobs skip cleanly when that area was untouched

This keeps hooks fast and avoids failures from missing `package.json`, `Gemfile`, or similar manifests in the repo root.

### 4. [Article-derived] Interactive commit helpers are optional, not universal

`commitlint` in `commit-msg` is a safe default. `commitizen` or similar tools in `prepare-commit-msg` can be useful, but they are best treated as:

- terminal-first workflows;
- optional local developer helpers; or
- configurations guarded by `interactive: true` and `LEFTHOOK: 0`.

If a team uses GUI commits heavily, prefer making the interactive helper local-only and keep the validation hook shared.

### 5. [Mixed] Use helper groups for small pipelines

When one hook needs a short sequence with dependencies, split it into a helper task and invoke it with `lefthook run <task>`. This is cleaner than cramming unrelated logic into one long shell command.

Typical example:

- `post-merge` detects changed files and runs a helper group
- the helper group uses `piped: true` or priorities so dependency install happens before migrations

### 6. [Mixed] Post-merge and post-checkout hooks should be conservative

These hooks can automate useful local maintenance, but only when they fail safely.

Good practice:

- gate them by changed files;
- exit early when required tools are missing;
- skip when the current branch or checkout event is irrelevant;
- keep machine-specific behavior in `lefthook-local.yml` or scripts.

This is especially important for Docker-based workflows.

### 7. [Article-derived] Documentation deserves hook coverage too

For docs-heavy repositories, articles recommend treating docs as code:

- lint Markdown;
- spell-check edited docs;
- validate links;
- mirror the same checks in CI for consistency.

This is a strong fit for repos with `/docs`, generated references, or Markdown-driven websites.

### 8. [Mixed] Shared remote configs trade duplication for coordination

Centralized configs can reduce repeated setup across many repos, but they also require a clear team update flow. When using `remote` or `remotes`:

- keep the shared config narrow and boring;
- verify the effective merged result with `lefthook dump`;
- document how teammates pick up updates.

Use this only when the organization truly benefits from cross-repo standardization.

## Source Notes

These practices were distilled from the official guides and README-linked articles, especially:

- `5 cool (and surprising) ways to configure Lefthook for automation joy`
- `Lefthook: knock your team’s code back into shape`
- `Keeping OSS documentation in check with docsify, Lefthook, and friends`
- `Smooth PostgreSQL upgrades in DockerDev environments with Lefthook`
- `A deep dive into Lefthook for React and React Native`

Quick source map:

- `parallel`, `stage_fixed`, `root`, `lefthook-local.yml`, `remotes`, `extends`, `piped`, `priority`, and `lefthook run` are documented in the bundled Lefthook docs and schema.
- fast `pre-commit` composition, docs-focused hooks, interactive commit helpers, and cautious local Docker automation are reinforced by README-linked articles.
- when a recommendation is labeled `[Article-derived]`, do not present it as mandatory Lefthook behavior.

Keep this file as workflow guidance. For exact nesting, precedence, and supported keys, defer to the bundled schema and local docs.
