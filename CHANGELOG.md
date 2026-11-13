# Changelog

## 2026-11-12
- **Incident (nightly run):** a Code node could not get a task runner on the n8n server; the run stopped after 12 minutes, and the error workflow v1 stayed silent because it started with a Code node itself.
- **Job-Scout v1.5**: readiness check before the run (3 tries, 10 minutes apart, otherwise stop with a Telegram message); Healthchecks.io pings (start, success, fail); all Code nodes retry up to 5× (5 s); *Scorer input* is now an Edit Fields node; failed Scorer calls are counted in the summary; part of the nightly limit is reserved for the Federal Employment Agency; workflow timeout 60 minutes because of the waiting times.
- **Scorer v1.5**: retries on all Code nodes; scoring unchanged.
- **Error alert v1.1**: no Code node; the message is built in the Telegram node.
- **Follow-up reminder v1.2**, **Employer-Scout v1.3.6**: Healthchecks.io pings, retries on Code nodes.
- **Job-Scout v1.6**, **Follow-up reminder v1.3**, **Employer-Scout v1.3.7**: every node has a one-sentence note (*Settings → Notes*); logic unchanged.
- **Monitoring**: Healthchecks.io (Telegram and e-mail) for the three scheduled workflows, UptimeRobot on the server's readiness endpoint.
- **Writer**: employer check from online reviews (*Premium / Solid middle / Rather not*); a package with employer check built and stored for my review.

## 2026-11-11
- First nightly run of v1.4 cancelled after 2 minutes: the workflow timeout had been set too low by mistake. Manual reruns failed because the server's own database was in recovery mode. Lesson: n8n's error workflow does not fire for cancelled or crashed runs, nor for manual runs → external watchdogs (see 2026-11-12).
- Writer: packages are written only after my explicit "yes" per job in Airtable; employers rated *Rather not* get no package.

## 2026-11-09 / 10
- **Job-Scout v1.3 / v1.4**: StepStone links from alert mails are resolved without following the redirect, only the clean job address is stored and loaded; cookie, login and block pages count as *no text* (pre-check, never Scorer); title pre-filter for alert links before any page is loaded; at most 25 pages per source; search terms rotate with a daily start.
- **Scorer v1.3 / v1.4**: work with customers or on the phone is a hard exclusion; a missing must requirement caps the score at 70 if a whole class of tools or knowledge is missing; degrees, years of experience, driving licence, languages, certificates, office software and tools from known classes are removed from the must gaps in code; a stated salary below the minimum is a hard exclusion (no salary = open question only); "start immediately" costs nothing; gaps only when the ad names them.

## 2026-11-08
- **Job-Scout v1.2**: three sources (job search API v6 + public detail page, Gmail job alerts, career pages on known ATS), commute rule, round-robin before the cap, short Telegram summary in HTML mode, daily at 03:17.
- **Scorer v1.2**: commute rule, language penalty only for explicit C2, sales roles scope ≤ 15, salary below minimum → conditions ≤ 3.
- **Follow-up reminder v1.1**: Telegram HTML mode.
- **Writer**: first application package by hand; rules for e-mail text, file names and attachments.
- Airtable: gross salary per year as one comparable unit; view "My selection".

## 2026-11-07
- **Scorer v1** (form) and **v1.1** (entrance for the Job-Scout).
- **Job-Scout v1** and **v1.1**.
- **Employer-Scout v1.3.5**: 17 keywords in four clusters, negative keywords in the prompt.

## 2026-11-02 / 03
- **Employer-Scout v1.3.4**: retries and fallback model after a 503 on a scheduled run.
- **Error alert** workflow (Telegram).
- Follow-up reminder v1.

## 2026-10-11 to 10-13
- **Employer-Scout v1 → v1.3.3**: checks against invented companies, bot-protected sites, address from the imprint page, distance via OpenStreetMap, weekly schedule with rotating keyword.
