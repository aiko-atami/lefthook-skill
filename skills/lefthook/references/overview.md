# Lefthook Overview

Use this reference when you need the short explanation of what Lefthook is, how it is installed, which config files it reads, and which commands are relevant during setup and debugging.

## What It Does

Lefthook is a Git hook manager. It installs hook entrypoints into `.git/hooks` and, when those hooks fire, it loads the current Lefthook configuration and runs the configured commands or scripts.

Key consequences:

- Treat the config as the source of truth.
- Do not assume reinstall is needed after every config edit.
- Expect hooks to read the current config on each run.

## Common Install Routes

Pick the install route that matches the repository ecosystem:

- Node.js: `npm install lefthook --save-dev`
- Go: `go install github.com/evilmartians/lefthook/v2@<version>`
- Ruby: `gem install lefthook`
- Python: `pipx install lefthook`

After the binary is available, run:

```bash
lefthook install
```

Notes:

- `lefthook install` creates an empty `lefthook.yml` if no config exists.
- In Node projects using the `lefthook` package, installation may happen automatically in `postinstall`.
- Installing specific hooks is supported: `lefthook install pre-commit commit-msg`

## Supported Main Config Filenames

Use exactly one main config format per project.

- YAML: `lefthook.yml`, `.lefthook.yml`, `.config/lefthook.yml`
- YAML: `lefthook.yaml`, `.lefthook.yaml`, `.config/lefthook.yaml`
- TOML: `lefthook.toml`, `.lefthook.toml`, `.config/lefthook.toml`
- JSON: `lefthook.json`, `.lefthook.json`, `.config/lefthook.json`
- JSONC: `lefthook.jsonc`, `.lefthook.jsonc`, `.config/lefthook.jsonc`

Lefthook also supports local override files named `lefthook-local.*` in the same format family. If the main config uses a leading dot, the local override should use the leading dot as well.

## Useful Commands

```bash
lefthook install
lefthook validate
lefthook dump
lefthook run pre-commit
lefthook run pre-commit --all-files
lefthook run pre-commit --file path/to/file.ts
lefthook run pre-commit --job lint --tag frontend
```

Command intent:

- `install`: install Git hook entrypoints
- `validate`: validate config structure
- `dump`: show the effective merged config
- `run`: execute a hook or custom task directly

## Hook Model

Lefthook supports standard Git hook names and also custom task groups that can be invoked through `lefthook run`.

Common hooks:

- `pre-commit`
- `commit-msg`
- `prepare-commit-msg`
- `pre-push`
- `post-merge`
- `post-checkout`

Useful rule of thumb:

- Use `pre-commit` for staged-file checks and auto-fixes.
- Use `commit-msg` for commit message validation and rely on `{1}` to access the commit message file path.
- Use `pre-push` for slower checks on outgoing changes and prefer `{push_files}` when file scoping matters.
