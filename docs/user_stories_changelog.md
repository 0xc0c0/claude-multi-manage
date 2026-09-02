# User Stories Changelog

Every change to [user_stories.md](user_stories.md), newest first. One dated
section per change; each bullet names the story IDs it touches and what
changed (added, status, criteria, delivery).

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
