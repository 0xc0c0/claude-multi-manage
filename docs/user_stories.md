# User Stories

Master record of every user story requested for `cmm` (claude-multi-manage),
newest first. Each entry keeps the original ask (lightly paraphrased), the
story, its acceptance criteria, status, and where it was delivered. Every
change to this file is logged in [user_stories_changelog.md](user_stories_changelog.md).

## Conventions

- IDs are `US-<n>`, assigned in the order a story was first asked for, and
  never reused. Stories are listed newest first.
- Status is one of **Proposed**, **In progress**, **Done**, **Dropped**.
- "Asked" is the date and place the request was made (a commit, a
  conversation, an issue). "Delivered in" lists the commits that shipped it.
- When a new feature or behaviour change is requested: add or update the story
  here first, log the change in the changelog, then implement it.

## Index

| ID | Story | Asked | Status |
|----|-------|-------|--------|
| [US-012](#us-012--keep-the-model-and-effort-level-a-session-was-using) | Keep the model and effort level a session was using | 2026-09-07 | Done |
| [US-011](#us-011--unattended-maintenance-of-idle-sessions) | Unattended maintenance of idle sessions | 2026-09-07 | Done |
| [US-010](#us-010--keep-a-master-record-of-user-stories) | Keep a master record of user stories | 2026-09-02 | Done |
| [US-009](#us-009--keep-remote-control-alive-in-dormant-sessions) | Keep Remote Control alive in dormant sessions | 2026-09-02 | Done |
| [US-008](#us-008--update-claude-in-place-without-losing-the-session) | Update Claude in place without losing the session | 2026-09-02 | Done |
| [US-007](#us-007--stay-a-single-dependency-free-bash-script) | Stay a single, dependency-free bash script | 2026-08-27 | Done |
| [US-006](#us-006--built-in-help-and-version) | Built-in help and version | 2026-08-27 | Done |
| [US-005](#us-005--install-the-tool-onto-path) | Install the tool onto PATH | 2026-08-27 | Done |
| [US-004](#us-004--kill-sessions) | Kill sessions | 2026-08-27 | Done |
| [US-003](#us-003--resume-an-existing-session) | Resume an existing session | 2026-08-27 | Done |
| [US-002](#us-002--list-sessions) | List sessions | 2026-08-27 | Done |
| [US-001](#us-001--start-a-new-claude-session-in-a-directory) | Start a new Claude session in a directory | 2026-08-27 | Done |

## Stories

### US-012 — Keep the model and effort level a session was using

- **Asked:** 2026-09-07, in conversation: "one thing that has been happening
  with updates — the models and effort levels get reverted back to defaults.
  Ensure whatever the last model and effort level the session had in use is
  retained after an update."
- **Story:** As a user who picks a model and an effort level per session, I
  want a restarted session to come back on the same model and effort it was
  using, so that an update does not silently drop me back to my global
  defaults.
- **Background:** `/model` writes the chosen model to `settings.json` as the
  default for *new* sessions, and `/effort` is explicitly session-only
  ("this session only"), so a plain `claude --resume <id>` starts on the
  global default model and the built-in effort. Claude Code records what a
  session actually ran in its transcript
  (`~/.claude/projects/<dir>/<session-id>.jsonl`), and accepts `--model`,
  `--effort` and `--settings '{"ultracode":true}'` on the command line.
- **Acceptance criteria:**
  - Before restarting a session, cmm works out the model and effort it was
    last using and passes them to the new Claude, so the restarted session
    reports the same model and effort as before.
  - The model is taken from the session's own transcript (the last `/model`
    the user ran, otherwise the model of the last assistant message) and is
    restored as a family alias (`fable`, `opus`, `sonnet`, `haiku`) so it
    keeps resolving to a model this Claude Code version knows.
  - The effort level is taken from the last `/effort` in the transcript,
    falling back to what cmm itself launched the session with; `ultracode` is
    restored as `--settings '{"ultracode":true}'`, and `auto` restores
    nothing.
  - Explicit arguments after `--` still win over the restored ones, and if the
    restarted Claude dies immediately, cmm retries once without the restored
    flags rather than leaving the session down.
  - `cmm list --model` shows the model and effort cmm would restore for each
    session.
- **Status:** Done
- **Delivered in:** 41378f5

### US-011 — Unattended maintenance of idle sessions

- **Asked:** 2026-09-07, in conversation: "I want to be able install a deamon
  or cron job that safely maintains up-to-date claude sessions under this
  manager, without interrupting active sessions (i.e. only touches idle
  sessions and require an update or have /rc in failure mode). This should be
  maximally cautious."
- **Story:** As a user with several long-lived sessions, I want a background
  service that keeps them on the current Claude Code version and keeps Remote
  Control connected, without ever interrupting a session I am using, so that I
  can leave sessions running for days and find them healthy.
- **Acceptance criteria:**
  - `cmm maintain` runs a maintenance pass over every `claude-*` session on the
    host: sessions whose footer says "Update installed · Restart to apply" are
    restarted in place (conversation, model and effort preserved), and
    sessions showing `/rc failed` get `/remote-control` typed for them.
  - A session is only touched when every check passes: Claude is idle at an
    empty prompt (no dialog, no unsent text, not working), its screen has not
    changed for a few seconds, no attached client has had keyboard activity
    for `--idle` minutes (default 15), and cmm has not restarted it within
    `--cooldown` minutes (default 60). Everything else is left for the next
    pass.
  - A pass restarts at most `--max` sessions (default 2) and stops restarting
    after the first failure; `--dry-run` reports what a pass would do without
    touching anything.
  - Only one pass runs at a time on a host (a lock), so a service, a cron job
    and a manual run cannot collide.
  - `cmm maintain --install` installs the service — a systemd user timer where
    available, otherwise a cron entry — so it survives logout and reboot;
    `--uninstall` removes it and `--status` shows what is installed, when it
    next runs, and the tail of its log.
  - `cmm maintain --daemon` keeps the zero-setup option of running the loop in
    a detached tmux session.
- **Status:** Done
- **Delivered in:** 41378f5

### US-010 — Keep a master record of user stories

- **Asked:** 2026-09-02, in conversation: "As with all my projects, I'm starting
  a master record of all user stories that are asked for in regard to the code
  I'm writing with Claude. Please do a reverse chronological review of the git
  history of this project and build a user_stories.md file in docs, along with
  a change log file that sits alongside it for tracking changes to user stories
  going forward."
- **Story:** As the project owner, I want every user story requested for this
  codebase recorded in one place, reconstructed from the git history and kept
  current from now on, so that I can see what was asked for, when, and how it
  evolved.
- **Acceptance criteria:**
  - `docs/user_stories.md` lists every story found in the git history and every
    later request, newest first, with an ID, status and delivering commits.
  - `docs/user_stories_changelog.md` sits beside it and records every addition
    or change to a story, dated.
  - The project instructions (`CLAUDE.md`) tell contributors to keep both files
    current whenever a story is requested, changed or delivered.
- **Status:** Done
- **Delivered in:** the commit that added `docs/` (`git log --follow docs/user_stories.md`)

### US-009 — Keep Remote Control alive in dormant sessions

- **Asked:** 2026-09-02, in conversation: "Claude now has remote-control
  features and that works great with sessions spawned from `cmm`; however,
  sometimes the `cmm` tmux sessions sit dormant after a number of hours, and the
  claude remote-control feature disconnects or goes stale. Can you fix this?"
- **Story:** As a user who drives cmm sessions from a phone or browser through
  Remote Control, I want sessions that have been idle for hours to stay
  reachable, so that I do not have to reattach locally and run
  `/remote-control` by hand.
- **Background:** Claude Code retries a dropped Remote Control connection for
  about 30 minutes, then gives up and shows `/rc failed` in the footer until
  somebody runs `/remote-control` in that session. A network blip or host sleep
  longer than that leaves a detached session stale indefinitely.
- **Acceptance criteria:**
  - `cmm keepalive` checks every `claude-*` session on the host and types
    `/remote-control` into sessions that show `/rc failed` while idle at an
    empty prompt.
  - It never types into a session that is busy, has unsent text in the prompt,
    is showing a dialog, or has had keyboard activity in the last two minutes;
    those are retried on the next check.
  - `--daemon` runs it detached in a tmux session named `cmm-keepalive`;
    `--status` and `--stop` manage it; `--once` and `--dry-run` support manual
    checks; `--interval` sets the cadence (default 300 s).
  - `--restart` optionally restarts a session in place when Claude refuses to
    reconnect and asks for a restart; `--ensure` optionally turns Remote
    Control on in sessions that have it off.
  - `cmm list` shows each session's Remote Control state (`rc:on`,
    `rc:failed`, `rc:off`, ...).
- **Status:** Done
- **Delivered in:** ee97c63

### US-008 — Update Claude in place without losing the session

- **Asked:** 2026-09-02, in conversation: "Claude routinely updates itself, so
  ideally the cmm tool has an `update` subcommand that exits a session, grabs
  the resume-id that claude drops to the terminal upon exit, and resumes claude
  (which carries the updated claude back into the context of the existing
  session). Right now, whenever I want to update claude, I have to fully exit a
  session and then run `cmm new` in the folder again, which brings none of the
  prior session context into the pane of reference."
- **Story:** As a user with several long-running sessions, I want
  `cmm update` to restart the Claude inside a session on the newly installed
  version while keeping the conversation and the tmux session, so that updating
  no longer means exiting and starting an empty context with `cmm new`.
- **Acceptance criteria:**
  - `cmm update [name|#]` runs `claude update`, then for each target session
    asks Claude to exit, reads the `Resume this session with: claude --resume
    <id>` hint Claude prints on exit, and starts `claude --resume <id>` in the
    same tmux pane. The session name and any attached clients are preserved.
  - Remote Control is turned back on if the session had it.
  - `--all` targets every session under `$PWD`; `--pending` only those whose
    footer shows "Update installed · Restart to apply".
  - Sessions that are busy, have unsent text, or show a dialog are skipped
    unless `--force` is given; the command refuses to restart the session it is
    itself running in.
  - `cmm restart` does the same without running the updater; arguments after
    `--` are passed to the restarted Claude.
  - `cmm list` flags sessions waiting for a restart with `update-ready`.
- **Status:** Done
- **Delivered in:** ee97c63

### US-007 — Stay a single, dependency-free bash script

- **Asked:** 2026-08-27, recorded as project constraints in `CLAUDE.md` and
  `README.md` in commit 1777709.
- **Story:** As the maintainer, I want cmm to remain one bash script that needs
  only `bash`, `tmux` and `claude` on `PATH`, so that it can be copied onto any
  host without setup.
- **Acceptance criteria:** a single executable `cmm` (`#!/usr/bin/env bash`,
  `set -euo pipefail`); subcommands are `cmd_<name>` functions with their help
  in `help_for`; no other runtime dependencies; no generated files; `bash -n
  cmm` passes.
- **Status:** Done
- **Delivered in:** 1777709

### US-006 — Built-in help and version

- **Asked:** 2026-08-27, commit 1777709 ("with good help/usage functionality").
- **Story:** As a user, I want `cmm help [command]`, `-h`/`--help` on every
  command, and `cmm version`, so that I can learn the tool without reading its
  source.
- **Acceptance criteria:** `cmm` with no arguments and `cmm help` print usage
  with a command list and examples; `cmm help <command>` and `cmm <command>
  --help` print per-command help; `cmm version` prints the version; unknown
  commands and options fail with a hint pointing at the help.
- **Status:** Done
- **Delivered in:** 1777709

### US-005 — Install the tool onto PATH

- **Asked:** 2026-08-27, commit 1777709.
- **Story:** As a user, I want `cmm install` to copy the script somewhere on my
  `PATH`, so that I can run `cmm` from any directory.
- **Acceptance criteria:** installs to `~/.local/bin` by default or
  `/usr/local/bin` with `--system`; refuses to overwrite an existing file
  without confirmation (`--force` skips the prompt); recognises when the source
  is already the installed file; warns when the destination is not on `PATH`.
- **Status:** Done
- **Delivered in:** 1777709

### US-004 — Kill sessions

- **Asked:** 2026-08-27, commit 1777709.
- **Story:** As a user, I want to kill one session by name or list number, or
  every session under the current directory, so that I can clean up finished
  work quickly.
- **Acceptance criteria:** `cmm kill <name|#>` accepts the list index, the full
  tmux name or the name without the `claude-` prefix; `cmm kill --all` kills
  every session at or under `$PWD` after a confirmation prompt (`--force`
  skips it).
- **Status:** Done
- **Delivered in:** 1777709

### US-003 — Resume an existing session

- **Asked:** 2026-08-27, commit 1777709.
- **Story:** As a user, I want to reattach to a running session by list number
  or name, or pick one interactively, so that I can get back to work in a few
  keystrokes.
- **Acceptance criteria:** `cmm resume [name|#]` attaches by list index, full
  name or suffix; with no argument it prints the list and prompts; when already
  inside tmux it switches the client instead of nesting.
- **Status:** Done
- **Delivered in:** 1777709

### US-002 — List sessions

- **Asked:** 2026-08-27, commit 1777709.
- **Story:** As a user, I want to see which Claude sessions exist for the
  directory I am in, and optionally across the host, so that I can find the
  right one.
- **Acceptance criteria:** `cmm list` prints index, name, attached state, age
  and working directory for sessions whose directory is at or under `$PWD`;
  `--all` shows every session on the host.
- **Status:** Done
- **Delivered in:** 1777709 (STATUS column added by US-008 / US-009)

### US-001 — Start a new Claude session in a directory

- **Asked:** 2026-08-27, commit 1777709 ("quickly starting and managing multiple
  claude sessions on the same host").
- **Story:** As a user, I want one command that starts Claude in the current
  directory inside its own tmux session and attaches me to it, so that I can
  run many independent sessions on one host and come back to them later.
- **Acceptance criteria:** `cmm new [name]` creates a detached tmux session
  named `claude-<name>` (default: the directory basename, with `.` and `:`
  replaced and a numeric suffix on collision) running `claude` in `$PWD`, then
  attaches; sessions are plain tmux sessions so `tmux` can manage them too.
- **Status:** Done
- **Delivered in:** 1777709 (`-- <claude args>` pass-through added by US-008)
