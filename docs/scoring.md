# Scoring

Jobs are matched by **scope of tasks, not by job title**. Job titles often say little about the work: a "Project Manager" ad can describe a sales job, and a "Team Assistant" ad can describe mostly process work.

## Weights

| Part | Range | Measured value |
| --- | --- | --- |
| Scope | 0–50 | overlap of the ad's tasks with evidenced experience |
| Skills and tools | 0–20 | required tools and methods that are evidenced in the profile |
| Language | 0–10 | required German level compared with written C1 (C2 with tools) and spoken C1 |
| Location and format | 0–10 | office within the commuting radius, or 100 % remote anywhere in Germany |
| Conditions | 0–10 | salary (if stated), contract, start date |

Threshold for the Writer: **75**.

## Penalties and bonus

Penalties are applied only when a requirement is stated clearly, not for "nice to have" or "ideally".

| Situation | Points |
| --- | --- |
| Programming is the core of the job | −15 |
| Spoken German is a core duty (calls, trainings, presentations) | −15 to −25 |
| German as mother tongue or C2 is explicitly required | −8 |
| A technical degree is required without "or comparable experience" | −5 |
| SAP/ERP is a core requirement (controlling roles) | −10 |
| A business or controlling degree is mandatory | −5 |
| The core of the ad is reception, cashier, payroll, KYC or secretarial work | −30 |
| Sales or acquisition is the main responsibility | scope capped at 15 |
| A must requirement from a missing class of tools or knowledge (SAP, programming, HGB …) | score capped at 70 |
| Internal role without customer contact | +5 |

"Very good", "fluent" or "business-fluent" German is counted as C1 and costs no points.

## Hard exclusions

- people management or team lead responsibility;
- "5+ years of professional experience in Germany" as a requirement;
- work with customers or on the phone as a core duty;
- a stated salary below the configured minimum (if no salary is stated, only an open question is added);
- on-site or hybrid work with the office outside the commuting radius (about 7 km straight line, at most 30 minutes by public transport or bike).

An excluded job is saved with the status `Nicht beworben`, so that it is not found again, and the Writer is blocked.

## Must requirements and known tool classes

Since v1.4, some items are removed in the code node from the model's list of missing must requirements before the cap is applied: degrees, years of experience, driving licence, languages, certificates, office software, and tools from a class that is already used by the candidate (another CRM, another task tracker such as Asana or Jira, another automation tool such as Power Automate). These items are kept as normal gaps. SAP/ERP, programming languages, HGB/IFRS, construction law, DATEV and BI tools such as Power BI or Tableau are kept as must gaps. A gap is listed only if the tool is named in the ad.

## Score computed in code

A score has to be reproducible and checkable. Reading an ad and explaining the fit is left to the model; arithmetic and caps are handled in code. Therefore only parts, penalties with reasons and lists are requested in the prompt. In the code node:

1. every part is clamped to its range;
2. only negative penalties are kept, each is capped at −30 and the sum at −60;
3. the +5 bonus is added;
4. the result is clamped to 0–100;
5. the whole calculation is written to the notes of the record.

Example note (illustrative numbers, real format):

```
Scope 40/50, Skills 15/20, Sprache 10/10, Ort 10/10, Bedingungen 7/10 = 82
Strafpunkte: -8 (C2 Deutsch gefordert)
Bonus interne Rolle: +5
Score: 79 (>= 75: Writer darf entwerfen)
Scorer v1.5, Modell gemini-3.1-flash-lite
```

## Tuning after the first real runs

| Observation | Change (v1.2) |
| --- | --- |
| A sales role titled "Project Manager" was given scope 25/50. | Sales as the main responsibility caps scope at 15. |
| A fixed salary below the minimum was still given conditions 5/10. | Conditions are capped at 3 with the note "salary below minimum". |
| −8 was applied for "very good German". | The penalty is applied only for explicit C2 or mother tongue. |
| A job in another city passed the location check. | The rule on the reference point and distance was tightened; one case is still open. |

| Observation | Change (v1.4) |
| --- | --- |
| Ads with customer or phone work were scored high. | Customer or phone work as a core duty is a hard exclusion. |
| Good jobs were blocked by one missing tool (another CRM). | The must cap is applied only if a whole class is missing; known classes are normal gaps. |
| "Start immediately" was counted as a gap. | No penalty, no gap and no open question are added. |
| Gaps were listed for tools that were not named in the ad. | A gap is listed only if the tool is named in the ad. |

Planned for v1.6 (decided on 13 Nov): the distance exclusion by the model is replaced with a "check address" flag; missing specialist knowledge (HGB/IFRS, construction law) caps the score, missing industry experience is counted only as a minus.
