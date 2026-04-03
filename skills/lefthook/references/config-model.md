# Config Model

Use this reference when you need to choose between top-level config keys, hook-level structure, and merge behavior.

## Top-Level Keys

Common top-level keys:

- `min_version`
- `lefthook`
- `source_dir`
- `source_dir_local`
- `rc`
- `output`
- `colors`
- `extends`
- `no_tty`
- `assert_lefthook_installed`
- `skip_lfs`
- `no_auto_install`
- `install_non_git_hooks`
- `glob_matcher`
- `remotes`
- `templates`
- any hook name such as `pre-commit`, `commit-msg`, `pre-push`

Prefer using only the keys that solve the current repo need. Most simple setups need only hook sections.

## Hook-Level Structure

Each hook can contain:

- `parallel`
- `piped`
- `follow`
- `fail_on_changes`
- `fail_on_changes_diff`
- `files`
- `exclude_tags`
- `exclude`
- `skip`
- `only`
- `setup`
- `jobs`
- `commands`
- `scripts`

Choose structure as follows:

- Use `commands` for simple named `run:` entries.
- Use `jobs` for ordered lists, groups, nested flow, or merge-friendly named jobs.
- Use `scripts` when running files from `source_dir/<hook>/`.

## `commands` vs `jobs` vs `scripts`

### `commands`

Use for the common case:

```yml
pre-commit:
  commands:
    lint:
      glob: "*.{js,ts}"
      run: pnpm eslint --fix {staged_files}
      stage_fixed: true
```

Benefits:

- compact;
- readable;
- good for named tasks that are easy to override.

### `jobs`

Use when you need grouping, ordering, or mixed behaviors:

```yml
pre-commit:
  parallel: true
  jobs:
    - name: frontend checks
      root: frontend/
      group:
        piped: true
        jobs:
          - run: pnpm lint {staged_files}
          - run: pnpm test
    - run: go test ./...
      root: backend/
```

Benefits:

- support lists and nested groups;
- allow named jobs that merge across configs;
- work well when some jobs use `run` and others use `script`.

### `scripts`

Use when the repo wants executable hook files under the configured script directory:

```yml
commit-msg:
  scripts:
    "template_checker":
      runner: bash
```

The corresponding executable lives under `.lefthook/commit-msg/template_checker` unless `source_dir` changes.

## Placeholders

Use these in `run:` values:

- `{files}`: result of a custom `files:` command
- `{staged_files}`: files staged for commit
- `{push_files}`: committed but unpushed files
- `{all_files}`: all tracked files
- `{cmd}`: original command string, useful for wrappers
- `{0}`: all hook arguments joined
- `{1}`, `{2}`, ...: positional hook arguments
- `{lefthook_job_name}`: current job or command name

Guidance:

- Prefer `{staged_files}` in `pre-commit`.
- Prefer `{push_files}` in `pre-push`.
- Prefer `{all_files}` for tools that should see the full project.
- Use `{1}` in `commit-msg` for tools like `commitlint --edit {1}`.

## Merge Behavior

Order of application:

1. main config
2. `extends`
3. `remotes`
4. `lefthook-local.*`

Implications:

- Put team-shared defaults in the main config.
- Use `extends` for reusable local files.
- Use `remotes` for shared multi-repo configs.
- Use `lefthook-local.*` for per-developer overrides, skips, wrappers, or Docker-specific changes.

## Filtering And Fixing

Use filters to keep hooks fast:

- `glob`
- `exclude`
- `file_types`
- `root`
- `files`

Use `stage_fixed: true` when the command rewrites tracked files and should add them back to the index after running.

Common mistake patterns:

- using `{staged_files}` with a whole-repo formatter;
- forgetting `--force-exclusion` for RuboCop when using `{all_files}`;
- putting Docker-only wrappers into the shared config instead of `lefthook-local.*`.
