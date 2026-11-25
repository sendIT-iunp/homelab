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

**Step 2 — Installing Claude Code** ⏳ in progress
```bash
curl -fsSL https://claude.ai/install.sh | bash
source ~/.bashrc
claude --version
```
- `curl -fsSL` downloads quietly, follows redirects and fails cleanly on errors. `| bash` runs the downloaded script.
- `source ~/.bashrc` reloads shell settings so the new `claude` command can be found.
