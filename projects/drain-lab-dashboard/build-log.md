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
---

## 2026-11-26 — Agent team launched

**Plugins installed** (user scope, so every Claude Code session on the server gets them, including amux workers):
```bash
claude plugin marketplace add anthropics/claude-plugins-official
claude plugin install security-guidance@claude-plugins-official
claude plugin install frontend-design@claude-plugins-official
claude plugin install context7@claude-plugins-official
```
Small hiccup: one pasted line arrived as `^[[200~claude plugin install …~` and failed with "command not found". Those extra characters are *bracketed paste* markers, which the terminal adds around pasted text; they sometimes leak through. Re-running the line fixed it.

**Project folder:** `~/homelab-dashboard`, with [CLAUDE.md](CLAUDE.md) (team rules) and [BUILD.md](BUILD.md) (spec) downloaded from this repo and committed as the first commit.

**Workers registered:**

| Worker | Model | Permissions | Why |
|---|---|---|---|
| `builder` | sonnet | `acceptEdits`: edits files freely, asks before every command | Can build, but can't run anything without me |
| `reviewer` | sonnet | `default`, with Edit/Write disabled | Can only read, run tests (with approval) and report |

Both are pre-approved for only `amux board` and `amux post`, so they can hand work to each other without me relaying every message. amux can also send email, place calls and post to social media, which is why I didn't pre-approve all `amux` commands.

Started both detached, sent each its role, and the builder began Phase 1 (read-only recon).

---

## 2026-11-26 (evening) — Phases 1–4 built, and a security fix

The builder got through recon, backend, frontend and the Docker Compose deploy. 29 tests pass.

**Security finding, caught after deploy and fixed:**
- **Problem:** the docker-socket-proxy setting from the spec (`CONTAINERS=1`) allows more than listing containers. It also allows reading any container's environment variables and downloading files out of any container. The proxy was listening on a localhost port, so any program on the server, running as any user, could have done the same. Once the Pi-hole password goes into `.env`, it would have been readable that way. Writes were already blocked, so nothing could have been changed.
- **Fix:** the proxy no longer listens on a network port at all. It now uses a private socket file inside a Docker volume that only the proxy and dashboard containers can reach. On the host, that folder is root-only.
- **Verified:** the port is gone, the Docker card still lists all containers, and stop/restart/delete/exec requests are still refused. All tests pass.
- **Lesson:** "read-only" in a proxy setting doesn't mean "harmless". Reading a container's environment can mean reading its secrets. Check what an "allow" flag *actually* exposes, not what its name suggests.

**Workflow problems to fix before Phase 5:**
- The reviewer never got review requests. amux messages only reach workers that share a *group*, and builder and reviewer weren't in one. So phases 1–4 were built without the second-agent review the setup was designed for.
- The session ended up in **auto mode**, which lets Claude approve its own commands. That goes against the rule for this server, so it's being switched back to manual approvals.

**Next:** put both workers in a group and have the reviewer review phases 1–4; add a Pi-hole *app password* to `.env`; enable HTTPS certificates in Tailscale and run `tailscale serve` for the dashboard.
