# Tutor10x uptime monitor

Off-VM uptime monitor for the public Tutor10x endpoints (ops#209 prevention #1).

It runs as a scheduled **GitHub Actions** workflow on GitHub's own infrastructure, so it keeps
alerting **even when the Tutor10x VM is down** — the gap that made the 2026-11-18 billing-suspension
outage silent (the only on-VM alert can't fire when the VM is off).

**Why this repo is public:** standard-runner minutes are free for public repos, so the 5-minute
schedule costs nothing and can't drain the org's shared private-repo Actions minutes. Everything
here is non-sensitive — public hostnames and HTTP status codes only. No secrets, no PATs.

## What it does
- `cron: */5` — probes `class.`, `chat./health`, `wil.`, `memory./appversion` (expect `200`) and apex `tutor10x.com/` (expect `301`).
- Probes run **in parallel**, in two rounds ~10s apart; an endpoint must fail **both** rounds to count as DOWN (ignores single blips). A full-outage run finishes in well under 2 minutes.
- On DOWN: opens — or updates — an issue labelled `incident` (+ `uptime`), **assigns it to `mmotahar` and @mentions him**, so he's emailed without having to watch the repo. The run also goes red.
- On recovery: comments "✅ Recovered" and closes the issue.

## TE drill
Actions → **Uptime monitor** → **Run workflow**, set `test_url` to a dead URL
(e.g. `https://class.tutor10x.com/__down_drill__` → 404, or a non-resolving host → 000).
The drill uses a separate `[uptime-test]` marker, so it opens/closes its **own** drill incident
and never touches the real `[uptime]` one. Re-run with `test_url` blank to exercise the recovery
close.
