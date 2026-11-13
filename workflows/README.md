# Workflows

The files are n8n exports (JSON). They are imported in n8n via *Workflows → Import from File*. Afterwards, separate credentials are selected in the nodes that are named in the sticky note of each workflow.

| File | Workflow |
| --- | --- |
| `job-scout.json` | Job-Scout v1.6 |
| `scorer.json` | Scorer v1.5 |
| `employer-scout.json` | Employer-Scout v1.3.7 |
| `follow-up-reminder.json` | Follow-up reminder (Telegram) v1.3 |
| `error-alert.json` | Error alert (Telegram) v1.1 |

A one-sentence German note is stored for every node under *Settings → Notes*.

The following placeholders have to be replaced: `YOUR_AIRTABLE_BASE_ID`, `YOUR_JOBS_TABLE_ID`, `YOUR_WISHLIST_TABLE_ID`, `YOUR_TELEGRAM_CHAT_ID`, `YOUR_HEALTHCHECKS_UUID` (one check per workflow), `YOUR_N8N_HOST` (readiness check), `YOUR_SCORER_WORKFLOW_ID` (node *Scorer aufrufen*), the reference point `REFERENCE_POINT` in the Parameter nodes, and the candidate profile, the minimum salary and `VERFUEGBAR_AB` (availability) in the Parameter node of the Scorer.

*Fehlerbenachrichtigung (Telegram)* is set as *Error workflow* in the settings of the other four workflows.

The Airtable field IDs (`fld…`) belong to the original base. For a different base, the fields have to be mapped again, or the HTTP nodes have to be switched to field names.
