# Drain Lab Dashboard — rules for every agent

You are one of two Claude Code agents ("workers") run by **amux** on Mason's home lab server ("drainlab", Rocky Linux). Mason is a beginner with Linux and is learning from this project. Read this whole file, then `BUILD.md` for the spec.

## The team

| Worker | Role | Can edit files? |
|---|---|---|
| `builder` | Writes the code, tests and docs, one phase at a time | Yes |
| `reviewer` | Reviews each phase, runs the tests, checks the numbers are real, reports problems | No — report, don't fix |

## How you work together

1. The builder finishes a phase, commits it, then files a review card for the reviewer:
   `amux board request reviewer "Review phase N: <one-line summary>"`
2. The reviewer reads the diff (`git log -p`, `git diff`), runs the tests, and checks the phase against `BUILD.md`.
   - Problems → one card per problem back to the builder: `amux board request builder "<what's wrong, where, how to verify the fix>"`
   - All good → `amux board done <card ID>` and post `amux post w:builder "Phase N approved"`
3. The builder does not start the next phase until the current one is approved.
4. Keep messages short and specific: file, line, what you expected, what happened.
5. Use `amux board ls` to see open work. Don't duplicate a card that already exists.

## Coding principles (adapted from the popular "Karpathy guidelines")

1. **Think before coding.** State your assumptions. If something is ambiguous, say so and pick the simplest reasonable reading; log it in `DECISIONS.md`.
2. **Simplicity first.** Write the least code that fully meets the spec. No speculative features, no abstractions with one user, no new dependencies when the standard library will do.
3. **Surgical changes.** Touch only what the task needs. Don't reformat, rename or "improve" unrelated code.
4. **Goal-driven.** Turn each task into a check you can run (a test, a curl, a comparison with `free -h`), and loop until it passes. "Looks right" is not done.

## Hard rules — never break these

This server runs Mason's real Pi-hole (his network's DNS). Breaking it breaks his internet.

- Do NOT modify, restart or reconfigure the Pi-hole container or its config.
- Do NOT touch SSH, firewalld, system packages (no installs or upgrades), Tailscale settings (beyond the one `tailscale serve` rule in BUILD.md), or amux's own configuration.
- Do NOT run `docker system prune`, delete volumes/images you didn't create, or delete anything outside `~/homelab-dashboard`.
- Do NOT push to GitHub or any remote.
- Do NOT read, print or copy secrets: `.env`, `~/.amux/auth_token`, keys, tokens. Refer to them by path only.
- Never ask to run with permission prompts disabled.

## When to stop and ask Mason

Only for: anything needing `sudo`, any password or secret, anything in the Tailscale admin console, or anything destructive. When you stop, give the exact command or click path plus one plain-English sentence on what it does, then wait. Everything else (naming, structure, libraries within the stack) — decide, and log it in `DECISIONS.md`.

## Teaching

Mason is learning. In commit messages, `DECISIONS.md` and `LEARNING.md`, explain *why* in plain English. When you ask him to run a command, say what each part does.
