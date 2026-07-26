---
name: commit-push-deploy
description: Git workflow where commits and pushes are automatic backups but deploys require explicit human confirmation. Activates when committing, pushing, deploying, or applying database migrations. Enforces asymmetric safety: back up freely, ship to production carefully.
---

# Commit Freely, Deploy Carefully

One principle: **commits and pushes are cheap backups — do them freely. Deploys touch the live site — they need a human's explicit yes.**

This splits the difference between auto-ship-everything (risky for production) and gate-everything (you lose the backup habit). Backups happen silently; production changes get gated.

## The three steps

1. **Commit** = save a snapshot locally. Safe; doesn't touch the live site. Proactively offer at natural checkpoints and write a meaningful message.
2. **Push** = backup to remote. Automatic via post-commit hook. Never ask about push; it just happens.
3. **Deploy** = put code live. The only step that touches production. Flag when ready, but **nothing goes live without an explicit "yes" from an authorized person.**

## Before committing

- Glance at `git status` + `git diff` so nothing accidental gets staged.
- For **code changes**, run lint/typecheck first — fix before committing if they fail. Skip this for docs/planning-only changes.

## Commit messages

Match the repo's existing style. If no convention exists, use conventional commits: `type(scope): summary` — e.g. `fix(stripe): capture payment on claim`, `docs: update deploy steps`.

## Decision tree

| Change type | Action |
|---|---|
| Docs/planning only | Commit + push. No live impact. |
| Code, still testing | Commit + push. Live site unchanged. |
| Code, verified and ready | Commit + push, then ask to deploy. |
| Database migration | Always ask explicitly. Destructive. Never auto-deploy. |

## Deploying

When code is tested and going-live makes sense, ask *"Ready to deploy?"* and wait for an explicit yes.

After deploying, **paste the command output** (deploy URL, log). Don't just say "done" — silent failures hide between "I ran it" and "it worked."

## Hard rules

- Never commit secrets, `.env`, or credentials.
- Never deploy without explicit confirmation from an authorized person.
- Never apply migrations without explicit confirmation.
- Push is automatic; deploy is human-gated. Never blur the two.
- If unsure whether to commit or deploy, ask.

## Authorized people

Defined in the project's `CLAUDE.md` or `AGENTS.md`. Default: the repo owner only. Add others explicitly — never assume someone is authorized.
