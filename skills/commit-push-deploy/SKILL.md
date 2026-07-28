---
name: commit-push-deploy
description: Git workflow where commits and pushes are automatic backups but deploys require explicit human confirmation, plus how much review a change needs before it goes live. Activates when committing, pushing, deploying, or applying database migrations. Enforces asymmetric safety: back up freely, ship to production carefully, and scale the review to what's at risk.
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

## Review before deploy — scale it to what's at risk

**Testing and reviewing are different things, and testing is the weaker one.**
A change can pass every test you wrote and still be wrong, because the tests
were written by whoever misunderstood the problem. Reviews catch what tests
can't: the case nobody thought to test.

Do NOT review every deploy — a copy fix doesn't earn it, and a review habit
that's tedious gets dropped. Pick the tier by **blast radius**:

| What changed | Before deploying |
|---|---|
| Docs, comments, copy with no logic | Nothing. Ship it. |
| Ordinary code — a page, a form, a display | Re-read your own diff, once, as if someone else wrote it. |
| **Money · safety nets and scheduled jobs · migrations · anything that sends to people · anything changing who can see what** | **Independent review, findings verified, before it goes live.** |

**The hard tier means:** reviewers who did NOT write the change, looking at it
from different angles, each trying to prove it broken rather than confirm it
fine. Then every finding independently checked before you act on it — half of
what a first-pass review reports doesn't survive a second look, and acting on
a false finding makes the code worse.

Why those categories:

- **Money** — a silent money bug is the worst kind. Nobody reports being
  underpaid by a rounding error; it just quietly happens.
- **Safety nets and crons deserve the SAME suspicion as the money code.** This
  is the counter-intuitive one and it's earned: the worst bug ever found in
  bin-sparkle was in a nightly sweep that was deleting customers' paid
  bookings, silently, and had been live the whole time. Nobody audits the thing
  whose job is to prevent problems.
- **Sending to people** — an email can't be unsent, and it multiplies by your
  whole list.
- **Who can see what** — the failure is invisible to you and obvious to the
  person it exposes.

**Review the fix, not just the bug.** Fixes to careful code introduce their own
bugs at a shocking rate — in one project, three times in one audit, three more
that same week, including a fix that reintroduced the exact bug it was fixing.
When you fix something in the hard tier, the fix gets its own review round.
Budget for two, not one.

**Never let a review round conclude "no issues" without saying what was
checked.** A review that names its angles can be judged; one that says "looks
good" can't.

## Deploying

When code is tested and going-live makes sense, ask *"Ready to deploy?"* and wait for an explicit yes.

**Say plainly what "checked" means when you ask.** "I tested it" and "it's been
reviewed" are different claims, and a hard-tier change needs the second one.
Never let "ready to deploy" imply a review that didn't happen.

**A deploy ships everything committed on the checked-out branch, not just what
you were working on.** Check `git log` against what's currently live before
deploying — more than once, a small change has carried a whole day of someone
else's unfinished work live with it.

After deploying, **paste the command output** (deploy URL, log). Don't just say "done" — silent failures hide between "I ran it" and "it worked."

**Then verify against the live site, not the deploy log.** A successful deploy
proves the upload worked, not that the thing works. Check the real URL — and
beware caching: a CDN can serve you the OLD page right after a deploy and make
a good ship look broken.

## Hard rules

- Never commit secrets, `.env`, or credentials.
- Never deploy without explicit confirmation from an authorized person.
- Never apply migrations without explicit confirmation.
- Push is automatic; deploy is human-gated. Never blur the two.
- If unsure whether to commit or deploy, ask.

## Authorized people

Defined in the project's `CLAUDE.md` or `AGENTS.md`. Default: the repo owner only. Add others explicitly — never assume someone is authorized.
