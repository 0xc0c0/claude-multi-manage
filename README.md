# claude-multi-manage

Command-line utility (`cmm`) for quickly starting and managing multiple
`claude` sessions on the same host via tmux.

## Install

```bash
git clone <repo-url>
cd claude-multi-manage
./cmm install            # copies cmm to ~/.local/bin
# or
./cmm install --system   # copies to /usr/local/bin (may need sudo)
```

Make sure the destination is on your `PATH`.

## Usage

```bash
cmm new [name] [-- args]   # start a new claude session in $PWD and attach
cmm list                   # list claude sessions under the current directory
cmm list --all             # list every claude session on the host
cmm resume [name|#]        # attach to an existing session (interactive if no arg)
cmm update [name|#]        # restart a session's claude on the latest version,
                           # keeping the conversation (--all, --pending)
cmm restart [name|#]       # same as update, without running 'claude update'
cmm maintain --install     # background service: apply waiting updates and
                           # reconnect Remote Control, in idle sessions only
cmm pause                  # before a reboot: stop every session under $PWD
                           # cleanly, remembering how to bring it back
cmm reload                 # after the reboot: bring them all back
cmm kill <name|#>          # kill a session; --all kills all under $PWD
cmm help [command]         # full help for any subcommand
```

Sessions are plain tmux sessions named `claude-<name>`, so you can also
manage them with `tmux` directly if you want.

## Updating Claude without losing a session

Claude Code installs updates in the background but keeps running the old
version until it is restarted; the footer then shows
`Update installed · Restart to apply`. `cmm update` restarts Claude in place:
it runs `claude update`, asks the Claude in the session to exit, reads the
`Resume this session with: claude --resume <id>` hint Claude prints on the way
out, and starts `claude --resume <id>` again in the same tmux pane. The
session name, attached clients and Remote Control all carry over.

```bash
cmm update                 # from a second window of a cmm session: that session
cmm update feature-x       # one session
cmm update --pending       # every session under $PWD waiting for a restart
cd ~ && cmm update --all   # every session on the host
```

Sessions that are busy or have unsent text in the prompt are skipped unless
you pass `--force`. `cmm list` shows `update-ready` next to sessions that need
it. Run it from outside the Claude you are restarting.

### The model and effort level come back too

Claude does not carry either across a resume: `/model` saves your choice as the
default for *new* sessions and `/effort` is explicitly session-only, so a plain
`claude --resume` starts on your global defaults. Before restarting, `cmm` reads
the session's own transcript to see which model it was replying with and which
`/effort` you last set, and passes them to the new Claude
(`--model opus --effort max`, or `--settings '{"ultracode":true}'` for
ultracode). The model is restored as a family alias (`opus`, `fable`,
`sonnet`, `haiku`) so an old session cannot come back on a version this Claude
Code no longer knows, and anything you pass after `--` wins.

```bash
cmm list --model           # what each session would come back as
```

`cmm` also turns off Claude's "resume from summary" prompt for the restart, so
the whole conversation comes back rather than a summary of it.

## Background maintenance

`cmm maintain` looks after sessions that sit idle for hours: it restarts the
ones whose footer says `Update installed · Restart to apply` (bringing back
their conversation, model and effort), and types `/remote-control` into the
ones showing `/rc failed` — Claude Code retries a dropped Remote Control
connection for about 30 minutes and then gives up until someone runs that
command.

It is deliberately timid. A session is only touched when Claude is idle at an
empty prompt (not busy, nothing typed, no dialog open), its screen has not
changed for a few seconds, no attached client has typed in it for `--idle`
minutes (default 15), and `cmm` has not restarted it within `--cooldown`
minutes (default 60). At most `--max` sessions (default 2) are restarted per
pass, and a failure stops the rest. Everything skipped is simply looked at
again next time, and only one pass runs at a time on a host.

```bash
cmm maintain --once --dry-run    # what a pass would do right now
cmm maintain --install           # systemd user timer (or cron), every 15 min
cmm maintain --install --cron --interval 1800 --ensure   # pick the details
cmm maintain --status            # what is installed, when it runs, recent log
cmm maintain --uninstall
cmm maintain --daemon            # no-setup alternative: a loop in tmux
```

Unattended passes log to `~/.cache/cmm/maintain.log`. With systemd, run
`loginctl enable-linger $USER` if you want it to keep running while you are
logged out. `cmm keepalive` still works as an alias for this command.

`cmm list` shows what each session is doing (`idle`, `busy`, `typing`,
`dialog`), whether it is `update-ready`, and its Remote Control state
(`rc:on`, `rc:failed`, `rc:off`).

## Rebooting the host

tmux sessions do not survive a reboot, so before a kernel or security update
run `cmm pause` from a directory above all your sessions (`~`, say). It asks
each Claude to exit cleanly, reads the resume id from the exit hint, and
writes what it needs to bring the session back (name, directory, resume id,
worktree, Remote Control state, model, effort level, and any unsent text in
the prompt) to a manifest under `~/.local/state/cmm/paused`. After the reboot,
`cmm reload` recreates every session from that manifest with its original name
and directory, resuming the same conversation with the same model, effort and
Remote Control, and types any unsent text back in without submitting it.

```bash
cd ~ && cmm pause          # shows what it will do, then asks
sudo reboot
cd ~ && cmm reload         # everything comes back; prints 'cmm list'
```

A session that is still working, or waiting for an answer on screen, is
waited for (`--wait`, default 10 minutes) and paused as soon as it is idle.
It is never interrupted: anything still busy when the wait runs out is left
running and listed, and `cmm pause` exits with status 1 so you know it is not
yet safe to reboot (`--force` interrupts instead). `--all` takes every session
on the host rather than those under `$PWD`, `--yes` skips the confirmation
and `--dry-run` only reports.

The tmux sessions are left in place with the exit hint on screen (`cmm list`
shows them as `paused`), so if there is no reboot after all, `cmm reload`
simply restarts Claude in them. Sessions that come back are removed from the
manifest; any that could not be started stay in it, and `cmm reload` retries
only those. `cmm reload --list` shows what is paused and `cmm list` mentions
it, so a forgotten reload is noticed. The maintenance service needs no
attention: it ignores paused sessions and picks the reloaded ones up again.

If a recorded directory is missing, `cmm reload` uses the one in the
session's transcript. An entry that cannot be brought back at all (its
directory is gone and the transcript does not say where it ran, or something
else is already running under that name) is listed with the
`cmm reload --forget NAME` that drops it.

## Requirements

- `bash`, `tmux`, and `claude` on `PATH`.

## Docs

- [docs/user_stories.md](docs/user_stories.md): master record of every user
  story requested for this tool.
- [docs/user_stories_changelog.md](docs/user_stories_changelog.md): log of
  changes to those stories.
