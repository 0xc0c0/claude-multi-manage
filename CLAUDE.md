# Project: claude-multi-manage

> command line utility for quickly starting and managing multiple claude sessions on the same host

## Stack
- bash only, very simple, with good help/usage functionality
- runtime deps: `bash`, `tmux`, `claude` on `PATH` — nothing else

## Conventions
- single script: `cmm` (executable, `#!/usr/bin/env bash`, `set -euo pipefail`)
- subcommands are `cmd_<name>` functions; per-command help lives in `help_for`
- sessions are plain tmux sessions named `claude-<name>`
- `update`/`restart`/`keepalive` read a session's screen with `tmux capture-pane`
  and drive it with `tmux send-keys`; they rely on strings Claude Code prints
  (`Resume this session with:`, `/rc failed`, `Restart to apply`, `esc to interrupt`)
- no generated files; no tests yet — verify with `bash -n cmm` and manual runs

## Docs
- `docs/user_stories.md` is the master record of user stories (`US-<n>` IDs,
  newest first); `docs/user_stories_changelog.md` logs every change to it.
- When a feature or behaviour change is requested: add or update the story
  first, log it in the changelog, then implement. Keep `README.md` usage and
  `help_for` in step.

## Local dev
- run: `./cmm help`, `./cmm list --all`
- syntax check: `bash -n cmm`
- try `update`/`restart`/`keepalive` on a throwaway session in a scratch
  directory (`tmux new-session -d -s claude-selftest -c /tmp/x claude`), never
  on a session you are working in
- install: `./cmm install` (to `~/.local/bin`) or `./cmm install --system`

## Notes for Claude
- Prefer small, focused commits with descriptive messages.
- Ask before adding new dependencies.
- Keep `.claude/settings.local.json` out of git (it holds machine-specific paths).
