# Architecture

## Principles

1. **One agent, one job.** Each workflow has a single responsibility and can be tested on its own.
2. **AI proposes, rules verify.** Facts that can be checked without AI (does the website exist, what is the address, how far is the office) are checked by code.
3. **The model does not do arithmetic.** Gemini returns structured parts; the score is computed in a code node.
4. **Human in the loop.** Nothing is sent to employers automatically. Drafts land in Google Drive; I review and apply myself.
5. **Free tier first.** Gemini free tier with a primary and a fallback model on separate quotas; one Gemini call per job; hard cap of jobs per run.

## Components

### Job-Scout (n8n, daily 03:17)

```
Schedule ─► Parameter ─► read known jobs (Airtable) ─► three branches in parallel
   ├─ Federal Employment Agency: 30 search terms → search API (v6) → title pre-filter
   │    → commute check from coordinates → public detail page → plain text
   ├─ Gmail label "Job-Alerts": extract job links (LinkedIn, StepStone, Indeed, XING)
   │    → load each page once → JSON-LD JobPosting (title, employer, place, description)
   └─ Wishlist career pages: detect known ATS from the URL
        (Personio, Greenhouse, Lever, Recruitee, Ashby, SmartRecruiters, Workable) → public API
─► merge ─► dedupe (canonical URL + normalised title/employer) ─► round-robin across sources
─► cap (25 per run) ─► per job: text ≥ 300 chars? ─► Scorer │ else "pre-check" record
─► short Telegram summary (only jobs ≥ 75 with links, the rest as counts)
```

- Platforms are **not scraped**. LinkedIn, StepStone and others send job alerts by e-mail; n8n reads one Gmail label via OAuth. A new platform only needs the same label.
- Canonical URLs (LinkedIn job ID, Indeed `jk`, StepStone ID, tracking parameters removed) keep the same job from being scored twice.
- Round-robin before the cap: without it, all 25 slots went to the first search term.

### Scorer (n8n + Gemini)

Two entrances: a form for manual use and an "Execute Workflow" trigger for the Job-Scout. Both run the same path:

```
Parameter (profile + rules) ─► build prompt ─► Gemini (3 tries) ─┬─► score in code ─► Airtable
                                                 └─ on error ─► fallback model ─┘
```

Output in Airtable: score, scope summary, matches, gaps, open questions, hard-exclusion text, status (`Bewertet` or `Nicht beworben`) and the full calculation in the notes. See [scoring.md](scoring.md).

### Employer-Scout (n8n + Gemini, Sundays 05:47)

```
Parameter (weekly keyword from 17 entries in 4 clusters) ─► read wishlist ─► Gemini (JSON schema, temperature 0)
─► parse + dedupe ─► V1 website exists ─► V2 careers page belongs to the company or a known ATS
─► V3 company name on the page ─► address from the imprint page (regex, no AI)
─► geocode (OpenStreetMap Nominatim, 1 req/s) ─► straight-line distance ─► Airtable
```

Companies failing V1 or V3 are never saved; they appear only in the run summary. If more than 30 % of a run is rejected, the summary flags the prompt or keyword.

### Follow-up reminder (n8n, daily 08:17)

Airtable filter formula: follow-up date ≤ today and status *applied / interview / answer received*. One Telegram message for all due items. No AI, no writes.

### Error alert (n8n)

Error Trigger workflow assigned as *Error workflow* in the scheduled workflows. Sends workflow, node, message and execution link to Telegram.

### Writer (Claude)

For jobs with score ≥ 75 and no exclusion:

- cover letter as **e-mail text** (subject line, one paragraph per line, no hard line breaks) because most applications go by e-mail; PDF letter only as a secondary format;
- CV tailored to the ad's wording, using **only evidenced experience** from a curated skills file;
- notes for the application portal (reference number, contact person, open questions).

Files go to a per-job folder in Google Drive. File names never contain the employer's name, because the employer may see them. Rules learned from the first package: approved wording is never changed without asking; only the CV is attached.

## Data model (Airtable)

**Jobs and applications**: title, link, source, scope, score 0–100, matches, gaps, hard criteria, status, discovered on, open questions, notes, applied on, follow up on, salary gross per year.
Status flow: `Neu` → `Bewertet` → `Entwurf bereit` → `Freigegeben` → `Beworben` → `Antwort erhalten` / `Interview` (or `Nicht beworben`).
A view "My selection" shows score ≥ 75 or any active application status.

**Employer wishlist**: name, Berlin address (from imprint), industry, work mode, careers page, status, source, last checked, notes (cluster, distance, warnings, model used).

## Reliability

| Problem | Measure |
| --- | --- |
| Gemini 503 "high demand" | 3 retries with pause, then fallback model with its own quota |
| Free-tier daily quota | 1 call per job, max 25 jobs per run; Employer-Scout 1 call per week |
| Silent failures on schedule | Error workflow → Telegram |
| Executions not inspectable | `saveDataSuccessExecution: all` on scheduled workflows |
| Training-provider n8n instance | narrow, replaceable keys; workflows exported as versioned JSON |
