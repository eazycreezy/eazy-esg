# Report Auditor Agent

**Purpose:** Analyse an existing sustainability report (PDF or Word) against GRI, TCFD, and/or
ASRS framework requirements. Produce a completeness and quality score per framework pillar,
identify gaps, and output two deliverables:
1. A formatted **PDF audit report** with scores, findings, and recommendations
2. An **Excel gap analysis workbook** with one sheet per framework, filterable by status and priority

---

## Trigger Conditions
- User uploads or references an existing sustainability report
- User asks: "audit my report", "check my ESG report against ASRS", "score my sustainability disclosure",
  "what's missing from our report", "run a compliance check on this"

---

## Workflow

### Step 1: Intake & Confirm Scope

When the user provides a report, confirm:
1. **Framework(s) to audit against** — GRI / TCFD / ASRS / All three
2. **Reporting period** — if not evident from the document
3. **Entity name** — if not evident
4. **Strictness level:**
   - **Mandatory only** — assess only legally required disclosures
   - **Full** — assess all recommended/best practice disclosures too

If the file is a PDF, use `pdfplumber` to extract text. If Word, use `python-docx`.

### Step 2: Extract Report Content

Parse the uploaded report and build an internal **content map**:

```python
# For PDF extraction:
import pdfplumber

with pdfplumber.open(report_path) as pdf:
    full_text = ""
    for page in pdf.pages:
        full_text += page.extract_text() or ""

# Chunk into sections by detecting headings
# Map sections to framework topics for analysis
```

Build a section index:
```
SECTION INDEX
──────────────────────────────────────
p.1–3    | Executive Summary / CEO Letter
p.4–8    | About This Report / Reporting Boundary
p.9–15   | Governance
p.16–24  | Environment / Climate
p.25–31  | Social / Workforce
p.32–36  | GRI Index / TCFD Index / Appendices
──────────────────────────────────────
```

### Step 3: Load Framework Requirements

Load the relevant reference files and build a **master checklist** of all indicators/disclosures
to assess:
- GRI → `references/gri-indicators.md`
- TCFD → `references/tcfd-pillars.md`
- ASRS → `references/asrs-requirements.md`

### Step 4: Assess Each Indicator

For each indicator in the checklist, analyse the extracted text and assign:

**Completeness Score (0–3):**
| Score | Label | Criteria |
|---|---|---|
| 3 | ✅ Fully Addressed | Indicator clearly and completely addressed with data/narrative as required |
| 2 | ⚠️ Partially Addressed | Indicator mentioned but incomplete (missing data, vague, no methodology) |
| 1 | 🔵 Referenced Only | Topic mentioned but no substantive disclosure |
| 0 | ❌ Not Found | No evidence of disclosure in report |

**Quality Score (0–3) — where disclosure exists:**
| Score | Label | Criteria |
|---|---|---|
| 3 | High | Specific, quantified, methodologically sound, comparable to prior period |
| 2 | Medium | Substantive but lacks specificity, comparatives, or methodology detail |
| 1 | Low | Present but qualitative only, vague, or boilerplate language |
| 0 | N/A | No disclosure to assess |

**Evidence:** Note the page number / section where the disclosure was found (or not found).

### Step 5: Calculate Scores

For each framework, calculate aggregate scores:

```
FRAMEWORK SCORECARD — ASRS S2

Pillar               | Indicators | Fully Met | Partial | Referenced | Missing | Score
─────────────────────────────────────────────────────────────────────────────────────────
Governance           |     4      |     2     |    1    |     0      |    1    | 58%
Strategy             |     6      |     1     |    2    |     1      |    2    | 33%
Risk Management      |     3      |     2     |    1    |     0      |    0    | 78%
Metrics & Targets    |     9      |     3     |    2    |     1      |    3    | 44%
─────────────────────────────────────────────────────────────────────────────────────────
TOTAL                |    22      |     8     |    6    |     2      |    6    | 50%

Overall Completeness Score: 50% 🟡
Overall Quality Score:      61% 🟡
Combined Audit Score:       55% 🟡
```

Ratings:
- 🟢 80%+ = Strong
- 🟡 50–79% = Developing
- 🔴 <50% = Significant gaps

### Step 6: Generate Excel Gap Analysis Workbook

Create a multi-sheet Excel workbook using openpyxl:

**Sheet 1: Summary Dashboard**
- Overall scores per framework (with colour-coded RAG status)
- Bar chart showing completeness by pillar
- Top 5 priority gaps

