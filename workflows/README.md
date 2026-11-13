# Workflows

n8n exports (JSON). Import in n8n via *Workflows → Import from File*, then select your own credentials in the nodes named in each workflow's sticky note.

| File | Workflow |
| --- | --- |
| `job-scout.json` | Job-Scout v1.6 |
| `scorer.json` | Scorer v1.5 |
| `employer-scout.json` | Employer-Scout v1.3.7 |
| `follow-up-reminder.json` | Follow-up reminder (Telegram) v1.3 |
| `error-alert.json` | Error alert (Telegram) v1.1 |

Every node has a one-sentence German note under *Settings → Notes*.

Placeholders to replace: `YOUR_AIRTABLE_BASE_ID`, `YOUR_JOBS_TABLE_ID`, `YOUR_WISHLIST_TABLE_ID`, `YOUR_TELEGRAM_CHAT_ID`, `YOUR_HEALTHCHECKS_UUID` (one check per workflow), `YOUR_N8N_HOST` (readiness check), `YOUR_SCORER_WORKFLOW_ID` (node *Scorer aufrufen*), the reference point `REFERENCE_POINT` in the Parameter nodes, and the candidate profile, minimum salary and `VERFUEGBAR_AB` (availability) in the Scorer's Parameter node.

Set *Fehlerbenachrichtigung (Telegram)* as *Error workflow* in the settings of the other four workflows.

Airtable field IDs (`fld…`) belong to my base. With your own base, map the fields again or switch the HTTP nodes to field names.
