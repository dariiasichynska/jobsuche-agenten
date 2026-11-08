# Privacy by design

| Data | Where it goes | Never goes to |
| --- | --- | --- |
| Public job ad text | Gemini (Scorer) | — |
| Short profile: experience, tools, languages, no name, no contact data | Gemini (Scorer) | employers, public repo |
| Search keywords, company names | Gemini (Employer-Scout) | — |
| Reference point for the commute rule | coordinates in code nodes; the Scorer prompt names only a nearby public square | — (no home address anywhere) |
| Full CV, contact data, application texts | Claude (Writer), Google Drive | Gemini, n8n |
| Job-alert e-mails | n8n via Gmail OAuth (one label, one dedicated mailbox) | any AI model |

Further measures:

- The n8n instance belongs to a training provider. Credentials are narrow and replaceable, workflows are exported regularly, and a move to my own instance is planned before the course ends.
- Workflows in this repository have IDs, chat IDs, the reference point, the candidate profile and salary expectations removed.
- No workflow sends anything to an employer. Applications are sent by me.
- Gemini data use (checked 2026-11-08, [Gemini API Additional Terms](https://ai.google.dev/terms), effective 2026-03-23): for users in the EEA, the UK and Switzerland the data terms of the paid service apply to all services, including the free quota. Prompts and responses are not used to improve Google's products; they are logged for a limited time to detect policy violations. Even so, only public ad text and an anonymous profile are sent.
