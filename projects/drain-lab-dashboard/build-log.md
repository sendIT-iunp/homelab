# Drain Lab Dashboard — Build Log

What I actually did, in order, including what went wrong and how I fixed it.

---

## 2026-11-25 — Planning and first setup steps

**Planning**
- Settled the design (see the [project README](README.md)): PWA instead of a native app, FastAPI backend, Docker Compose, read-only v1, reachable only over Tailscale.
- Wrote a project brief and a build prompt for Claude Code so the plan is written down before any code.
- Changed the plan so the docker-socket-proxy sits on a private Docker network with no published ports. Inside a container, `127.0.0.1` means the container itself, so binding the proxy there wouldn't have worked.

**Step 1 — Installed tmux and git** ✅
```bash
sudo dnf install -y tmux git
```
- `dnf` is Rocky Linux's package manager. `-y` auto-confirms.
- **tmux** keeps terminal sessions running on the server after I disconnect, which matters a lot from an iPhone because iOS drops SSH when Termius goes to the background.
- **git** tracks changes to the project so I can roll back.

**Step 2 — Installed Claude Code** ✅
```bash
curl -fsSL https://claude.ai/install.sh | bash
source ~/.bashrc
claude --version
```
- `curl -fsSL` downloads quietly, follows redirects and fails cleanly on errors. `| bash` runs the downloaded script.
- `source ~/.bashrc` reloads shell settings so the new `claude` command can be found.

**Troubleshooting:** `claude --version` first returned `bash: claude: command not found`. To tell "not installed" apart from "installed but not on my PATH", I checked the folder the installer uses:
```bash
ls ~/.local/bin
# ls: cannot access '/home/masonbettinger/.local/bin': No such file or directory
```
The folder didn't exist, so the installer had never actually run. I re-ran the curl command (pasting it instead of typing it on my phone) and it installed.

**Lesson:** "command not found" doesn't always mean a PATH problem. Check whether the program exists first, then whether the shell can find it.

**Side note for later:** the server showed a firmware notice about a UEFI Secure Boot revocation update (CVE-2026-8863). It's unrelated to this project; I'll look at it separately.
