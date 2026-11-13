# Privacy by design

| Data | Sent to | Never sent to |
| --- | --- | --- |
| Public job ad text | Gemini (Scorer) | — |
| Short profile: experience, tools, languages, no name, no contact data | Gemini (Scorer) | employers, public repository |
| Search keywords, company names | Gemini (Employer-Scout) | — |
| Reference point of the commute rule | coordinates in code nodes; only a nearby public square is named in the Scorer prompt | — (no home address is used anywhere) |
| Full CV, contact data, application texts | Claude (Writer), Google Drive | Gemini, n8n |
| Job-alert e-mails | n8n via Gmail OAuth (one label, one dedicated mailbox) | any AI model |
| Run status (start / success / fail) | Healthchecks.io, as a ping without content | — |
| Server availability | UptimeRobot, by a check of the public readiness endpoint | — |

Further measures:

- The n8n instance is provided by a training provider. Narrow, replaceable credentials are used, the workflows are exported regularly, and a migration to a separate instance is planned before the course ends.
- IDs, chat IDs, Healthchecks ping addresses, the n8n host, the reference point, the candidate profile, the availability date and the salary expectations are removed from the workflows in this repository.
- Alert e-mails are contained in the saved executions on the n8n server. Saving of successful runs is switched off after the workflows have run stably for about a week.
- Nothing is sent to an employer by a workflow. Applications are sent manually.
- Use of data by Gemini (checked on 2026-11-08, [Gemini API Additional Terms](https://ai.google.dev/terms), effective 2026-03-23): for users in the EEA, the UK and Switzerland, the data terms of the paid service are applied to all services, including the free quota. Prompts and responses are not used to improve Google's products; they are logged for a limited time to detect policy violations. Nevertheless, only public ad text and an anonymous profile are sent.
