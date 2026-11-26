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

**Fix:** in progress, see below.

**Security notes for this server:**
- amux's quick-start examples use `--yolo`, which skips Claude Code's permission prompts. I'm not using that on this server because it runs my real Pi-hole.
- amux runs a web server (port 8824). It should only be reachable locally or over Tailscale, never through a public port or a firewall opening.

## 2026-11-25 — Firmware notice

The server showed a notice about a UEFI Secure Boot revocation update (CVE-2026-8863), which blocks old, vulnerable bootloaders. Still to look at: whether to apply it with `fwupdmgr`.