**Sheet 2: ASRS Gap Analysis** (and/or GRI, TCFD)
Columns:
| Column | Content |
|---|---|
| Framework | ASRS S2 |
| Pillar | e.g. Strategy |
| Indicator Code | e.g. S2.22 |
| Indicator Name | e.g. Climate scenario analysis |
| Obligation | Mandatory / Conditional / Recommended |
| Completeness Score | 0–3 |
| Completeness Label | ✅ Fully / ⚠️ Partial / 🔵 Referenced / ❌ Missing |
| Quality Score | 0–3 |
| Quality Label | High / Medium / Low / N/A |
| Evidence Location | e.g. "p.18 Climate Risk section" |
| Gap Description | Specific description of what's missing or weak |
| Recommended Action | What needs to be done to remediate |
| Effort | High / Medium / Low |
| Priority | 1 (Critical) / 2 (Important) / 3 (Nice to have) |
| Status | Open / In Progress / Closed |
| Owner | [blank — for user to fill] |
| Target Date | [blank — for user to fill] |

Apply formatting:
- Header row: dark green background, white bold text
- Completeness 0 (❌ Missing): light red fill
- Completeness 1 (🔵 Referenced): light yellow fill
- Completeness 2 (⚠️ Partial): light orange fill
- Completeness 3 (✅ Full): light green fill
- Auto-filter on all columns
- Freeze top row
- Column widths optimised for readability

**Sheet 3: Remediation Workplan**
Pre-populated with all gaps (Priority 1 & 2), sorted by priority then effort (low effort first).
Columns: Priority | Gap | Framework | Indicator | Action | Effort | Owner | Target Date | Status

After creating the workbook, run recalculation:
```bash
python /mnt/skills/public/xlsx/scripts/recalc.py output.xlsx
```

### Step 7: Generate PDF Audit Report

Use reportlab to produce a professional PDF with these sections:

**Cover Page:**
- "ESG Compliance Audit Report"
- Entity name, reporting period, audit date
- Prepared by: Eazy ESG
- Frameworks assessed
- DRAFT watermark if data gaps remain

**Section 1: Executive Summary (1 page)**
- Overall audit findings in plain language
- Combined score with RAG rating
- Top 3 strengths
- Top 3 critical gaps
- Recommended immediate actions

**Section 2: Scorecard (1 page)**
- Table: Framework × Pillar scores
- Visual score bars

**Section 3: Framework-by-Framework Findings**
For each framework assessed:
- Pillar-level narrative summary
- Key strengths (what's done well)
- Key gaps (what's missing or weak)
- Specific indicator findings (brief, grouped by pillar)

**Section 4: Gap Analysis Summary**
- Table of all gaps sorted by priority
- Cross-referenced to Excel workbook

**Section 5: Recommended Next Steps**
- Prioritised action list
- Note that full gap detail is in the Excel workbook

**Appendix: Methodology**
- Scoring rubric explanation
- Frameworks and versions used
- Limitations of automated analysis

#### ReportLab implementation notes:
- Use A4 pagesize
- Use `SimpleDocTemplate` with Platypus for flow
- Colour palette: Dark green (#1B5E20), medium green (#388E3C), light green (#C8E6C9),
  amber (#FFA000), red (#C62828), grey (#546E7A)
- Use `Table` with `TableStyle` for scorecards
- Never use Unicode subscript characters — use `<sub>` tags in Paragraph objects
- Page numbers in footer

### Step 8: Save and Present Outputs

Save both files with consistent naming:
```
[EntityName]_ESG_Audit_[Framework]_[Period].pdf
[EntityName]_ESG_Audit_[Framework]_[Period]_GapAnalysis.xlsx
```

Copy to `/mnt/user-data/outputs/` and present both files to the user.

### Step 9: Summary in Chat

After presenting files, provide a brief chat summary:

> "**Audit complete.** Here's what I found:
>
> 📊 **Overall Score: [X]% [🟢/🟡/🔴]**
> - ASRS S2: [X]% | TCFD: [X]% | GRI: [X]%
>
> 🔴 **[n] critical gaps** require attention before filing
> 🟡 **[n] partial disclosures** need strengthening
> ✅ **[n] indicators** fully addressed
>
> Your two outputs:
> - PDF audit report with findings and recommendations
> - Excel workbook with full gap analysis — filter by Priority or Status to build your action plan"

---

## Quality & Integrity Rules

- **Never fabricate evidence** — only mark an indicator as addressed if text evidence exists
- **Conservative scoring** — when in doubt, score lower and explain why in Gap Description
- **Cite page numbers** — always reference where (or where not) a disclosure was found
- **Distinguish absence from silence** — note if an indicator may have been excluded intentionally
  (e.g. entity considers it immaterial) vs simply omitted
- **Automated analysis disclaimer** — always note in both outputs that this is an AI-assisted
  analysis and should be reviewed by a qualified sustainability professional before use
