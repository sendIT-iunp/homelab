# Linux Command Cheat Sheet

Commands I've actually used in my lab, with what they do. Grows as I go.

## Packages (Rocky Linux)

| Command | What it does |
|---|---|
| `sudo dnf install -y <pkg>` | Install a package. `sudo` runs as admin, `-y` auto-confirms |

## Shell basics

| Command | What it does |
|---|---|
| `source ~/.bashrc` | Reload shell settings without logging out |
| `echo 'line' >> file` | Append a line to the end of a file (`>` would overwrite it) |
| `ls <folder>` | List what's in a folder. `~` means my home folder |
| `cmd1 \| cmd2` | "Pipe": send the output of one command into another |
| `curl -fsSL <url>` | Download a URL quietly, follow redirects, fail cleanly on errors |
| `export PATH="$HOME/.local/bin:$PATH"` | Add a folder to the places Linux looks for commands |

## tmux (sessions that survive disconnecting)

| Command | What it does |
|---|---|
| `tmux new -s <name>` | Start a named session |
| `Ctrl+B`, then `D` | Detach, leaving the session running |
| `tmux attach -t <name>` | Reattach to a session |
| `tmux ls` | List running sessions |
