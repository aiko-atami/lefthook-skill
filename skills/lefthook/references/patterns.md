# Common Patterns

Use this reference when drafting a config quickly. Adapt commands to the repo's actual package manager and scripts.

## Minimal Pre-Commit

```yml
pre-commit:
  commands:
    lint:
      glob: "*.{js,ts,jsx,tsx}"
      run: pnpm eslint {staged_files}
```

## JS Or TS Lint And Format

```yml
pre-commit:
  commands:
    eslint:
      glob: "*.{js,jsx,ts,tsx}"
      run: pnpm eslint --fix {staged_files}
      stage_fixed: true
    prettier:
      glob: "*.{js,jsx,ts,tsx,json,md,yml,yaml}"
      run: pnpm prettier --write {staged_files}
      stage_fixed: true
```

Use this only if the repo already uses ESLint and Prettier. Otherwise swap in the project’s formatter or linter.

## Commit Message Validation

```yml
commit-msg:
  commands:
    commitlint:
      run: pnpm commitlint --edit {1}
```

For Commitizen plus Commitlint:

```yml
prepare-commit-msg:
  commands:
    commitzen:
      interactive: true
      run: pnpm exec cz --hook
      env:
        LEFTHOOK: "0"

commit-msg:
  commands:
    commitlint:
      run: pnpm commitlint --edit {1}
```

## Monorepo With `root`

```yml
pre-commit:
  commands:
    frontend lint:
      root: frontend/
      glob: "*.{js,ts,tsx}"
      run: pnpm eslint --fix {staged_files}
      stage_fixed: true
    backend test:
      root: backend/
      glob: "*.go"
      run: go test ./...
```

Use `root` when commands should execute from a subdirectory. Keep the trailing slash.

## Docker Or Local Wrappers

Prefer shared templates when the wrapper may change locally:

```yml
templates:
  wrapper:

pre-commit:
  jobs:
    - name: rubocop
      run: "{wrapper} bundle exec rubocop --force-exclusion -- {staged_files}"
```

Local override:

```yml
templates:
  wrapper: dip
```

For direct local wrapping of a single job:

```yml
pre-commit:
  jobs:
    - name: rubocop
      run: dip {cmd}
```

## Local Overrides

```yml
# lefthook-local.yml
pre-commit:
  commands:
    eslint:
      skip: true

post-merge:
  files: "git diff-tree -r --name-only --no-commit-id ORIG_HEAD HEAD"
  commands:
    install deps:
      glob: "package.json"
      run: pnpm install
```

Use `lefthook-local.*` for:

- developer-specific skips;
- Docker or `dip` wrappers;
- extra local hooks;
- private tooling that should not be committed to team config.

## Docs Hygiene

```yml
pre-commit:
  parallel: true
  commands:
    markdownlint:
      glob: "*.md"
      run: markdownlint-cli2 {staged_files}
    spelling:
      glob: "*.md"
      run: cspell {staged_files}
    links:
      glob: "*.md"
      run: lychee {staged_files}
```

Use this in documentation-heavy repositories. Keep the same core checks in CI as well.

## Post-Merge Pipeline

```yml
post-merge:
  files: "git diff-tree -r --name-only --no-commit-id ORIG_HEAD HEAD"
  commands:
    client-deps:
      glob: "{package.json,pnpm-lock.yaml}"
      run: pnpm install
    migrations:
      run: lefthook run migrations

migrations:
  piped: true
  files: "git diff-tree -r --name-only --no-commit-id ORIG_HEAD HEAD"
  commands:
    bundle:
      priority: 1
      glob: "Gemfile*"
      run: bundle install
    migrate:
      priority: 2
      glob: "db/migrate/*"
      run: bundle exec rails db:migrate
```

Use helper groups plus `piped: true` when later steps depend on earlier ones.

## Guarded Local `post-checkout` Script

```yml
post-checkout:
  scripts:
    local-sync:
      runner: bash
```

Use this with a script under `.lefthook/post-checkout/` when the logic is machine-specific or depends on tools such as Docker or `dip`. Add guard clauses in the script so it exits cleanly when the environment is not relevant.

## Debugging Flow

When a hook misbehaves, follow this order:

1. run `lefthook validate`
2. run `lefthook dump`
3. run `lefthook run <hook>`
4. retry with `--all-files` or explicit `--file`
5. inspect whether `glob`, `exclude`, `root`, and placeholders match the hook type
