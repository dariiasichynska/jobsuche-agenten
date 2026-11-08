# Changelog

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
