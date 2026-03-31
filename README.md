# Eligibility Assessment Evaluator

A single-file browser app for evaluating genetic risk assessment engine logic against test cases and clinical guidelines. No installation or server required — open `index.html` directly in any modern browser.

**Live:** https://cassiehajek.github.io/eligibility-evaluator/

---

## What It Does

Given a set of test cases (XLSX), intended outcomes, and clinical guideline references, the app:

1. Runs each case through a genetic eligibility engine
2. Compares the tool outcome to the intended outcome (criteria match)
3. Compares the tool outcome to clinical guidelines (guidelines match)
4. Displays an interactive results table with filtering, detail view, and export
5. Generates a written summary of the evaluation including known engine gaps
6. Saves evaluations to browser storage so prior runs can be reviewed later

---

## Getting Started

### Option A — Use the live app
Visit https://cassiehajek.github.io/eligibility-evaluator/ and upload your files.

### Option B — Run locally
Download `index.html` and open it in any browser. No server needed.

---

## Inputs

### Engine source (required for custom engines)
| Field | Description |
|---|---|
| GitHub Repo URL | e.g. `https://github.com/org/patient-ui` |
| GitHub Token | Required for private repos (`ghp_...`) |
| Engine file path | Path to the `.ts` engine file within the repo |

Click **Fetch Engine from GitHub** to load. If no repo is provided, the app uses the **bundled fallback engine** (Helix `cancerEligibilityEngine`).

### Test Cases XLSX
Upload a spreadsheet with one row per scenario. Expected columns:

| Col | Field |
|---|---|
| A | Scenario ID (e.g. `q3_breast_le50`) |
| B | Input description |
| C | Rule category |
| D | Intended outcome (`ELIGIBLE` / `INELIGIBLE` / `REQUIRES_REVIEW`) |
| E | MA Logic Validated (`1` = validated, `0` = known gap) |
| H | Notes |

### Criteria XLSX (optional)
A decision matrix mapping conditions to outcomes. Used for criteria-match comparison. Expected layout matches the standard NCCN criteria format with outcome in column AB, NCCN reference in AC, and guidelines excerpt in AD.

### Guidelines (optional)
Upload one or more PDF guideline documents, or type in a reference (e.g. `NCCN HBOC v2.2024`) and click **+ Add**. All references are listed in the evaluation summary and saved with each evaluation record.

---

## Running an Evaluation

1. Upload test cases XLSX
2. (Optional) Upload criteria XLSX and/or guidelines
3. Click **▶ Run Evaluation**

Results appear immediately. Click any row to see full detail in the right panel.

---

## Results Table

| Column | Description |
|---|---|
| Scenario ID | Unique test case identifier |
| Rule Category | e.g. Personal History – HBOC |
| Tool Outcome | Engine result: `ELIGIBLE` / `INELIGIBLE` / `REQUIRES_REVIEW` |
| Intended | Expected outcome from test cases file |
| Criteria | Whether tool matches intended outcome |
| Guidelines | Whether tool is consistent with clinical guidelines |

Color coding: green = ELIGIBLE / match, gray = INELIGIBLE, yellow = REQUIRES_REVIEW, red = mismatch.

### Filters
- Free-text search (scenario ID or description)
- Outcome, Criteria match, Guidelines match dropdowns
- Rule category dropdown

---

## Evaluation Summary

After each run a summary panel shows:
- Run date/time and engine used
- Guidelines validated against (PDFs and/or text references)
- Criteria match and guidelines consistency rates
- All criteria mismatches with tool vs. expected outcome
- Known engine gaps vs. guidelines, grouped by case
- Per-category breakdown

---

## Saving & Loading Evaluations

- **💾 Save** — prompts for a name and saves the full run to browser localStorage
- **+ New** — clears the current workspace (saved evaluations are preserved)
- Saved runs appear in the **Saved Evaluations** panel on the left; click any to restore

Saved evaluations persist across browser sessions. They are stored in the browser only — not synced to any server.

---

## Export

**⬇ Export CSV** downloads the current (filtered) results with columns:

`ScenarioID, Input Description, Rule Category, Tool Outcome, Intended Outcome, Consistent with Criteria, Consistent with Guidelines, Guidelines Note, MA Logic Validated`

---

## Bundled Engine

When no GitHub repo is provided, the app uses a JavaScript port of the Helix `cancerEligibilityEngine`. This engine evaluates hereditary cancer eligibility across:

- HBOC (Breast/Ovarian) — BRCA1/2 criteria
- Lynch Syndrome — LS-related cancer criteria
- Li-Fraumeni Syndrome (LFS)
- Hereditary Diffuse Gastric Cancer (HDGC)
- MEN1 / MEN2
- Renal Cell Carcinoma
- Melanoma
- Hematologic malignancies (AML, MDS)
- Polyposis syndromes
- Ashkenazi Jewish ancestry combinations

**Known engine gaps** (cases where the engine diverges from NCCN guidelines):
- FH-only MEN1 rule not implemented (pituitary adenoma, PHPT, multigland hyperplasia, duodenal NET, foregut carcinoid)
- Meningioma not included in LFS tumor spectrum
- Single melanoma + FH melanoma with ≥2 non-skin cancers not handled
- Cowden/PTEN criteria not implemented

---

## GitHub Fetch & TypeScript Support

When a GitHub repo and engine path are provided, the app:
1. Fetches the `.ts` engine file via the GitHub API
2. Also attempts to fetch `screeningTypes.ts` and `cancerSearchData.ts` if present
3. Transforms TypeScript to runnable JavaScript (strips types, interfaces, imports)
4. Executes the engine in-browser and uses it for all evaluations

This allows any team to point the app at their own eligibility engine repo without modifying the app itself.

---

## Tech Stack

- HTML/CSS/JS — single file, no build step
- [Tailwind CSS](https://tailwindcss.com/) via CDN
- [SheetJS](https://sheetjs.com/) via CDN for XLSX parsing
- Browser `localStorage` for saved evaluations
- GitHub REST API for engine fetching
