---
name: end-session
description: Hand a project over cleanly at the end of a working session. Activates when the user says "end a session", "wrap up", "hand this over", or "I'm stopping here". Verifies every claim against the real system before writing it down, updates the project's state file, commits, and produces a copy-paste prompt for the next session.
---

# End a Session

One principle: **the next session starts from what you wrote down. If it's stale, they act on a lie.**

A handoff is not a summary of what you did. It is a correct description of
**where things actually are now** — which is a different thing, and the gap
between them is where the next session wastes its time.

## The order matters: verify, then write

**Do not write the handoff from memory or from the conversation.** By the end of
a long session your memory of the system is a mix of what was true at the start,
what you changed, and what you *intended* to change. Check the real thing.

For every claim you are about to write down, and every claim already in the
state file:

1. **Check it against the live system.** The service's API, the production
   database, real DNS, the deployed version. Not the repo, not your notes.
2. **If it's wrong, say so explicitly in the commit message.** "This said X; it
   is actually Y" is far more useful to the next session than a silent edit.
3. **If you can't verify it, mark it unverified.** An honest "not checked" is
   safe. A confident wrong statement is not.
4. **Timestamp what you did verify.** "Re-read from the live API at
   <date/time>" tells the next session how much to trust it and when to
   re-check. A bare fact with no date reads as permanently true.
5. **Check that anything scheduled has actually run.** Crons, queues, sync
   jobs, backups, scheduled reports. Don't check that they are *configured* —
   check the last time each one really executed, and compare that to when you
   last changed the thing it runs. Something that silently stopped, or that has
   not run since your changes went out, is invisible in every other check you
   do, and it means your work is untested no matter how good the code is. This
   is often the single most important line in the handoff.

Stale-but-plausible entries are the dangerous ones: a finished task still listed
as pending, a credential that has been rotated, a value that changed. The next
session has no reason to doubt them.

## What the handoff must contain

- **Current state.** What is deployed, what version, when. What is running.
- **What changed this session,** in outcomes not commits.
- **What is open,** ordered, with enough context to act without this
  conversation. Anything needing a human decision, flagged as such.
- **What was deliberately NOT done, and why.** Otherwise the next session
  "fixes" it and undoes a decision.
- **Traps.** Credentials with surprising scopes, environments that differ,
  commands that look safe and aren't, anything that cost you time to discover.
  This is the highest-value section and the one most often skipped.
- **Anything corrected,** called out as a correction.

Money, data loss and security items go first, not buried in a list.

## Where it goes

- **The project's own state file** — `CLAUDE.md`, `AGENTS.md`, or whatever that
  project uses. Update it **in place**. Do not append a new dated section each
  session; that produces a changelog nobody reads and contradictions nobody
  notices.
- **Long detail** goes in a dated doc under `docs/` or `reports/`, and the state
  file links to it.
- Keep general working rules out of it — those live here. Only what is specific
  to that project goes in that project.

## Improve this procedure as you use it

**If you notice something this procedure missed, add it here — don't just do it
silently and move on.** You are the only one who sees the gap, and only at the
moment you trip over it. By the next session it is forgotten.

So: while running the handoff, if you find yourself doing a useful check that
isn't written here, or wishing a step existed, **edit this file, then tell the
user plainly what you added and why.** Do not ask permission first — a missing
step in a checklist is a bug in the checklist, and fixing it is part of the job.
Do say what you changed, so they can push back.

Two rules on what goes in:

- **Keep it generic.** This procedure runs across many different projects with
  different state files. Write the transferable lesson, not the project it came
  from — no project names, no service names, no file paths, no specific
  values. If a step only makes sense on one project, it belongs in that
  project's own state file, not here. A good test: strip the project you were
  working on out of the sentence, and see whether it still says something
  useful.
- **Only add what would have changed the outcome.** A step that would have
  caught a real problem, or saved real time. Not everything you happened to do.
  A checklist nobody finishes is worse than a short one.

The same applies in reverse: if a step here turns out to be wrong, misleading,
or no longer worth doing, change or remove it and say so.

## Then commit

Commit and push. The commit message is part of the handoff — say what was
corrected and what is still open, not just "update docs".

Commit any change you made to this procedure too, and say in the message what
prompted it.

## Finally: hand over a prompt

End by giving the user a **copy-paste block** to open the next session with.
It should:

1. Point at the state file and the key docs, in reading order.
2. **Tell the next session to verify the docs against the live system before
   trusting them,** and to report anything that doesn't match. You have just
   spent a session discovering that written-down claims go stale; assume yours
   will too.
3. State the next one or two jobs concretely.
4. Repeat any hard constraint that applies (don't deploy without asking, don't
   run migrations without asking, read the mandatory playbook first).

Keep it short enough to paste without editing.

## Hard rules

- **Never write a claim you have not checked.** Mark it unverified instead.
- **Never leave a contradiction in the state file.** If two entries disagree,
  find out which is true and delete the other.
- **Never quietly drop something that is still broken** because the session ran
  long. Unfinished is fine; invisible is not.
- Clean up test data and temporary files before you finish.
- If the session ended mid-task, say exactly where it stopped and what the next
  concrete step is.
- **Never leave a gap in this procedure unwritten.** If the handoff needed a
  step that isn't here, add it and say so — see "Improve this procedure as you
  use it" above.
