# Project: claude-multi-manage

> command line utility for quickly starting and managing multiple claude sessions on the same host

## Stack
- bash only, very simple, with good help/usage functionality
- runtime deps: `bash`, `tmux`, `claude` on `PATH` — nothing else

## Conventions
- single script: `cmm` (executable, `#!/usr/bin/env bash`, `set -euo pipefail`)
- subcommands are `cmd_<name>` functions; per-command help lives in `help_for`
- sessions are plain tmux sessions named `claude-<name>`
- no generated files; no tests yet — verify with `bash -n cmm` and manual runs

## Local dev
- run: `./cmm help`, `./cmm list --all`
- syntax check: `bash -n cmm`
- install: `./cmm install` (to `~/.local/bin`) or `./cmm install --system`

## Notes for Claude
- Prefer small, focused commits with descriptive messages.
- Ask before adding new dependencies.
- Keep `.claude/settings.local.json` out of git (it holds machine-specific paths).
