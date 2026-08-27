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
cmm new [name]         # start a new claude session in $PWD and attach
cmm list               # list claude sessions under the current directory
cmm list --all         # list every claude session on the host
cmm resume [name|#]    # attach to an existing session (interactive if no arg)
cmm kill <name|#>      # kill a session; --all kills all under $PWD
cmm help [command]     # full help for any subcommand
```

Sessions are plain tmux sessions named `claude-<name>`, so you can also
manage them with `tmux` directly if you want.

## Requirements

- `bash`, `tmux`, and `claude` on `PATH`.
