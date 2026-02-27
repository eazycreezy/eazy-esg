# Report Analyser Agent

**Purpose:** Ingest an existing sustainability report (PDF, Word, or text), analyse it against
GRI, TCFD, and/or ASRS framework requirements, score completeness and quality per indicator,
identify gaps, and produce two outputs:
1. A formatted PDF assessment report for stakeholder review
2. An Excel gap analysis workbook for detailed remediation tracking

---

## Trigger Conditions
- User uploads an existing sustainability / ESG / climate report for review
- User says "analyse my report", "check my disclosure", "how does this stack up against ASRS",
  "score my sustainability report", "what am I missing from my report"
- User wants to know their compliance status before a filing deadline

---

## Workflow

### Step 1: Intake & Confirm Framework

Acknowledge the uploaded document and confirm scope:

> "I've received your sustainability report. Before I analyse it, a couple of quick questions:
> 1. Which framework(s) should I assess against? (GRI / TCFD / ASRS — or all three?)
> 2. What is the reporting period covered?
> 3. Is this a draft or a published report?"

If the user has already specified the framework, skip the question and proceed.

### Step 2: Extract Report Content

Use `pdfplumber` (if PDF) or direct text extraction (if Word/text) to extract the full report content.
Organise extracted content by section where possible.

Look for standard ESG report structure:
- Executive summary / CEO message
- Governance section
- Environment / Climate section
- Social / People section
- Appendices / GRI/TCFD index
- Data tables / performance summary

### Step 3: Load Framework Requirements

Load the relevant reference files:
- `references/asrs-requirements.md` — for ASRS assessment
- `references/tcfd-pillars.md` — for TCFD assessment
- `references/gri-indicators.md` — for GRI assessment

### Step 4: Indicator-by-Indicator Assessment

For each applicable framework indicator, assess the report against three dimensions:

#### Completeness Score (0–3)
| Score | Meaning |
|---|---|
| 0 | Not addressed — no relevant content found |
| 1 | Mentioned — topic acknowledged but no substantive disclosure |
| 2 | Partial — some disclosure present but incomplete vs. requirement |
| 3 | Full — disclosure appears to meet the requirement |

#### Quality Score (0–3)
| Score | Meaning |
|---|---|
| 0 | N/A (no content to assess) |
| 1 | Low — vague, generic, or boilerplate language |
| 2 | Medium — specific to the entity but lacks depth or supporting data |
| 3 | High — specific, quantified, with methodology and context |

#### Evidence Found
Note the specific page/section reference and a brief quote or description of what was found
(or "Not found" if absent). Never fabricate evidence.

### Step 5: Calculate Scores

**Per indicator:** Combined Score = (Completeness + Quality) / 6 × 100%

**Per framework pillar/section:**
Pillar Score = average of indicator scores within that pillar

**Overall Framework Score:**
Overall = weighted average of pillar scores (equal weighting unless sector-specific)

**RAG Status per indicator:**
- 🟢 Green: Combined score ≥ 67% (score 4–6)
- 🟡 Amber: Combined score 34–66% (score 2–3)
- 🔴 Red: Combined score ≤ 33% (score 0–1)

### Step 6: Generate PDF Assessment Report

Use `reportlab` to produce a professional PDF with the following structure:

---

**PDF Structure:**

**Cover Page**
- Title: "ESG Report Assessment — [Framework] Compliance Review"
- Entity name, Reporting period, Date of assessment
- Prepared by: Eazy ESG

**Page 2: Executive Summary**
- Overall scores per framework (visual scorecard)
- Top 3 strengths (highest scoring indicators)
- Top 3 critical gaps (mandatory indicators scoring 0–1)
- Recommended immediate actions (numbered list)

**Page 3+: Framework Assessment — Section by Section**
For each pillar/section:
- Pillar score + RAG status
- Table: Indicator | Requirement | Evidence Found | Completeness | Quality | RAG
- Brief narrative on overall pillar strength and key gaps

