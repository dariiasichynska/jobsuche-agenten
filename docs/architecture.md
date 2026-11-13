# Architecture

## Principles

1. **One agent, one task.** Each workflow has a single responsibility and can be tested on its own.
2. **Suggestions by AI, checks by rules.** Facts that can be checked without AI (whether the website exists, the address, the distance to the office) are checked in code.
3. **No arithmetic by the model.** Structured parts are returned by Gemini; the score is computed in a code node.
4. **Human in the loop.** Nothing is sent to employers automatically. Drafts are stored in Google Drive; they are reviewed, and applications are sent manually.
5. **Free tier first.** The Gemini free tier is used with a primary and a fallback model on separate quotas; one Gemini call is made per job, and the number of jobs per run is capped.

## Components

### Job-Scout (n8n, daily 03:17)

```
Schedule ─► ping start ─► server ready? (3 tries, 10 min apart; else Telegram + ping fail, stop)
─► Parameter ─► read known jobs (Airtable) ─► three branches in parallel
   ├─ Federal Employment Agency: 30 search terms → search API (v6) → title pre-filter
   │    → commute check from coordinates → public detail page → plain text
   ├─ Gmail label "Job-Alerts": extract job links (LinkedIn, StepStone, Indeed, XING)
   │    → load each page once → JSON-LD JobPosting (title, employer, place, description)
   └─ Wishlist career pages: detect known ATS from the URL
        (Personio, Greenhouse, Lever, Recruitee, Ashby, SmartRecruiters, Workable) → public API
─► merge ─► dedupe (canonical URL + normalised title/employer) ─► round-robin across sources
─► cap (25 per run, 8 slots reserved for the Federal Employment Agency) ─► per job: text ≥ 300 chars? ─► Scorer │ else "pre-check" record
─► short Telegram summary (only jobs ≥ 75 with links, the rest as counts, failed Scorer calls counted) ─► ping success
```

- Platforms are **not scraped**. Job alerts are sent by LinkedIn, StepStone and others by e-mail, and one Gmail label is read by n8n via OAuth. For a new platform, only the same label is needed.
- With canonical URLs (LinkedIn job ID, Indeed `jk`, StepStone ID, tracking parameters removed), the same job is not scored twice.
- Sources are processed round-robin before the cap is applied; otherwise all 25 slots were taken by the first search term.

### Scorer (n8n + Gemini)

The Scorer is started by a form (manual use) or by an "Execute Workflow" trigger (Job-Scout). The same path is run in both cases:

```
Parameter (profile + rules) ─► build prompt ─► Gemini (3 tries) ─┬─► score in code ─► Airtable
                                                 └─ on error ─► fallback model ─┘
```

The following values are written to Airtable: score, scope summary, matches, gaps, open questions, hard-exclusion text, status (`Bewertet` or `Nicht beworben`) and the full calculation in the notes. See [scoring.md](scoring.md).

### Employer-Scout (n8n + Gemini, Sundays 05:47)

```
Parameter (weekly keyword from 17 entries in 4 clusters) ─► read wishlist ─► Gemini (JSON schema, temperature 0)
─► parse + dedupe ─► V1 website exists ─► V2 careers page belongs to the company or a known ATS
─► V3 company name on the page ─► address from the imprint page (regex, no AI)
─► geocode (OpenStreetMap Nominatim, 1 req/s) ─► straight-line distance ─► Airtable
```

Companies that fail V1 or V3 are never saved; they are listed only in the run summary. If more than 30 % of a run are rejected, the prompt or keyword is flagged in the summary.

### Follow-up reminder (n8n, daily 08:17)

Records are selected by an Airtable filter formula: follow-up date ≤ today and status *applied / interview / answer received*. One Telegram message is sent for all due items. No AI is used, and no data is written.

### Error alert (n8n)

An Error Trigger workflow is assigned as *Error workflow* in the scheduled workflows. Workflow, node, message and execution link are sent to Telegram. Since v1.1, no Code node is used, so the message is also sent when the task runner of the server is down.

### Watchdogs (outside n8n)

- **Healthchecks.io**: one check is set up per scheduled workflow (Job-Scout, Follow-up reminder, Employer-Scout). `/start` is pinged after the trigger and the plain URL at the end; `/fail` is also pinged by the Job-Scout when the server is not ready. If the success ping does not arrive within the grace time, an alert is sent by Telegram and e-mail. Cancelled and crashed runs, which are not seen by the n8n error workflow, are covered in this way.
- **UptimeRobot**: `/healthz/readiness` of the server is checked every 5 minutes; alerts are sent by e-mail.
- During the task-runner failure on 12 Nov, "up" was returned by the readiness endpoint, so this check is not sufficient on its own. That case is covered by the retries on Code nodes and by the Healthchecks pings.

### Writer (Claude)

The Writer is triggered by a status change in Airtable: the column *Meine Entscheidung* is set to "yes" for a job with a score of 75 or more and no exclusion. Then:

- the employer is rated from online reviews (Kununu, Glassdoor, Google) as *Premium / Solid middle / Rather not*, and the key facts are noted without links; nothing is invented. For *Rather not*, no package is created;
- a cover letter is written as **e-mail text** (subject line, one paragraph per line, no hard line breaks), because most applications are sent by e-mail; a PDF letter is produced only as a secondary format;
- the CV is tailored to the wording of the ad; **only evidenced experience** from a curated skills file is used;
- notes for the application portal are added (reference number, contact person, open questions).

The files are stored in one Google Drive folder per job. The employer's name is never used in file names, because the files may be seen by the employer. Rules from the first package: approved wording is never changed without approval; only the CV is attached.

## Data model (Airtable)

**Jobs and applications**: title, link, source, scope, score 0–100, matches, gaps, hard criteria, status, discovered on, open questions, notes, applied on, follow up on, salary gross per year.
Status flow: `Neu` → `Bewertet` → `Entwurf bereit` → `Freigegeben` → `Beworben` → `Antwort erhalten` / `Interview` (or `Nicht beworben`).
Additional columns: *Nr.* (auto number, used to refer to a job by its number), *Meine Entscheidung* (yes/no for the Writer), *Arbeitgeber-Check*, *Fakten*.
In the view "My selection", jobs with a score of 75 or more or with an active application status are shown.

**Employer wishlist**: name, Berlin address (from imprint), industry, work mode, careers page, status, source, last checked, notes (cluster, distance, warnings, model used).

## Reliability

| Problem | Measure |
| --- | --- |
| Gemini 503 "high demand" | 3 retries with pause, then fallback model with its own quota |
| Free-tier daily quota | 1 call per job, max 25 jobs per run; Employer-Scout 1 call per week |
| Silent failures on schedule | Error workflow → Telegram (no Code node since v1.1) |
| Cancelled or crashed runs, server down | Healthchecks.io pings, UptimeRobot, readiness check before the run |
| Task runner briefly not available | every Code node retries up to 5× with 5 s pause |
| Executions not inspectable | `saveDataSuccessExecution: all` is set on scheduled workflows for now; it is switched off once the runs are stable, because alert e-mails are contained in the execution data |
| n8n instance of a training provider | narrow, replaceable keys; workflows are exported as versioned JSON |
