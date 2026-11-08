# Workflows

n8n exports (JSON). Import in n8n via *Workflows → Import from File*, then select your own credentials in the nodes named in each workflow's sticky note.

| File | Workflow |
| --- | --- |
| `job-scout.json` | Job-Scout v1.2 |
| `scorer.json` | Scorer v1.2 |
| `employer-scout.json` | Employer-Scout v1.3.5 |
| `follow-up-reminder.json` | Follow-up reminder (Telegram) v1.1 |
| `error-alert.json` | Error alert (Telegram) |

Placeholders to replace: `YOUR_AIRTABLE_BASE_ID`, `YOUR_JOBS_TABLE_ID`, `YOUR_WISHLIST_TABLE_ID`, `YOUR_TELEGRAM_CHAT_ID`, the reference point `REFERENCE_POINT` in the Parameter nodes, and the candidate profile and minimum salary in the Scorer's Parameter node.

Airtable field IDs (`fld…`) belong to my base. With your own base, map the fields again or switch the HTTP nodes to field names.