**Final Page: Gap Summary & Recommended Actions**
- Prioritised list of gaps (mandatory first, then conditional, then recommended)
- Suggested remediation for each gap
- Disclaimer: "This assessment is AI-generated and should be reviewed by a qualified ESG professional before use in regulatory filings."

---

**PDF Formatting Requirements (reportlab):**
- Page size: A4
- Colours: Dark green header (#1B4332), white text on headers, light green row alternates (#D1FAE5)
- Font: Helvetica throughout
- RAG indicators: filled coloured circles (●) — green #16A34A, amber #D97706, red #DC2626
- Cover page: solid dark green background with white title text
- Never use Unicode subscript characters — use ReportLab `<sub>` tags for CO₂e

### Step 7: Generate Excel Gap Analysis Workbook

Use `openpyxl` to produce a multi-sheet workbook:

**Sheet 1: Dashboard**
- Entity, period, assessment date
- Scorecard table: Framework | Indicators Assessed | Green | Amber | Red | Overall Score %
- Conditional formatting on score cells

**Sheet 2: Full Gap Analysis**
Columns:
| Framework | Pillar | Indicator Code | Indicator Name | Obligation | Completeness (0-3) | Quality (0-3) | Combined Score | RAG Status | Evidence / Page Ref | Gap Description | Recommended Action | Priority | Estimated Effort | Owner |

- Freeze top row
- Auto-filter on all columns
- Conditional formatting: RAG Status column coloured by value
- Sort: Mandatory gaps first, then by score ascending (worst first)

**Sheet 3: Remediation Tracker**
Pre-populated from gap analysis, designed for ongoing tracking:
Columns:
| # | Indicator | Gap Description | Action Required | Priority | Assigned To | Target Date | Status | Notes |

- Status dropdown: Not Started / In Progress / Under Review / Complete
- Colour-coded status column

**Sheet 4: Score Summary**
- Pivot-style summary by framework and pillar
- Bar chart: overall scores per framework
- Trend column (blank — for user to populate next period)

### Step 8: Save & Present Outputs

Save files as:
- `[EntityName]_ESG_Assessment_[Framework]_[Period].pdf`
- `[EntityName]_ESG_GapAnalysis_[Period].xlsx`

If entity name unknown, use `ESG_Report`.

Run `scripts/recalc.py` on the Excel file to validate formulas before presenting.

Present both files to the user with a brief summary:

> "Assessment complete. Here's what I found:
> - **Overall ASRS readiness: [X]%** ([n] green / [n] amber / [n] red indicators)
> - **Biggest gaps:** [top 3 gaps in one sentence each]
> - **Immediate priorities:** [1–2 actions before next deadline]
>
> Your PDF report and Excel gap tracker are ready for download."

---

## Quality & Accuracy Rules

- **Evidence-based only** — only score what is actually present in the report. Do not give credit
  for content that is generic, boilerplate, or not specific to the entity.
- **Conservative scoring** — when in doubt between two scores, use the lower score. It is better
  to flag a potential gap than to miss one.
- **Page references mandatory** — every scored indicator must include a page/section reference or
  "Not found".
- **No hallucinated quotes** — if quoting from the report, use exact text. If paraphrasing, note
  "paraphrased".
- **Mandatory vs optional** — always distinguish mandatory ASRS requirements from recommended TCFD
  disclosures and chosen GRI Topic Standards.
- **Disclaimer always present** — the PDF must always include the AI assessment disclaimer.
  This output is a decision-support tool, not a legal compliance determination.

---

## Error Handling

- If PDF is scanned/image-based and text extraction fails: notify user, suggest they provide
  a text-searchable version or paste key sections
- If report covers multiple entities: ask user to confirm primary reporting entity
- If reporting period unclear: ask before scoring — period affects which ASRS group obligations apply
- If report is very short (<5 pages): flag that it may be a summary, not a full disclosure report,
  and ask if a more complete version is available
