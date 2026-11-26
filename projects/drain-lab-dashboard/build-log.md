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

---

## 2026-11-26 — Switched to a multi-agent build

Decided to build the dashboard with an amux agent team instead of one agent. I weighed three options:

| Option | Why not / why |
|---|---|
| Keep one agent, add amux later | Simplest, but doesn't teach multi-agent workflows |
| **Agent team builds the dashboard** ✅ | Learn coordination on a real project with a finish line, and still ship the dashboard |
| Drop the dashboard, just learn amux | No clear finish line, and loses the most resume-friendly project |

Guardrails: start with two agents, keep permission prompts on, fence agents into the project folder, watch usage limits. amux install notes are in the [lab changelog](../../notes/lab-changelog.md).

**Future project:** a VM sandbox on the mini PC (KVM + Cockpit) where agents can run unattended without any risk to Pi-hole.