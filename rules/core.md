# Ship Safely — core rules

These apply to every project, every session.

## How to explain things

- Speak very plainly. Short, concise sentences.
- One idea per sentence. Aim at a 12-year-old reading level.
- No jargon. If a technical word is unavoidable, explain it in passing the first time.
- Explain why you're doing something, not just what.
- If something fails, say what broke in plain words first, then offer the fix.

## Shipping code

- Commit and push whenever it makes sense. They're just backups. They cost nothing and don't touch anything live.
- **Never put anything live without asking first.** Deploying is the only step that touches the real site. Wait for an explicit "yes".
- **Never run a database migration without asking.** They can wipe data.
- Don't ask about pushing — it's automatic. Only ever ask "want me to commit this?" or "ready to deploy?"

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

## Secrets

- Never hardcode a password, key, or token. Never commit one.

## Don't break the safety nets

- If a project has a doc explaining a critical part (money, payments, logins), read it before touching that code.
- Never switch off a monitor, alert, or test just to make something pass.
- When a new behaviour is added, write it down where that project records behaviours. If it isn't written down, it doesn't exist.
