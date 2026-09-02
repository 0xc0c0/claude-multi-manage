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
cmm keepalive --daemon     # keep Remote Control reconnected in idle sessions
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

## Keeping Remote Control alive

Claude Code retries a dropped Remote Control connection for about 30 minutes
and then gives up, showing `/rc failed` in the footer until someone runs
`/remote-control` in that session. Detached sessions that sit idle for hours
therefore go stale after any network blip. `cmm keepalive` checks every
`claude-*` session and types `/remote-control` into the ones that have failed
while idle at an empty prompt.

```bash
cmm keepalive --once --dry-run   # see what it would do
cmm keepalive --daemon           # run every 5 minutes in tmux session cmm-keepalive
cmm keepalive --status           # is it running? recent log
cmm keepalive --stop
```

`cmm list` shows the Remote Control state of each session (`rc:on`,
`rc:failed`, `rc:off`).

## Requirements

- `bash`, `tmux`, and `claude` on `PATH`.

## Docs

- [docs/user_stories.md](docs/user_stories.md): master record of every user
  story requested for this tool.
- [docs/user_stories_changelog.md](docs/user_stories_changelog.md): log of
  changes to those stories.
