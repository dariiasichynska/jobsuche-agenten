# Lessons learned

Real problems from building and running the workflows are listed in the order in which they occurred.

## AI facts need a rule behind them

- **Invented addresses.** A wrong Berlin address was given by Gemini for 7 of 22 companies, and the address of one company was reused for several unrelated firms. Fix: the address is read from the company's own imprint page with a regex; the AI address is kept only as a marked fallback ("not verified").
- **Guessed career pages.** URLs such as `domain.com/karriere` were constructed by the model. Fix: a careers page is accepted only if it responds and belongs to the company's domain or to a known applicant tracking system.
- **Real companies rejected.** Names with special characters, short domains or brackets ("Brand (Legal Name GmbH)") failed the name check. Fix: legal forms are normalised (including B.V. and S.A.), the parts inside and outside brackets are compared separately, and bot-protected sites are accepted if the domain matches the name.

## Free-tier limits are set per model

- A scheduled run failed with 503 "high demand". Retries alone did not help.
- The first fallback model was only an alias of the primary model and shared its daily quota. Two older models returned 404 for new API users.
- Working set-up: primary model `gemini-3.1-flash-lite`, fallback model `gemini-3.8-flash`, with separate quotas. Model names are kept in one Parameter node.
- Peak load is avoided by scheduling runs at night and on Sunday morning.

## Public APIs change without notice

- The v4 search route of the job search API started to return 403. v6 is working.
- The REST detail endpoints (v3, v4) also returned 403. The **public detail page** is rendered on the server and contains the full ad, so this page is read instead.
- The nationwide home-office query returns 0 results in v6. This issue is open.

## n8n details that cost hours

- No `URL` object is available in the Code node sandbox. URLs are parsed with string functions, and the tests are run without `URL` as well.
- HTTP responses in text mode are returned in `data`, not in `body`. Both fields are read.
- An imported IF node in an older format evaluated `true` as false (warning icon). Fix: IF nodes are built in the current format (typeVersion 2.2 with `conditions.options.version: 2`).
- In a loop, only the last run is returned by `$('Node').all()`. In the summary, all run indexes are iterated, so that every scored job is counted.
- With a Wait node inside the loop, manual runs stayed in the state "Running" after they had finished. The node was removed; the Gemini call is slow enough on its own.
- Telegram rejected messages because job titles contain `*` and `_` (Markdown). Fix: the HTML parse mode is used, and `& < >` are escaped.
- An empty run created a record "job without title". Fix: a guard for the "nothing" item was added.

## Volume and fairness between sources

- All 25 slots of one run were taken by the first search term. Fix: search terms and sources are processed round-robin before the cap is applied.
- In the first run, 244 detail pages were loaded. Fix: the list is capped at twice the per-run limit before details are loaded.
- The commute filter is applied before the Scorer: offices that are too far away without home office are dropped by means of the coordinates that are already supplied by the API. Gemini calls are saved in this way.

## Texts for people

- In a cover letter copied from a PDF, every layout line break is kept. Since most applications are sent by e-mail, e-mail text is used as the main output: one paragraph per line.
- The employer's name is never used in file names.
- Approved wording is never changed without approval, and nothing is added without a decision (for example attachments).

## Failures at night

- **The error workflow has blind spots.** It is triggered only for runs that end with the status *Error* and were started by a schedule or another workflow. Runs that are cancelled by a timeout, crashed runs and manual tests are not reported. This gap is closed by external watchdogs (Healthchecks.io).
- **The alarm must not depend on the failing component.** On 12 Nov the task runner of the server was down. The error workflow started with a Code node and therefore failed in the same way as the workflow it was meant to report. In v1.1, the message is built in the Telegram node itself.
- **"Up" does not mean "working".** "Up" was returned by the readiness endpoint while Code nodes could not be run. A successful run is proven only by retries on Code nodes and by a success ping at the end of each run.
- **A timeout is not a fix.** The cause of a long run is hidden by a forced limit. The first nightly run was cancelled by a timeout that had been set to 2 minutes by mistake; the actual causes were found on the server (database in recovery mode, task runner not available).
- **Alert channels are checked against the free plan.** Telegram alerts are not included in the free plan of UptimeRobot, so its alerts are sent by e-mail. Telegram alerts are sent by Healthchecks.io and the error workflow.

## Distances are estimated poorly by AI

- A job about 3 km from the reference point was excluded as "more than 7 km away", because the distance was estimated by the model from the district name. Planned fix (v1.7): no distance exclusion is made by the model; Berlin ads without an exact address are marked with a "check address" flag.
