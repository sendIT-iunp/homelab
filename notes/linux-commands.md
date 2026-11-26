# Linux Command Cheat Sheet

Commands I've actually used in my lab, with what they do. Grows as I go.

## Packages (Rocky Linux)

| Command | What it does |
|---|---|
| `sudo dnf install -y <pkg>` | Install a package. `sudo` runs as admin, `-y` auto-confirms |

| `dnf provides <command>` | Find which package contains a missing command |

## Shell basics

| Command | What it does |
|---|---|
| `source ~/.bashrc` | Reload shell settings without logging out |
| `echo 'line' >> file` | Append a line to the end of a file (`>` would overwrite it) |
| `ls <folder>` | List what's in a folder. `~` means my home folder |
| `cmd1 \| cmd2` | "Pipe": send the output of one command into another |
| `curl -fsSL <url>` | Download a URL quietly, follow redirects, fail cleanly on errors |
| `export PATH="$HOME/.local/bin:$PATH"` | Add a folder to the places Linux looks for commands |

## Services (systemd)

| Command | What it does |
|---|---|
| `systemctl --user enable --now <svc>` | Start a user service now and on every boot (`enable` = at boot, `--now` = also right now) |
| `systemctl --user status <svc>` | Is it running? Shows recent log lines too |
| `journalctl --user -u <svc> -f` | Follow a service's logs live (`Ctrl+C` to stop) |
| `sudo loginctl enable-linger <user>` | Let a user's services keep running after logout and start at boot |
| `ss -tlnp` | List programs listening on network ports, and on which address |

## Firewall (firewalld)

| Command | What it does |
|---|---|
| `sudo firewall-cmd --get-active-zones` | Which zone each network interface is in |
| `sudo firewall-cmd --list-all` | What the default zone allows (add `--zone=<name>` for another zone) |
| `sudo firewall-cmd --permanent --zone=<z> --add-service=<svc>` | Allow a named service (ssh, http, dns…) in a zone |
| `sudo firewall-cmd --permanent --zone=<z> --add-port=<port>/tcp` | Allow one port in a zone |
| `sudo firewall-cmd --reload` | Apply the saved (`--permanent`) rules |
| `cat <file>` | Print a file's contents |

## tmux (sessions that survive disconnecting)

| Command | What it does |
|---|---|
| `tmux new -s <name>` | Start a named session |
| `Ctrl+B`, then `D` | Detach, leaving the session running |
| `tmux attach -t <name>` | Reattach to a session |
| `tmux ls` | List running sessions |
