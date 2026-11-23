# TEACH Patient Questionnaire 2 Dashboard — Setup Guide (Live REDCap)

This is a copy of the TEACH dashboard (same tabs: Home, Recruitment Cascade,
Cascade Table, Enrolled by Sex, Enrolled by HIV Status, Missing Forms,
Missing TEACH Outcomes, Close-out) pointed at the **Patient Questionnaire 2**
REDCap project.

## Files

Put these in their **own folder** (e.g. `teach_q2_dashboard/`), separate from
the TEACH (Questionnaire 1) dashboard:

- `redcap_connect.R` — pulls all Questionnaire 2 records and builds every
  derived table (eligibility, cascade, missing forms, close-out).
- `app.R` — the dashboard.
- `.Renviron` — you create this (step 2). Holds the API token.

Do **not** copy the old `manifest.json` from the TEACH folder. It lists the
TEACH files and their checksums; rsconnect regenerates it for this folder when
you deploy.

## 1. Install packages (once)

```r
install.packages(c(
  "shiny", "bslib", "dplyr", "tidyr", "DT", "plotly",
  "lubridate", "stringr", "curl", "jsonlite"
))
```

## 2. Set the Questionnaire 2 API — REQUIRED

The URL/token written inside `redcap_connect.R` are **placeholders only**.
Questionnaire 2 is a different REDCap project, so it needs its own token:

1. In REDCap, open the **Patient Questionnaire 2** project → **API** →
   generate/copy your API token.
2. In the dashboard folder, create a plain-text file named `.Renviron`:

   ```
   TEACH2_REDCAP_URL=https://utirc.mak.ac.ug:8181/api/
   TEACH2_REDCAP_TOKEN=paste_the_questionnaire_2_token_here
   ```

   Change the URL too if Questionnaire 2 lives on a different REDCap server.
3. Restart R (Session → Restart R) so the file is read.

The variable names start with `TEACH2_` on purpose, so this token can never be
mixed up with the TEACH (Questionnaire 1) token. If the token isn't set, the
app still opens but shows no data and R prints a warning.

## 3. Run

```r
setwd("path/to/teach_q2_dashboard")
shiny::runApp("app.R")
```

## 4. What's different from the TEACH dashboard

| Item | TEACH (Q1) | Questionnaire 2 |
|---|---|---|
| Screening date | `screening_date` | `enroll_date` |
| Eligibility | REDCap `eligibility_status` | **Calculated**: 18+ years AND `transfer_in` = No AND `mdr` = No AND `distance_hospital` = Yes |
| 2nd follow-up SBQ | Month 5 | Month 6 (due day 180) |
| API token | `REDCAP_TOKEN` | `TEACH2_REDCAP_TOKEN` |

Eligibility rules: "No" as soon as any answered criterion fails; blank while
criteria are still unanswered. Age is 18+ unless `age_bracket` says "Less than
18 years". The original REDCap value is kept as `eligibility_status_redcap`.

## 5. Settings to check (top of `redcap_connect.R`)

If any of these don't match the Questionnaire 2 project exactly, the related
tab will be empty — change the value there:

- Event names: `EVENT_BASELINE`, `EVENT_FU1`, `EVENT_FU2`
  (currently `1. Baseline`, `2. Month_Two_followup`, `3. Month_Six_followup`)
- Repeating form names: `INSTR_LAB`, `INSTR_TB`, `INSTR_ART`
- `FU2_LABEL` / `FU2_DUE_DAY` (Month 6, day 180)
- Lab schedule (`LAB_SCHEDULE`) is still Month 2 and Month 5, as in TEACH.

## Security note

Never paste the token into chat, scripts or shared folders, and keep
`.Renviron` out of GitHub. The TEACH (Q1) `redcap_connect.R` still has a token
typed into it — consider regenerating that token in REDCap and moving it into
a `.Renviron` file as well.
