# Scoring

Matching is by **scope of tasks, not by job title**. Employers often give a role a title that says little about the work; a "Project Manager" ad can be a sales job, and a "Team Assistant" ad can be mostly process work.

## Weights

| Part | Range | What it measures |
| --- | --- | --- |
| Scope | 0–50 | overlap of the ad's tasks with real experience |
| Skills and tools | 0–20 | required tools and methods that are evidenced in the profile |
| Language | 0–10 | required German level vs. written C1 (C2 with tools) and spoken C1 |
| Location and format | 0–10 | office within commuting distance, or 100 % remote anywhere in Germany |
| Conditions | 0–10 | salary (if stated), contract, start date |

Threshold for the Writer: **75**.

## Penalties and bonus

Applied only when the ad states a requirement clearly, not for "nice to have" / "ideally".

| Situation | Points |
| --- | --- |
| Programming is the core of the job | −15 |
| Spoken German is a core duty (calls, trainings, presentations) | −15 to −25 |
| German as mother tongue / C2 explicitly required | −8 |
| Technical degree required without "or comparable experience" | −5 |
| SAP/ERP as core requirement (controlling roles) | −10 |
| Business or controlling degree mandatory | −5 |
| Core of the ad is reception, cashier, payroll, KYC, secretary work | −30 |
| Sales or acquisition is the main responsibility | scope capped at 15 |
| A must requirement from a missing class of tools or knowledge (SAP, programming, HGB …) | score capped at 70 |
| Internal role without customer contact | +5 |

"Very good", "fluent" or "business-fluent" German counts as C1 and costs nothing.

## Hard exclusions

- people management or team lead responsibility,
- "5+ years of professional experience in Germany" as a requirement,
- work with customers or on the phone as a core duty,
- a stated salary below my minimum (no salary stated = only an open question),
- on-site or hybrid work with the office outside commuting distance (about 7 km straight line, max. 30 minutes by public transport or bike).

An excluded job is saved with status `Nicht beworben`, so it is not found again, and the Writer is blocked.

## Must requirements and known tool classes

Since v1.4 the code node takes some items out of the model's list of missing must requirements before the cap is applied: degrees, years of experience, driving licence, languages, certificates, office software, and tools from a class I already use (another CRM, another task tracker such as Asana or Jira, another automation tool such as Power Automate). They remain normal gaps. What stays a must gap: SAP/ERP, programming languages, HGB/IFRS, construction law, DATEV, BI tools such as Power BI or Tableau. Gaps are listed only when the ad itself names the tool.

## Why the final score is computed in code

A score has to be reproducible and checkable. The model is good at reading an ad and explaining what fits; adding numbers and applying caps is a job for code. So the prompt asks only for parts, penalties with reasons and lists. The code node:

1. clamps every part to its range,
2. keeps only negative penalties, caps each at −30 and the sum at −60,
3. adds the +5 bonus,
4. clamps the result to 0–100,
5. writes the whole calculation into the record's notes.

Example note (illustrative numbers, real format):

```
Scope 40/50, Skills 15/20, Sprache 10/10, Ort 10/10, Bedingungen 7/10 = 82
Strafpunkte: -8 (C2 Deutsch gefordert)
Bonus interne Rolle: +5
Score: 79 (>= 75: Writer darf entwerfen)
Scorer v1.5, Modell gemini-3.1-flash-lite
```

## Tuning after the first real runs

| Observed | Change (v1.2) |
| --- | --- |
| A sales role titled "Project Manager" got scope 25/50 | sales as main responsibility → scope ≤ 15 |
| Fixed salary below minimum still got conditions 5/10 | conditions ≤ 3, note "salary below minimum" |
| −8 applied for "very good German" | penalty only for explicit C2 / mother tongue |
| A job in another city passed the location check | rule on the reference point and distance tightened; one case still open |

| Observed | Change (v1.4) |
| --- | --- |
| Ads asking for customer or phone work scored high | customer or phone work as a core duty = hard exclusion |
| One missing tool (another CRM) blocked good jobs | must cap only when a whole class is missing; known classes are normal gaps |
| "Start immediately" counted as a gap | no penalty, no gap, no open question |
| Gaps for tools the ad never mentioned | a gap is named only if the ad names the tool |

Planned for v1.6 (decided 13 Nov): no distance exclusion from the model, a "check address" flag instead; specialist knowledge (HGB/IFRS, construction law) caps the score, industry experience is only a minus.
