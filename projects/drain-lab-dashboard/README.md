# Drain Lab Dashboard

A read-only dashboard that shows at a glance whether my home lab server and its services are healthy. It runs on the server and installs on my iPhone as a home screen web app.

**Status:** 🟡 Setting up — see the [build log](build-log.md).

## Why I'm building it

- **Learn Linux and Docker** by building something real on my own server.
- **Portfolio project** for IT and data center job applications.
- **A tool I'll actually use** to check the server from my phone.

## What it will show

- A status banner: green, yellow or red, with the reason
- Server stats: uptime, CPU, load, RAM, disk, temperature, with 60-minute sparklines
- Every Docker container and its state
- Pi-hole stats: blocking on/off, queries today, blocked today, % blocked
- Tailscale status
- A JSON endpoint (`/api/status`) for a future iPhone widget and alerts

## Design decisions

| Area | Decision | Why |
|---|---|---|
| iPhone app | Home screen web app (PWA), not a native app | Native iPhone apps need a Mac and Xcode; a PWA still opens full-screen with its own icon |
| Backend | Python 3.12, FastAPI, psutil, httpx | Simple, well documented, good for small APIs |
| Frontend | Plain HTML, CSS, JavaScript | No build step to learn or maintain |
| Deployment | Docker Compose with healthchecks and pinned image versions | Restarts on its own and behaves the same every time |
| Docker access | Read-only docker-socket-proxy on a private network | Full access to the Docker socket is effectively root on the host; the proxy only allows reading container info |
| Scope (v1) | Read-only, no control buttons | Lower risk while I learn |
| Network | Local only, shared to my Tailscale network over HTTPS with `tailscale serve` | No public ports and no firewall changes |
| Secrets | `.env` file, git-ignored | Passwords never end up in a repo |

## How it's being built

I'm building it with **Claude Code** running on the server inside **tmux**, so the work keeps going when I disconnect. I approve sudo commands, passwords and anything destructive myself.

## Planned for v2

- Phone alerts when the server, Pi-hole or a container goes down
- An iPhone home screen widget (Scriptable) that reads `/api/status`
- Safe restart buttons for containers
