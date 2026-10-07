# Project: claude-multi-manage

> command line utility for quickly starting and managing multiple claude sessions on the same host

## Stack
- bash only, very simple, with good help/usage functionality
- runtime deps: `bash`, `tmux`, `claude` on `PATH` — nothing else

## Conventions
- single script: `cmm` (executable, `#!/usr/bin/env bash`, `set -euo pipefail`)
- subcommands are `cmd_<name>` functions; per-command help lives in `help_for`
- sessions are plain tmux sessions named `claude-<name>`
- `update`/`restart`/`maintain` read a session's screen with `tmux capture-pane`
  and drive it with `tmux send-keys`; they rely on strings Claude Code prints
  (`Resume this session with:`, `/rc failed`, `Restart to apply`, `esc to interrupt`,
  and the exit dialogs `Background work is running` / `Exiting worktree session`),
  on its transcripts under `~/.claude/projects/<dir>/<session-id>.jsonl` (for the
  model and effort a session was using), and on `--model`, `--effort`,
  `--settings` and `CLAUDE_CODE_RESUME_THRESHOLD_MINUTES`
- what Claude Code itself says about a session comes from `claude agents --json`
  (status and session id by pid; `busy` includes background work), falling back
  to `~/.claude/sessions/<pid>.json` while `claude` cannot run; an update in
  progress shows as `claude --version` failing or a fresh `~/.claude/.update.lock`
  (npm installs leave a placeholder `claude` that exits 1), and nothing may be
  stopped or started then
- a session's prompt is the `❯` line directly under the input box's top border,
  near the bottom of the screen: earlier prompts stay on screen as history and
  dialogs are drawn below them, so anchor on the border, never on the last `❯`;
  the footer is what follows the box's bottom border, with the list of running
  agents under it, so never read it as "the last N lines"
- tmux `-F` output is sanitised without a UTF-8 locale (a tab separator becomes
  `_` under cron), so `_collect_sessions` separates fields with `|`
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
- try `update`/`restart`/`maintain` on a throwaway session in a scratch
  directory (`tmux new-session -d -s claude-selftest -c /tmp/x claude`), never
  on a session you are working in; to exercise a maintenance pass, source the
  script and override `_collect_sessions` / `_pane_update_ready` so the pass
  only sees the throwaway one
- also check a pass in a cron-like environment
  (`env -i HOME=$HOME PATH=... cmm maintain --once --dry-run`)
- install: `./cmm install` (to `~/.local/bin`) or `./cmm install --system`

## Notes for Claude
- Prefer small, focused commits with descriptive messages.
- Ask before adding new dependencies.
- Keep `.claude/settings.local.json` out of git (it holds machine-specific paths).
