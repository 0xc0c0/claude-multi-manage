# User Stories Changelog

Every change to [user_stories.md](user_stories.md), newest first. One dated
section per change; each bullet names the story IDs it touches and what
changed (added, status, criteria, delivery).

## 2026-10-03

- US-013: corrected after a stale manifest entry was reported on 2026-09-29.
  A session whose Claude had already exited was recorded without a directory
  (a dead tmux pane reports none), so `cmm reload` could never recreate it
  and only said that its directory did not exist. Pause now falls back to
  the tmux session's start directory and then the transcript's working
  directory, reload recovers a missing directory from the transcript, and
  entries that cannot be brought back are listed with the `--forget` command
  that drops them. **Done** in ec796a5.

## 2026-09-13

- Added US-013 (pause every session before a reboot and bring them back
  afterwards) from the 2026-09-13 request for a graceful pause of all
  sessions ahead of a system reboot and a graceful reload from their
  contexts, run from a master location such as `~`; **Done** in 999fd7e.
  Decisions recorded with the story: busy sessions are waited for, never
  interrupted (only `--force` interrupts); unsent prompt text is saved and
  typed back on reload; reload is manual, nothing runs at boot.

## 2026-09-07

- Added US-011 (unattended maintenance of idle sessions) from the 2026-09-07
  request for a daemon or cron job that only touches idle sessions needing an
  update or showing `/rc failed`; **Done** in 41378f5.
- Added US-012 (keep the model and effort level a session was using) from the
  2026-09-07 report that models and effort levels revert to defaults after an
  update; **Done** in 41378f5.
- US-009: `cmm keepalive` became `cmm maintain` (old name kept as an alias) and
  its default interval moved from 300 s to 900 s when US-011 folded updates
  into the same pass.
- US-008: restarts now suppress Claude's "resume from summary" prompt so a
  resumed session always comes back with its full context, and restore the
  session's model and effort (US-012).

## 2026-09-02

- Created the record from a reverse-chronological review of the git history.
  The history held one commit, 1777709 (2026-08-27, "cmm: bash CLI to start
  and manage multiple claude sessions via tmux"), from which US-001 to US-007
  were reconstructed; all **Done**.
- Added US-008 (update Claude in place without losing the session) from the
  2026-09-02 request; **Done** in ee97c63.
- Added US-009 (keep Remote Control alive in dormant sessions) from the
  2026-09-02 request; **Done** in ee97c63.
- Added US-010 (keep a master record of user stories) from the 2026-09-02
  request; **Done** with the commit that added this file.
- US-001: noted the `-- <claude args>` pass-through added with US-008.
- US-002: noted the STATUS column added with US-008 and US-009.
