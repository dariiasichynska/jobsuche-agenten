# Lessons learned

Real problems from building and running the workflows, in the order they appeared.

## AI facts need a rule behind them

- **Invented addresses.** Gemini gave a wrong Berlin address for 7 of 22 companies, and reused one company's address for several unrelated firms. Fix: read the address from the company's own imprint page with a regex; the AI address is kept only as a marked fallback ("not verified").
- **Guessed career pages.** The model built URLs like `domain.com/karriere`. Fix: a careers page counts only if it answers and belongs to the company's domain or a known applicant tracking system.
- **Real companies rejected.** Names with special characters, short domains or brackets ("Brand (Legal Name GmbH)") failed the name check. Fix: normalise legal forms (including B.V., S.A.), compare the part outside and inside brackets separately, accept bot-protected sites when the domain matches the name.

## Free tier limits are per model

- A scheduled run failed with 503 "high demand". Retries alone did not help.
- The first "fallback" model was only an alias of the primary and shared its daily quota. Two older models returned 404 for new API users.
- Working setup: primary `gemini-3.1-flash-lite`, fallback `gemini-3.8-flash`, genuinely separate quotas. Model names live in one Parameter node.
- Scheduling at night and on Sunday morning avoids peak load.

## Public APIs change without notice

- The job search API's v4 search route started returning 403. v6 works.
- The REST detail endpoints (v3, v4) also returned 403. The **public detail page** is server-rendered and contains the full ad, so the workflow reads that instead.
- The nationwide home-office query returns 0 in v6. Open.

## n8n details that cost hours

- The Code node sandbox has no `URL` object; URL parsing has to be done with string functions, and tests must run without it too.
- HTTP responses in text mode arrive in `data`, not `body`. Read both.
- An imported IF node in an older format evaluated `true` as false (warning icon). Fix: build IF nodes in the current format (typeVersion 2.2 with `conditions.options.version: 2`).
- In a loop, `$('Node').all()` returns only the last run. The summary iterates over run indexes to count every scored job.
- A Wait node inside the loop kept manual runs in "Running" after they had finished. Removed; the Gemini call itself is slow enough.
- Telegram rejected messages because job titles contain `*` and `_` (Markdown). Fix: HTML parse mode with `& < >` escaped.
- An empty run created a record "job without title". Fix: guard for the "nothing" item.

## Volume and fairness between sources

- 25 slots per run were all taken by the first search term. Fix: round-robin across search terms and sources before the cap.
- The first run loaded 244 detail pages. Fix: cap the list at twice the per-run limit before loading details.
- Commute filter before the Scorer: offices too far away without home office are dropped using the coordinates the API already provides, which saves Gemini calls.

## Writing for people

- A cover letter copied from a PDF keeps every layout line break. Most applications go by e-mail, so the main output is now e-mail text: one paragraph per line.
- File names must not contain the employer's name.
- Approved wording is never changed without asking, and nothing is added that I did not decide (for example attachments).

## When the server fails at night

- **The error workflow has blind spots.** It fires only for runs that end with status *Error* and were started by a schedule or another workflow. A run cancelled by a too-short timeout, a crashed run and every manual test stay silent. External watchdogs (Healthchecks.io) close that gap.
- **The alarm must not depend on what fails.** On 12 Nov the server's task runner was down. The error workflow started with a Code node, so it failed in the same way as the workflow it was meant to report. Version 1.1 builds the message in the Telegram node itself.
- **"Up" is not "working".** The readiness endpoint answered "up" while Code nodes could not run. Retries on Code nodes and a success ping at the end of each run are what actually prove a run worked.
- **A timeout is not a fix.** A forced limit hides the cause of a long run. The first nightly run was cancelled by a timeout set to 2 minutes by mistake; the real causes were on the server (database in recovery mode, task runner not available).
- **Check the free plan before relying on an alert channel.** UptimeRobot's free plan has no Telegram alerts, so it reports by e-mail; Telegram alerts come from Healthchecks.io and the error workflow.

## AI estimates distances badly

- A job about 3 km from my reference point was excluded as "more than 7 km away", because the model judged the distance from the district name. Planned fix (v1.7): no distance exclusion from the model; Berlin ads without an exact address get a "check address" flag instead.
