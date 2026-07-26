---
name: source-of-truth
description: When investigating issues with third-party services or instructing the user on how to configure them, always check the actual service first — its API, dashboard, logs, or official docs. Never start from the database (which can lag), never answer from memory (UIs change). Activates when debugging third-party integrations, checking service status, or giving instructions for Stripe, Cloudflare, Brevo, Google, n8n, Porkbun, or any external tool.
---

# Source of Truth First

Two rules that prevent the most common and expensive mistakes:

1. **When investigating:** the third-party service is the source of truth, not your database.
2. **When instructing:** the official docs are the source of truth, not your memory.

## When investigating an issue

If a problem involves any third-party service (Stripe, Cloudflare, Brevo, Google, n8n, Porkbun, etc.), start by checking the actual service — its API, dashboard, logs, or webhook events.

Your database is a mirror. It can lag, desync, or be wrong. Work from the source outward:

1. **Source first** — query the service's API or check its dashboard/logs
2. **Then your database** — compare what you see
3. **Then your code** — understand why they differ

Never assume a webhook fired, a payment succeeded, or an account is onboarded based only on what's in the database. Check the service.

## When giving instructions

Before telling the user how to navigate a dashboard, toggle a setting, or configure a third-party service, check that service's official documentation (via `webfetch`) and cite the source.

Never answer from memory. UIs change, layouts get renamed, and stale instructions waste time.

**When checking docs:**

- Start at the service's docs homepage (e.g., `docs.stripe.com`, `developers.cloudflare.com`)
- If a specific page returns 404, try the docs index or homepage — don't give up after one attempt
- Cite the URL you got the information from

## Prefer the API over the dashboard

When you need to configure something in a third-party service (add webhook events, update settings, create resources), check if there's an API for it first.

- The API is stable — it doesn't change with UI redesigns
- The API is scriptable — can be version-controlled and repeated
- The API is verifiable — you can query the result immediately

Write a script to do it via the API instead of telling the user to click through the dashboard. The dashboard is for viewing; the API is for configuring.
