# Changelog

## 2026-11-13
- The documentation was rewritten in an impersonal, passive style. No workflow was changed.

## 2026-11-12
- **Incident (nightly run):** no task runner was available for a Code node on the n8n server. The run was stopped after 12 minutes, and no message was sent by the error workflow v1, because it also started with a Code node.
- **Job-Scout v1.5:** a readiness check is run before each run (3 tries, 10 minutes apart; otherwise the run is stopped and a Telegram message is sent). Healthchecks.io pings are sent (start, success, fail). All Code nodes are retried up to 5 times (5 s). *Scorer input* was replaced with an Edit Fields node. Failed Scorer calls are counted in the summary. Part of the nightly limit is reserved for the Federal Employment Agency. The workflow timeout was set to 60 minutes because of the waiting times.
- **Scorer v1.5:** retries were added to all Code nodes. The scoring was not changed.
- **Error alert v1.1:** the Code node was removed; the message is built in the Telegram node.
- **Follow-up reminder v1.2** and **Employer-Scout v1.3.6:** Healthchecks.io pings and retries on Code nodes were added.
- **Job-Scout v1.6**, **Follow-up reminder v1.3** and **Employer-Scout v1.3.7:** a one-sentence note was added to every node (*Settings → Notes*). The logic was not changed.
- **Monitoring:** Healthchecks.io (Telegram and e-mail) was set up for the three scheduled workflows, and UptimeRobot for the readiness endpoint of the server.
- **Writer:** an employer check from online reviews (*Premium / Solid middle / Rather not*) was added. A package with an employer check was created and stored for review.

## 2026-11-11
- The first nightly run of v1.4 was cancelled after 2 minutes, because the workflow timeout had been set too low by mistake. Manual reruns failed, because the database of the server was in recovery mode. As a consequence, external watchdogs were planned, because cancelled, crashed and manual runs are not reported by the n8n error workflow (see 2026-11-12).
- Writer: a package is created only after "yes" has been set for the job in Airtable. No package is created for employers rated *Rather not*.

## 2026-11-09 / 10
- **Job-Scout v1.3 / v1.4:** StepStone links from alert e-mails are resolved without following the redirect; only the clean job address is stored and loaded. Cookie, login and block pages are treated as *no text* (pre-check, never Scorer). A title pre-filter is applied to alert links before any page is loaded. At most 25 pages are loaded per source. Search terms are rotated with a daily start.
- **Scorer v1.3 / v1.4:** work with customers or on the phone is a hard exclusion. A missing must requirement caps the score at 70 if a whole class of tools or knowledge is missing. Degrees, years of experience, driving licence, languages, certificates, office software and tools from known classes are removed from the must gaps in code. A stated salary below the minimum is a hard exclusion; if no salary is stated, only an open question is added. "Start immediately" costs no points. Gaps are listed only if they are named in the ad.

## 2026-11-08
- **Job-Scout v1.2:** three sources are used (job search API v6 with the public detail page, Gmail job alerts, career pages on known ATS). A commute rule, a round-robin before the cap and a short Telegram summary in HTML mode were added. The workflow is run daily at 03:17.
- **Scorer v1.2:** a commute rule was added. The language penalty is applied only for explicit C2. The scope of sales roles is capped at 15. A salary below the minimum caps the conditions at 3.
- **Follow-up reminder v1.1:** the Telegram HTML mode is used.
- **Writer:** the first application package was created by hand. Rules for e-mail text, file names and attachments were defined.
- Airtable: the gross salary per year is stored as one comparable unit. The view "My selection" was added.

## 2026-11-07
- **Scorer v1** (form) and **v1.1** (entrance for the Job-Scout) were created.
- **Job-Scout v1** and **v1.1** were created.
- **Employer-Scout v1.3.5:** 17 keywords in four clusters and negative keywords in the prompt are used.

## 2026-11-02 / 03
- **Employer-Scout v1.3.4:** retries and a fallback model were added after a 503 error in a scheduled run.
- The **error alert** workflow (Telegram) was created.
- Follow-up reminder v1 was created.

## 2026-10-11 to 10-13
- **Employer-Scout v1 → v1.3.3:** checks against invented companies, handling of bot-protected sites, the address from the imprint page, the distance via OpenStreetMap and a weekly schedule with a rotating keyword were added.
