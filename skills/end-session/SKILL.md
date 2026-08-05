---
name: end-session
description: Hand a project over cleanly at the end of a working session. Activates when the user says "end a session", "wrap up", "hand this over", or "I'm stopping here". Verifies every claim against the real system before writing it down, updates the project's state file, commits, and writes the next session's instructions into that state file (replacing the previous session's, never appending).
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

   **And beware checking too soon.** A read taken moments after a scheduled slot
   can miss the record of the run that just happened — the job fires, writes its
   row a beat later, and your query returns the previous one. That produces a
   confident "it never ran" in the handoff about a job that ran fine, which is
   worse than not checking: the next session goes looking for a fault that does
   not exist. If a scheduled thing looks dead, wait past its next slot and look
   again before writing it down.

   **A last-run time alone cannot tell you a job is healthy — check what it was
   supposed to PRODUCE, and what it CONSUMES.** A job can fire perfectly on
   schedule and achieve nothing every time, because the work it does fails a
   check inside it, or its input queue is empty, or its output is rejected
   downstream. From the outside that looks identical to a job that stopped
   running. So make three reads, not one: when it last ran, whether the thing it
   creates has appeared since, and the state of whatever it works through.
   Anything marked failed or errored in that queue is the actual diagnosis, and
   it is usually one query away.

   **Then check the recorded run times against the schedule it is configured
   with — as days and hours, not just "recently".** If the timestamps do not
   land where the configuration says they should, the notes' reading of that
   schedule is wrong, not the scheduler: day-of-week and timezone conventions
   differ between tools, and a written-down "runs Tuesdays" is an assumption
   nobody re-derived. Two recorded runs are enough to see the real interval.
   Getting this wrong sends every future session hunting a dead job that is
   running fine — and it survives handoff after handoff, because each one copies
   the claim rather than re-measuring it.

6. **Ask whether a failing alarm is telling the truth before you repeat it.** A
   monitor, health check or test that reports a problem is a claim, not a fact.
   Two things make that claim wrong in opposite directions, and both are common
   at handoff time: a check whose *own logic* is broken (so it fails on correct
   behaviour), and a check whose thresholds no longer match how the system
   normally behaves (so a routine state reads as a fault). Either way it will
   have been failing for a while, which is exactly why nobody looks at it any
   more. Before writing "X is unhealthy", verify the underlying thing
   independently. If the alarm is wrong, say so in the handoff and fix or flag
   the alarm — **a permanently-failing check is a safety net that has been
   switched off without anyone deciding to.**

7. **Check what is actually DEPLOYED against what you committed — and check
   whether anyone else committed while you worked.** Deploying ships everything
   that is committed, not just the change someone had in mind, so a deploy by
   another person (or another session) can put your work live without anyone
   deciding to. Equally, a deploy you assume happened may not have. Compare the
   live version id and its timestamp against your commit log before writing a
   single word about what is or is not live. If two sessions ran on the repo at
   once, say so in the handoff — it explains commits and deploys that otherwise
   look inexplicable to the next session.

   **And do not answer "what is deployed" from version ids alone — probe the
   running system.** Many deploy tools ship the working DIRECTORY, not a commit.
   When a checkout is shared, code that exists in nobody's git history can be
   live in production: someone's uncommitted work was in the folder at the
   moment a build was cut. Comparing version ids to your commit log cannot see
   that, because there is nothing in the log to see. So for anything that
   matters — a route that should require a login, a feature that should still
   be switched off — **ask the live system directly and compare the answer to
   what the code in your tree says it should be.** A difference means something
   is deployed that nobody committed. This is also how you discover that a
   privileged endpoint went out without its guard, which no amount of reading
   git will ever tell you.

8. **Check which branch is actually deployed, and whether the default branch
   would undo it.** "What is deployed" is not only a version id — it is a
   version id *from a branch*. When production is running a feature branch,
   the default branch is no longer a safe thing to deploy: shipping it would
   silently remove live work. Compare the deployed build to the default branch
   and, if they differ, say so at the very top of the handoff in those words.
   A next session that assumes "deploy the main branch" is the normal, safe
   action will undo everything without being warned by anything else.

9. **Assume another session may be LIVE in the same checkout right now — not
   just in the history.** A live one switches branches under your running dev
   server (which hot-reloads, so you silently test the wrong code), owns the
   processes you're about to kill as "orphans", and deploys while you work.
   Before you switch branches, kill a stray process, or clean up, check what
   is running and whose it is — and if two sessions shared the checkout, the
   handoff must say which branch each one owns.

10. **A freshly deployed system can lie to you for a few minutes.** Caches,
   CDNs and edge networks serve the previous copy while a rollout propagates,
   so a check run straight after deploying can report the old behaviour — or
   the old *content* alongside the new. This corrupts the one step everything
   else here depends on. Verify with a fresh session and a cache-busting
   request, and if a result contradicts what you just shipped, re-check before
   believing it in either direction. Reporting "the fix didn't work" when it
   did is as damaging as the reverse.

11. **If your usual access breaks, reach the same fact another way — and never
   carry the old tool's conventions across.** A stored login expiring mid-session
   is common, and the temptation is to write "could not verify" for everything
   that depended on it. Usually another route exists: the service's REST API with
   a token from the project's own secrets, a web dashboard, an admin endpoint on
   the running system. Take it, and record in the handoff which route you used
   and why, so the next session isn't blocked by the same expiry.

   **But treat the substitute as a different tool, because it is.** The same data
   through a different interface can be ordered, paginated, named or filtered
   differently — most dangerously, **a list the CLI returns oldest-first may come
   back newest-first from the API.** Habits formed on one silently produce a
   confident wrong answer on the other, and "what is currently deployed" is
   exactly the kind of fact this corrupts. Never take the first or last element
   of a list to mean newest: **sort explicitly by the timestamp**, and sanity-check
   the result against something you did yourself this session.

12. **Recompute every derived number. Never carry one forward and adjust it by
   feel.** State files accumulate figures that are really measurements of the
   system — how far one branch is ahead of another, how many rows, how many
   checks a suite runs, how many places quote a value. Each session reads the
   last number, reckons it has gone up a bit, and writes a slightly bigger one.
   Nobody ever re-measures, and after a few sessions the figure is wrong by a
   factor. It is convincing precisely because it has been there for ages and
   keeps changing, which reads as maintenance. **If a number can be produced by
   a command, run the command.** If it cannot, say where it came from and that
   it is an estimate. And when you find one that was wrong, say so in the
   handoff and in the commit — a silently corrected number teaches the next
   session nothing, and the same drift starts again.

   **Some numbers cannot be stated without changing them — say which commit
   they belong to.** A count of commits, of files changed, of lines in the state
   file: writing it down is itself a change, so the figure is stale the instant
   it is committed, and the correction is stale again. Do not chase it. Name the
   point it was true at — "N as of commit abc1234" — so the next session can
   tell whether the drift since is one handoff commit or six months of work. A
   bare number with no anchor invites exactly the adjust-by-feel habit this step
   exists to stop.

   **And take the reading at the END, not when you first look.** A number can go
   stale inside the same session that measured it: you count something, keep
   working, change or commit the very thing you counted, and hand over a figure
   that was right an hour ago. It is the identical failure to carrying one
   forward between sessions, except it happens to a number you personally
   verified — which is exactly why you will not doubt it. Measure in the same
   pass as writing the handoff, after the last change lands, and if it moved
   since your earlier reading, say so.

13. **Grep the state file for the values you just superseded.** Having written
   the new numbers, search the whole file for the OLD ones — the previous
   version id, the previous branch head, the previous counts, the phrase that
   has just stopped being true ("not deployed", "nothing built yet"). A state
   file repeats its key facts in several places: a summary at the top, a table
   further down, an aside in a list. Updating the section you were looking at
   and leaving the other three is the most common way a handoff ends up
   contradicting itself, and the copies you miss are the ones nobody was
   thinking about — which is exactly why the next session believes them. This
   is mechanical and takes a minute.

14. **Before writing "tested" or "all passing", check the test actually reached
   the thing.** A suite can go green without exercising the code at all —
   because auth silently failed and every request got a login page, because a
   fixture no longer matches a state the system permits, because the run hit a
   cache, or because it measured a URL that does not exist and got a friendly
   error page. Nothing about that looks like a failure; it looks like success,
   and it is the most convincing wrong sentence you can put in a handoff.
   So for anything you are about to describe as verified, confirm the
   **precondition** as well as the outcome: that it was logged in, that it
   loaded the page you meant, that the assertion would have failed had the
   behaviour been wrong. Cheapest version: break it on purpose once and watch it
   go red. If you cannot show it failing, you have not shown it passing.

15. **When the state file says something has NEVER happened, go and look in the
   place that would record it if it had.** "No customer has reviewed yet", "no
   refund has ever been processed", "nothing has been sold", "that job has never
   run" — these are the easiest claims in the whole file to carry forward,
   because nothing ever prompts you to check them. There is no failure to
   investigate, no alarm going off, no number that looks odd. They simply sit
   there being quietly repeated, and each session that repeats one makes the
   next session trust it more.

   They are also the claims that go stale most suddenly, because the thing they
   describe usually happens exactly once and then is true forever. The cost of
   checking is one count against the store that would hold it. The cost of not
   checking is a handoff that tells the next session a milestone has not been
   reached when it has — so they go looking for a fault in a system that just
   started working, or they leave a feature switched off that is now live and
   unwatched. **Count the rows. It takes seconds and it is the cheapest way to
   catch the change nobody was watching for.**

16. **Count what is sitting in a queue waiting for a HUMAN, right now.** Not
   what a job did, not what a number says — what has arrived and is waiting on
   a person: an application awaiting approval, a payout awaiting sign-off, a
   message nobody answered, a request pending review, an invitation accepted
   but not acted on. These are invisible to every other check here. No alarm
   fires, no test fails, no figure looks odd — the system is behaving perfectly
   by holding the item and waiting, which is exactly the problem.

   They also arrive on someone else's schedule, so one can land in the last hour
   of a session that started before it existed. And the cost of missing one is
   paid by a real person who is waiting and hearing nothing, which is a different
   and worse kind of cost than a stale number.

   **One count per queue, and name anyone who is waiting at the very top of the
   handoff** — above the technical state, because a person waiting outranks a
   version id. If the state file describes someone as not having responded,
   this is also the check that discovers they have.

Stale-but-plausible entries are the dangerous ones: a finished task still listed
as pending, a credential that has been rotated, a value that changed. The next
session has no reason to doubt them.

**Beware the entry that was true when written and is now actively misleading.**
The worst kind points the next session in a *safe-sounding* direction that is no
longer safe — "this code is not deployed" when it now is, "this job has no
payment record" when it now has one. A stale warning is more dangerous than a
stale fact, because it is written to be obeyed.

## What the handoff must contain

- **Current state.** What is deployed, what version, when. What is running.
- **What changed this session,** in outcomes not commits.
- **What is open,** ordered, with enough context to act without this
  conversation. Anything needing a human decision, flagged as such.
- **What was deliberately NOT done, and why.** Otherwise the next session
  "fixes" it and undoes a decision.
- **Where the work lives.** If anything from the session sits on an unmerged
  branch or in an undeployed commit, name the branch and head commit, and
  spell out what putting it live requires — in order, including anything that
  must happen BEFORE the deploy (a migration, a config value, a third-party
  setup step). Work that isn't on the default branch is invisible to a next
  session that never thinks to look, and a deploy checklist without its
  ordering is how the ordering gets violated.
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
- **If the work landed somewhere other than the project whose state file loads
  automatically, update that state file anyway — to say the work was elsewhere.**
  A session often runs from one project's folder while the actual change goes
  into another repo, a shared library, or a third-party dashboard. The state
  file that opens next is the one you were sitting in, and if it is untouched it
  silently implies nothing happened and its figures are current. Both readings
  are wrong. Write one line at the top naming where the work went, and say
  plainly that nothing in this project moved.

  **Then do not re-assert that file's existing figures as if you had checked
  them.** You did not — you were working elsewhere, and re-verifying an entire
  unrelated project is a session's work on its own. Blanking them is worse.
  **Split the block instead: what you re-read this session with today's
  timestamp, and what is carried forward with the timestamp it originally had,
  under a heading that says so.** A single undifferentiated list makes the
  oldest entry look as fresh as the newest, which is exactly the failure this
  whole procedure exists to prevent. Re-check the cheap, high-consequence ones
  regardless — what is deployed, and whether the scheduled jobs are still
  running — because those are what an unrelated session is most likely to have
  broken without anyone noticing.

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

## Write the next session's instructions into the state file

**Do not hand the user a prompt to copy and paste.** The state file is loaded
automatically when someone opens the project, so the instructions for the next
session belong *in it*. Anything that needs copying gets forgotten, pasted
stale, or pasted twice.

Put a single **"Start here"** block directly under the project's title and
one-line description, above every other section — the first thing after the
reader knows what the project is. If a block already exists somewhere else in
the file, move it; never add a second.

**Replace it wholesale every session. Never append, never leave two.** Delete
the block the previous session wrote and put yours in its place — the whole
point is that exactly one set of instructions exists and it is always the
current one. Wrap it in markers so the next session can find and replace it
without guessing:

```
<!-- NEXT-SESSION:START -->
## ▶ Start here — written <date>, end of session
...
<!-- NEXT-SESSION:END -->
```

**The markers are exact, bare and on their own lines** — no prose inside the
tags, nothing else on the line. Find an existing block by searching for the
substring `NEXT-SESSION:START`, never for the whole comment: a marker with a
note baked into it, or wrapped onto two lines, will not match and you will
append a second block instead of replacing the first.

**Before writing, deal with what is already there:**

- **No markers, but a "Start here" / "Next session" section exists** — delete
  that section and write a marked block in its place. Say so in the commit.
- **A START with no END, or markers that don't match** — delete from the START
  marker to the next `---` or top-level heading, then write a clean block.
- **More than one block** — keep none of them. Write one fresh, and say in the
  commit message that duplicates were removed.
- **No state file at all** — create one at the repo root (`CLAUDE.md` unless the
  project uses something else) and put the block in it.
- **A state file that isn't Markdown** — use that file's own comment syntax, or
  a plain `=== START HERE ===` / `=== END START HERE ===` pair. The visible
  heading is the only boundary a human sees in a rendered view, so keep it.

The block contains:

1. **What to read, in order** — the last session's write-up, anything mandatory
   before touching risky areas, then the state section.
2. **"Verify before you trust this."** Say when the figures were last checked
   against the live system, and name how to check them. You have just spent a
   session discovering that written-down claims go stale; assume yours will
   too. Make clear that reporting a mismatch is useful work, not a detour.
3. **The next one to three jobs, concretely and in order** — specific enough to
   act on without this conversation, including what "it worked" looks like.
4. **The project-specific constraints only** — who can approve a deploy here,
   which doc is mandatory before which code, what must never be touched. Point
   at the general rules file for the rest rather than restating it: general
   rules are already loaded every session, and a second copy is free to drift.

Keep it short enough to be read every time. Everything long goes in the dated
session doc and gets linked from point 1.

Then tell the user, in a sentence, that it is written down and they don't need
to paste anything.

## Then commit

Commit and push. The commit message is part of the handoff — say what was
corrected and what is still open, not just "update docs".

**Then confirm the push actually landed** — compare the local and remote head
commits. Some repos push automatically on commit and some don't, so a
successful commit is not evidence of a successful push. A handoff that exists
only on your machine is not a handoff.

If you touched more than one repo this session, check every one of them.

Commit any change you made to this procedure too, and say in the message what
prompted it.

## Hard rules

- **Never write a claim you have not checked.** Mark it unverified instead.
- **Never leave a contradiction in the state file.** If two entries disagree,
  find out which is true and delete the other.
- **Never quietly drop something that is still broken** because the session ran
  long. Unfinished is fine; invisible is not.
- Clean up test data and temporary files before you finish.
- **Before deleting any working directory, check whether it contains a LINK to
  something shared.** Sessions routinely link a throwaway copy to a shared
  dependency folder to avoid a slow reinstall. A recursive or forced delete can
  then follow that link and destroy the *shared* thing, not the copy — and the
  damage is close to invisible, because the top-level entry count of the shared
  folder is unchanged and only something deep inside is gone. It surfaces later
  as a build failing on a missing package, in a session that has no idea a
  cleanup caused it. **Remove the link first, confirm it is gone, confirm the
  shared folder is still intact, and only then delete the directory.** If a
  forced delete fails with a permission error, treat that as a warning and look
  before overriding it — a partial delete may already have happened.
- **Restore any file you overwrote from version control, not from a backup you
  made yourself.** Sessions that stage a file temporarily — copying a working
  file over a shared entry point, then putting it back — routinely restore the
  wrong thing, because the backup was taken after the first overwrite rather
  than before it. `git checkout -- <file>` is the only restore that is
  certainly correct. Then confirm the file is what it should be, not merely
  that a restore command ran, and check no `.bak` files were left behind.
- If the session ended mid-task, say exactly where it stopped and what the next
  concrete step is.
- **Never leave a gap in this procedure unwritten.** If the handoff needed a
  step that isn't here, add it and say so — see "Improve this procedure as you
  use it" above.
