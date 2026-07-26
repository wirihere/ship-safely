# Ship Safely

A pack of [Agent Skills](https://github.com/anthropics/skills) for working on production projects without breaking them.

Most AI coding tools either auto-ship everything (dangerous for live sites) or gate everything (you lose the backup habit). Ship Safely splits the difference: **backups happen silently, production changes get gated.**

## What's inside

| Skill | What it does |
|---|---|
| **commit-push-deploy** | Commits and pushes are automatic backups. Deploys need an explicit human "yes." Database migrations always ask first. |
| **source-of-truth** | When investigating or instructing, check the third-party service's API/docs first — not the database, not memory. Prefer the API over the dashboard. |

## Install

In Claude Code (or any client that supports the Agent Skills plugin format):

```
/plugin marketplace add <your-github-username>/ship-safely
```

Or reference the skills directly by importing the `SKILL.md` files into your project's `CLAUDE.md`:

```
@path/to/ship-safely/skills/commit-push-deploy/SKILL.md
@path/to/ship-safely/skills/source-of-truth/SKILL.md
```

## Why these skills exist

Two patterns cause the most damage when AI assists on a production codebase:

1. **Silent deploys** — code goes live without anyone confirming it should. A botched deploy on a live business (real customers, real money) is the worst kind of bug.
2. **Stale assumptions** — the AI trusts the database over the actual service, or gives dashboard instructions from memory that no longer match the UI.

Ship Safely gates both. Deploys need a human. Investigation starts at the source.

## Philosophy

- **Commit freely** — saving snapshots costs nothing and builds the backup habit.
- **Push automatically** — backups should be silent.
- **Deploy carefully** — production is sacred; only a human says "go live."
- **Check the source** — the service is the truth; the database is a mirror; memory is unreliable.

## License

MIT — use it, fork it, share it.
