# Drain Lab Dashboard — build spec

A read-only dashboard Mason opens on his iPhone as a home-screen web app (PWA) to see, at a glance, whether his server and services are healthy. Rules and teamwork are in `CLAUDE.md`.

## Environment (verify in Phase 1, don't assume)

- Mini PC "drainlab", Rocky Linux (RHEL-based), SELinux probably enforcing, Docker installed.
- Pi-hole runs in a Docker container. Its setup is not fully finished — it may be unreachable; handle that gracefully.
- Tailscale is up on `tailscale0`. firewalld has a dedicated `tailscale` zone that already allows ssh, dns, http, **https** and 8824/tcp. Do not change the firewall.
- amux runs on this machine (port 8824). Don't touch it.

## Stack (decided — follow it)

- Backend: Python 3.12, FastAPI, uvicorn, psutil, httpx.
- Frontend: plain HTML/CSS/JavaScript in a static folder. No frameworks, no build step, no external CDNs or fonts.
- Docker Compose, `restart: unless-stopped`, a healthcheck, pinned image versions.
- Docker access through `tecnativa/docker-socket-proxy` with ONLY container read access (`CONTAINERS=1`, everything else off), on a private Compose network with no published ports. (Inside a container, 127.0.0.1 is the container itself.) If the dashboard needs host networking for accurate host stats, keep the proxy unreachable from outside and log the choice. The dashboard never mounts `docker.sock` directly.
- **Read-only v1:** no restart/stop buttons or control actions of any kind.
- Listens on `127.0.0.1:8088` only. Exposed to the tailnet over HTTPS with `tailscale serve` (persistent/background mode, syntax for the installed version). No public ports, no login screen in v1.
- Secrets in `.env` (git-ignored); commit `.env.example`.
- Name "Drain Lab". Dark theme, mobile-first (~390px wide), big readable cards, iPhone safe areas (`viewport-fit=cover`, `env(safe-area-inset-*)`).

## What it shows

1. Status banner: green "All good", yellow, or red — with the reason (e.g. "Pi-hole container stopped").
2. Server: hostname, uptime, CPU %, load average, RAM used/total, disk used/total for `/`, CPU temperature if available (skip quietly if not), sparklines of CPU and RAM for the last 60 minutes (in memory, no database).
3. Docker: every container — name, image, state, health, uptime. Anything not running listed first, in red.
4. Pi-hole: blocking enabled?, queries today, blocked today, % blocked. Detect v5 vs v6 and use the right API. Unreachable → say so clearly, don't crash.
5. Tailscale: is `tailscale0` up, and this server's Tailscale IP.
6. "Last updated" time; auto-refresh every 10 seconds.

Also: `GET /api/status` (all of the above as JSON, for a future widget/alerts) and `GET /api/health` (for the healthcheck).

## PWA

`manifest.json` (standalone, theme/background colours, name "Drain Lab"), apple-touch-icon 180×180 plus 192 and 512 icons (generate a simple clean icon), apple-mobile-web-app meta tags, and a service worker that caches the app shell while always fetching live data fresh.

## Gotchas

- SELinux: never relabel system paths with `:z`/`:Z`. If the socket proxy needs it, `security_opt: label=disable` on that container only.
- Host stats must be the HOST's numbers, not the container's. The reviewer will compare against `uptime`, `free -h` and `df -h /`.
- If `docker` needs sudo for Mason's user, don't work around it — stop and ask.

## Phases (builder commits after each, then requests review)

1. **Recon (read-only):** OS version, SELinux mode, Docker/Compose versions, containers, Pi-hole version and port, Tailscale version/status, MagicDNS/HTTPS availability. Write findings and decisions to `DECISIONS.md`.
2. **Backend + API**, with pytest tests for the data-parsing code.
3. **Frontend + PWA.**
4. **Docker Compose deploy.** Every card shows correct data; `/api/status` returns valid JSON.
5. **HTTPS via `tailscale serve`.** Confirm `https://<machine>.<tailnet>.ts.net` works.
6. **Docs + final report.**

## Docs

- `README.md` — what it is; start/stop/update; where logs are.
- `LEARNING.md` — for a beginner: what each piece is and why (FastAPI, Compose, the socket proxy, read-only, tailscale serve/HTTPS, PWA), plus a cheat sheet of the 10 commands Mason will actually use.
- `DECISIONS.md` — every decision and why.
- Comment the code so a beginner can follow it.

## Final report (builder posts it when done)

The URL; iPhone steps (Tailscale on → Safari → Share → Add to Home Screen); anything Mason must do himself; what was verified and what didn't work; three v2 ideas (alerts when something goes down, an iPhone widget reading `/api/status`, safe restart buttons).
