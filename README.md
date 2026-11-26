# Drain Lab — Mason's Home Lab

My home lab and the projects I build on it. I'm a Computer Science student at San Jacinto College and I'm working toward IT help desk and data center roles. This repo documents what I set up, how, and what I learned along the way.

## The lab

| Piece | What it is |
|---|---|
| Server | Mini PC running **Rocky Linux** (RHEL-based) |
| Containers | **Docker**, running services like Pi-hole |
| Remote access | **Tailscale** (private mesh VPN) + SSH from Windows and iPhone (Termius) |
| DNS / ad blocking | **Pi-hole** in a Docker container |

Nothing on the server is exposed to the public internet. Remote access goes only through my private Tailscale network.

## Projects

| Project | Status | Summary |
|---|---|---|
| [Drain Lab Dashboard](projects/drain-lab-dashboard/) | 🟡 In progress | A read-only status dashboard for the server, installed on my iPhone as a home screen web app |
| [Pi-hole Setup and Testing](projects/pihole-setup/) | 🟡 In progress | Network-wide ad and tracker blocking with Pi-hole in Docker |

## Notes

- [Linux command cheat sheet](notes/linux-commands.md) — commands I've used, with what they do
- [Lab changelog](notes/lab-changelog.md) — server changes outside specific projects (tools installed, updates)

## How this repo is organized

```
projects/<project-name>/
  README.md      what the project is, the design decisions and why
  build-log.md   dated log of what I actually did, including what broke
notes/           reference notes I build up over time
```

Passwords, IP addresses and network names are left out of every file on purpose.
