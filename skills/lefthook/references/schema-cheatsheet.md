# Schema Cheatsheet

Use this reference when you need exact field names and valid nesting without loading the full upstream JSON schema.

## Top Level

```text
min_version
lefthook
source_dir
source_dir_local
rc
output
colors
extends
no_tty
assert_lefthook_installed
skip_lfs
no_auto_install
install_non_git_hooks
glob_matcher
remotes
templates
<hook-name>
```

## Hook Object

```text
parallel
piped
follow
fail_on_changes
fail_on_changes_diff
files
exclude_tags
exclude
skip
only
setup
jobs
commands
scripts
```

## Command Object

Required:

```text
run
```

Optional:

```text
files
root
fail_text
timeout
skip
only
tags
file_types
glob
exclude
env
priority
interactive
use_stdin
stage_fixed
```

## Job Object

One of these is required:

```text
run
script
group
```

Optional:

```text
name
runner
args
root
files
fail_text
timeout
glob
exclude
tags
file_types
env
interactive
use_stdin
stage_fixed
skip
only
```

Group object:

```text
root
parallel
piped
jobs
```

## Script Object

```text
runner
args
skip
only
tags
env
priority
fail_text
timeout
interactive
use_stdin
stage_fixed
```

## Special Values

`fail_on_changes` accepts:

```text
true
1
0
false
never
always
ci
non-ci
```

`glob_matcher` accepts:

```text
gobwas
doublestar
```

## Supported Git Hook Names

Common built-in hook names include:

```text
applypatch-msg
pre-applypatch
post-applypatch
pre-commit
pre-merge-commit
prepare-commit-msg
commit-msg
post-commit
pre-rebase
post-checkout
post-merge
pre-push
pre-receive
update
post-receive
post-update
post-rewrite
```

Lefthook also allows custom task names that can be executed with `lefthook run <name>`.
