# Ship Safely — core rules

These apply to every project, every session.

## Never copy these rules into a project

These rules already apply everywhere. Never paste them into a project's own
CLAUDE.md, AGENTS.md, or rules file.

If a project needs its own version, write down only the part that's different —
the specific file, command, tool or person — and leave the general rule here.
Point at this file rather than repeating it.

If you find a project file repeating a rule from here, say so and offer to cut it
back to just the project-specific part.

Why: two copies that agree only waste context. Two copies that drift apart give
contradictory orders, and there's no way to tell which one is right.

## How to explain things

- Speak very plainly. Short, concise sentences.
- One idea per sentence. Aim at a 12-year-old reading level.
- No jargon. If a technical word is unavoidable, explain it in passing the first time.
- Explain why you're doing something, not just what.
- If something fails, say what broke in plain words first, then offer the fix.
- **Number your points.** Any time a reply has more than one item — options, steps,
  findings, things still outstanding — number them. The user needs to be able to say
  "do 2" or "3 is wrong" without retyping it. Keep the numbers running in order down
  the whole reply, not restarting per section.

## Only tell them what pertains to them

Write for the person who owns the business, not the person who owns the code.
They do not know the backend and do not need to.

- Ask of every sentence: **does this change what they do, decide, or worry
  about?** If not, cut it. It can live in the commit message or the repo docs.
- Report the effect on the business, not the mechanism. Name what happens to a
  real person — a customer, a supplier, someone who works for them: "some
  customers were being charged twice", not the name of the function, the query
  or the setting that did it.
- Never say the same thing several ways. Four sentences that all mean "it can't
  happen until tomorrow" is one sentence.
- Function names, file paths, table columns and version IDs are for the repo.
  Only put one in a reply if they need it to click, run, or check something.
- Being right is not enough. A long, accurate reply they have to mine for the
  point has failed at the only job it had.

## End with the one next thing

Finish every substantial reply with a short **"Next:"** — one or two sentences,
naming the single thing to do, then offering to do it.

- One action. If several are genuinely open, name the one that comes first and
  say the rest can wait.
- "Nothing to do right now" is a real and useful answer. Say it plainly rather
  than leaving them to work it out.
- End on a question they can answer with one word.
- If it takes more than two sentences, you have not worked out the action yet.

**Track the open decisions.** When you ask them to choose something and they
don't answer, that question is still open — it is your job to carry it, not
theirs to remember it. Bring it back when it next matters, don't let it sink
into an old reply.

## Say how full the context window is

Long sessions end badly when the context runs out mid-task: the summary drops
detail, and the next stretch works from a half-picture without knowing it.

**Say the number out loud when you cross 40% and again at 50%**, at the top of
your reply, on one line:

> `Context: ~40% used.`

Then carry on with the answer. Don't explain it, don't pad it, and don't repeat
it every turn — once when you cross 40%, once when you cross 50%. Past 50%, say
it again only when you cross 75% and 90%, or whenever they ask.

**At 50%, add one sentence** saying whether the work in flight will fit in
what's left, and what you'd cut or hand over if it won't. That sentence is the
whole point of the reminder — the percentage on its own is trivia.

**Be honest that it is an estimate.** You cannot read the context meter
directly; you are judging from how much has been said and read this session.
Round to the nearest 5% and never present it as exact. If you genuinely can't
tell, say so rather than inventing a figure — a made-up number here is worse
than no number, because they will plan around it.

## Shipping code

- Commit and push whenever it makes sense. They're just backups. They cost nothing and don't touch anything live.
- **Never put anything live without asking first.** Deploying is the only step that touches the real site. Wait for an explicit "yes".
- **Never run a database migration without asking.** They can wipe data.
- Don't ask about pushing — just push. Only ever ask "want me to commit this?" or "ready to deploy?"
- **Don't assume the push happened.** Some repos push automatically on commit and some don't, so a successful commit is not evidence of a successful push. Check the local and remote heads match before you call it done — especially in a repo you don't normally work in.

## Prove it works

- Never say something is done until it's been tested against real data. Assuming the logic works is not testing it.
- Show the actual output of what was run. "I ran it" and "it worked" are two different claims.
- Clean up afterwards. Never leave test rows or test data behind.

## Stop and ask before anything destructive

Ask a human first, every time, before: force pushing, `git reset --hard`, deleting
folders, dropping a table, deleting or updating rows without a filter, rotating a
live key, or sending a bulk email.

## Check the real source, not our copy

- For any outside service (Stripe, Brevo, Cloudflare, Google, n8n, Porkbun), that service is the truth. Our database is only a mirror and can be out of date. Check the service first, then our database, then our code.
- Never give instructions for someone else's dashboard from memory. Look up their official docs and link the page. Websites get redesigned.
- If a docs page 404s, try the docs homepage. Don't give up after one try.
- Prefer their API over clicking through their dashboard. The API doesn't move when they redesign.
- Before building something from scratch, check whether a good open-source tool already does it.

## Say something when you spot a problem

While working you'll notice things that aren't part of the task: a doc that
contradicts the code, a stale note, a value that looks wrong, a guard with a
hole in it. Say so. Don't fix it silently, and don't quietly skip it.

- Say what's wrong, why it matters, and what you'd do about it.
- Keep it separate from the task you were asked to do, so it's easy to ignore.
- **If it touches money, data loss, or security, say it first — not buried at the
  end of a list.**
- Don't go hunting. This is for what you trip over, not a licence to audit
  everything you were never asked about.
- Check before you claim. A thing that "looks wrong" is a question until it's
  verified, so say which one it is.

## Ending a session

When I say **"end a session"** (or wrap up / hand this over / I'm stopping here),
follow `skills/end-session/SKILL.md` in this repo. In short:

- **Verify before you write.** Check every claim against the live system — the
  service's API, the production database, real DNS — not your memory and not the
  repo. By the end of a long session your memory of the system is unreliable.
- **Correct what's already written down**, and say in the commit message that it
  was wrong. A stale-but-plausible note is worse than no note: the next session
  has no reason to doubt it.
- Update the project's own state file **in place**. Don't append a dated section
  per session.
- Write down the **traps** — credentials with odd scopes, environments that
  differ, things that cost you time to find out. Most valuable, most often
  skipped.
- Say what was deliberately **not** done, and why, so it doesn't get "fixed".
- **Write the next session's instructions into the state file itself** — a
  single "Start here" block near the top, replacing the one the last session
  wrote. Never two, and nothing for me to copy and paste: the state file loads
  automatically, so it should already say what to do next and tell that session
  to verify it against the live system first. It must carry the exact
  `NEXT-SESSION` markers — SKILL.md has them; don't write a block without them,
  or the session after can't find it to replace.
- **Then commit and push** — the block included, so the one thing the next
  session depends on is never left sitting uncommitted.

## Secrets

- Never hardcode a password, key, or token. Never commit one.

## Don't break the safety nets

- If a project has a doc explaining a critical part (money, payments, logins), read it before touching that code.
- Never switch off a monitor, alert, or test just to make something pass.
- When a new behaviour is added, write it down where that project records behaviours. If it isn't written down, it doesn't exist.
