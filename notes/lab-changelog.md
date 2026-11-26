# Lab Changelog

Changes to the server that aren't part of a specific project.

## 2026-11-26 — Installing amux

[amux](https://github.com/mixpeek/amux) is an open-source "agent multiplexer": it runs several Claude Code sessions inside tmux and gives you a web dashboard and iPhone app to watch and control them.

**First attempt failed:**
```bash
curl -fsSL https://amux.io/install.sh | bash
# ✗ cargo is required and this shell is non-interactive — install rust (rustup), then re-run ./install.sh
```
The installer cloned the repo to `~/Dev/amux`, then stopped because amux is written in **Rust**, and the Rust toolchain (`cargo`) wasn't installed.

**Second mistake:** I ran the rustup command without `| sh` at the end:
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs
```
That only *downloads and prints* the install script; nothing runs until it's piped into a shell. (Printing a script first is actually a good habit for reading what it does before running it.)

**Fix:**
```bash
sudo dnf install -y gcc                                          # C compiler Rust needs for linking
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh   # this time piped into sh, so it runs
source "$HOME/.cargo/env"                                        # load Rust into the current session
cd ~/Dev/amux && ./install.sh                                    # re-run the amux installer
```
Rust compiled amux's server and CLI in about 42 seconds and installed them to `~/.local/bin`. The installer then failed near the end:
```
./install.sh: line 397: shasum: command not found
```
`shasum` (a checksum tool) comes with macOS, but on Rocky Linux it lives in the `perl-Digest-SHA` package. Fix: install that package and re-run the installer.

Re-running `./install.sh` completed. It installed:
- the amux CLI and server in `~/.local/bin`, with data and an auth token in `~/.amux`
- hooks wired into Claude Code's global settings (`~/.claude/settings.json`), so every Claude Code session on this server now reports to amux and follows its boundary rules
- systemd **user services**: `amux-server`, `amux-builder.timer`, `amux-worker-start`, plus optional virtual-desktop (Xvfb/VNC) and browser-automation (Playwright) services

**Which services to turn on, and which not (yet):**

| Service | Decision | Why |
|---|---|---|
| Linger (`loginctl enable-linger`) | On | Without it, user services stop when I log out and don't start after a reboot |
| `amux-server` | On | The dashboard and API |
| `amux-builder.timer` | Off for now | Rebuilds amux on a schedule; I'd rather update deliberately than have code change under running agents |
| `amux-worker-start` | Off for now | Auto-starts every registered agent at boot; not wanted until the workflow is tested |
| Xvfb/VNC, Playwright lanes | Off | Not needed yet; a VNC desktop is extra attack surface |

**Lesson:** installers written mainly for macOS often assume Mac tools exist. "command not found" inside a script usually means one missing package, not a broken install.

**Security notes for this server:**
- amux's quick-start examples use `--yolo`, which skips Claude Code's permission prompts. I'm not using that on this server because it runs my real Pi-hole.
- amux runs a web server (port 8824). It should only be reachable locally or over Tailscale, never through a public port or a firewall opening.

## 2026-11-25 — Firmware notice

The server showed a notice about a UEFI Secure Boot revocation update (CVE-2026-8863), which blocks old, vulnerable bootloaders. Still to look at: whether to apply it with `fwupdmgr`.
