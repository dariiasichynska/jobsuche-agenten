# Jobsuche-Agenten — an agentic workflow for a personal job search

A small team of AI-assisted workflows is used to find jobs, to score them by **scope of tasks rather than job title**, to prepare application drafts and to send follow-up reminders. The workflows are built with n8n, the free Gemini API tier, Claude and Airtable.

Design rule: **suggestions are made by AI, facts are checked by rules, decisions are made by a human.** Nothing is applied for, sent or published by a workflow on its own.

> Status (13 Nov 2026): in progress. Five n8n workflows are run on a schedule and are monitored from outside (Healthchecks.io, UptimeRobot). The Writer, including an employer check, is triggered by a status change in Airtable; a daily scheduled run is planned. See [Roadmap](#roadmap).

---

## Components in use

| Agent | Version | Tool | Schedule | Task |
| --- | --- | --- | --- | --- |
| **Job-Scout** | v1.6 | n8n | daily, 03:17 | The server is checked for readiness, new job ads are collected from three sources, duplicates are removed, every ad with enough text is passed to the Scorer, and a short Telegram summary is sent. |
| **Scorer** | v1.5 | n8n + Gemini | called by the Job-Scout or a form | Part scores and reasons are returned by one Gemini call; the **final score is computed in code**, not by the model. |
| **Employer-Scout** | v1.3.7 | n8n + Gemini | Sundays, 05:47 | A wishlist of Berlin employers is built; every AI suggestion is checked against the real website and imprint before it is saved. |
| **Follow-up reminder** | v1.3 | n8n | daily, 08:17 | A Telegram reminder is sent 7 days after an application or interview. No AI is used. |
| **Error alert** | v1.1 | n8n | on failure | A Telegram message is sent when a scheduled workflow ends with an error. The workflow is built without a Code node. |
| **Watchdogs** | – | Healthchecks.io, UptimeRobot | every run / every 5 min | An alert is sent by Telegram and e-mail when a run does not start or finish in time, or when the n8n server is down. |
| **Writer** | – | Claude | triggered by a status change in Airtable | The employer is rated from online reviews (*Premium / Solid middle / Rather not*). For jobs with a score of 75 or more, an application package is prepared: a cover letter as e-mail text, a CV tailored to the ad and notes for the application portal. |

Runs are scheduled at night or on Sunday morning, outside other people's working hours and away from API peak times.

---

## Architecture

```mermaid
flowchart LR
  subgraph SRC["Sources"]
    BA[Federal Employment Agency<br/>job search API]
    GM[Gmail label<br/>LinkedIn / StepStone alerts]
    WL[Career pages of the wishlist<br/>known ATS only]
  end
  SRC --> JS[Job-Scout<br/>merge · dedupe · commute check · cap]
  JS -->|ad text ≥ 300 chars| SC[Scorer<br/>1 Gemini call + score in code]
  JS -->|no text| PRE[Airtable: 'pre-check'<br/>link opened by hand]
  SC --> AT[(Airtable<br/>jobs & applications)]
  AT -->|score ≥ 75, no exclusion| DEC{{Decision<br/>yes / no}}
  DEC -->|status set to yes| WR[Writer · Claude<br/>employer check + draft package]
  WR --> HU{{Human review}}
  HU -->|application sent manually| AT
  AT --> FU[Follow-up reminder<br/>Telegram]
  ES[Employer-Scout<br/>weekly] --> WLT[(Airtable<br/>employer wishlist)]
  WLT --> WL
  HC[[Healthchecks.io<br/>start / success / fail pings]] -. watches .- JS
  HC -. watches .- ES
  HC -. watches .- FU
```

The work is split between n8n and Claude:

- Repetitive, high-volume steps are handled by **n8n and the free Gemini tier**. Only public ad text and a short profile **without contact data** are sent to Gemini.
- Every text that is read by an employer is written with **Claude**. Full personal data is kept there.
- **Airtable** is used as the single source of truth: one table for jobs and applications, one for the employer wishlist.

Details: [docs/architecture.md](docs/architecture.md)

---

## Scoring

Weights: scope of tasks 50 · skills and tools 20 · language 10 · location and format 10 · conditions 10.

Part scores, penalties, matches, gaps and open questions are returned by Gemini as JSON. In a code node, each part is clamped to its range, penalties are capped (each at −30, in total at −60), a bonus for internal roles is added and the final score is computed. The full calculation is stored in every Airtable record, so each score can be checked by hand.

The Writer is blocked by hard exclusions: people management, "5+ years of experience in Germany" as a requirement, work with customers or on the phone as a core duty, a stated salary below the configured minimum, and an office outside the commuting radius when on-site or hybrid work is required.

A missing **must** requirement caps the score at 70, but only if a whole class of tools or knowledge is missing (for example SAP, programming, HGB). Another CRM, another task tracker or another automation tool is treated as a gap only, because tools of that class are already used by the candidate.

Details: [docs/scoring.md](docs/scoring.md)

---

## First real runs (November 2026)

- Federal Employment Agency, one run: 958 hits → 109 after the title pre-filter → 27 dropped as too far away without home office → round-robin across 30 search terms → 25 ads scored.
- Gmail alerts: 35 job links were extracted in one run.
- Employer-Scout, earlier tests: a wrong Berlin address was given by Gemini for **7 of 22** companies. Since v1.3, the address is read from each company's imprint page by a rule, and the AI address is used only as a marked fallback.

## Incident 2026-11-12

**Cause.** In the 03:17 run, no task runner was available on the n8n server for a Code node. The error workflow also started with a Code node, so no error message was sent.

**Impact.** The run was stopped after twelve minutes. Four jobs were saved; no summary was sent.

**Fix.**

- Before each run, the readiness endpoint of the server is checked up to three times, ten minutes apart. If the server is not ready, the run is stopped and a Telegram message is sent.
- Every Code node is retried up to five times. The node that failed was replaced by an Edit Fields node, which needs no task runner.
- The message of the error workflow is built without a Code node.
- A ping is sent to Healthchecks.io at the start and at the end of every run. If the success ping does not arrive in time, an alert is sent by Telegram and e-mail. Cancelled and crashed runs, which are not covered by the n8n error workflow, are detected in this way.
- The server is checked by UptimeRobot every five minutes.

Lessons learned: [docs/lessons-learned.md](docs/lessons-learned.md)

---

## Roadmap

- [x] Employer-Scout with checks against invented companies, addresses and links
- [x] Job-Scout with three sources and a commute rule
- [x] Scorer with a deterministic final score
- [x] Follow-up reminder and error alerts via Telegram
- [x] Watchdogs: readiness check, retries, Healthchecks.io, UptimeRobot
- [x] Writer: first application package as a style reference; employer check (Kununu, Glassdoor, Google reviews → Premium / Solid middle / Rather not)
- [ ] Writer as a daily scheduled task
- [ ] Job-Scout v1.7 / Scorer v1.6: a "check address" flag instead of a distance exclusion, an overflow queue instead of dropped jobs over the nightly limit, one application per company
- [ ] Job-Scout v2: no title filter, a low-cost AI triage by tasks, a weekly "search strategist" that derives new search terms from well-scored jobs
- [ ] ATS check of the CV
- [ ] Weekly review
- [ ] Migration of n8n from the training provider's instance to a separate server (VPS) before February 2027

---

## Repository layout

```
README.md
docs/
  architecture.md      components, data flow, Airtable tables, schedules
  scoring.md           weights, penalties, exclusions, score computed in code
  lessons-learned.md   real failures and their fixes
  privacy.md           data sent to each AI service, data that is never sent
workflows/             n8n exports with IDs and personal data removed
CHANGELOG.md
```

## Removed before publishing

Airtable base and table IDs, the Telegram chat ID, the Healthchecks ping addresses, the n8n host, the reference point of the commute rule, the candidate profile, the availability date and the salary expectations were removed. The structure and code of the workflows are kept; placeholders are marked `YOUR_…`.

## Licence

MIT, see [LICENSE](LICENSE).
