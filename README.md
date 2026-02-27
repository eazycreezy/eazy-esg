# Eazy ESG

**ESG reporting Assistant in Claude — GRI, TCFD, and ASRS frameworks.**

Eazy ESG in an open-source plugin that handles the full sustainability disclosure workflow: ingest your data, map it to the right framework, identify gaps, and draft compliant reports — all inside a Claude conversation. No external integrations, no API keys, no setup required.

---

## Supported Frameworks

| Framework | Coverage |
|---|---|
| **GRI** | Global Reporting Initiative — Universal + Topic Standards |
| **TCFD** | Task Force on Climate-related Financial Disclosures — all 4 pillars |
| **ASRS** | Australian Sustainability Reporting Standards — AASB S1 & S2 (aligned with IFRS ISSB) |

---

## What It Does

Eazy ESG routes your requests to a team of specialist agents based on what you're trying to accomplish:

| Agent | What it does |
|---|---|
| **Data Intake** | Ingests ESG data from spreadsheets, CSVs, or emails; validates and normalises all fields |
| **Framework Mapper** | Maps your data to GRI, TCFD, or ASRS disclosure requirements with specific indicator codes |
| **Gap Analyser** | Scores completeness across your disclosures and flags missing data with remediation guidance |
| **Report Writer** | Drafts board summaries, narrative sections, and structured disclosure content |
| **Compliance Monitor** | Surfaces the latest regulatory updates across GRI, TCFD, and ASRS |
| **Report Analyser** | Audits and scores an existing sustainability report against framework requirements |
| **Report Auditor** | Deep-checks draft reports for accuracy, consistency, and standard alignment |
| **Invoice Processor** | Extracts utility consumption data (electricity, gas, LPG, water, diesel) from PDF invoices into Excel |

---

## Usage

Install the plugin in Claude, then just describe what you need. Eazy ESG figures out which workflow to run:

```
"Here's our emissions spreadsheet for the year."
→ Data intake and field validation

"Map this to GRI."
→ Framework mapping with indicator codes

"What's missing from our TCFD disclosure?"
→ Gap analysis with completeness score and remediation steps

"Draft our Scope 1 and 2 narrative for the annual report."
→ Report writing grounded in your actual data

"What are the latest ASRS requirements?"
→ Compliance monitoring update

"Audit our existing sustainability report against GRI."
→ Report scoring and gap analysis

"Here are our electricity and gas invoices."
→ PDF extraction to structured Excel output
```

You can also chain steps in a single session:

```
Upload data → Map to framework → Identify gaps → Draft report sections
```

---

## Key Behaviours

- **Always asks which framework** (GRI / TCFD / ASRS) before mapping if you haven't specified
- **Never invents data** — missing metrics are marked `[DATA REQUIRED]`, never estimated
- **Cites specific indicator codes** in every output (e.g. GRI 305-1, TCFD Metrics & Targets, ASRS S2.29)
- **Flags gaps before drafting** — you'll know what data is missing before any report section is written
- **Multi-sector aware** — doesn't apply sector-specific materiality assumptions without confirmation

---

## Installation

1. Download or clone this repository
2. In the Claude desktop app, go to **Plugins** and select **Install from folder**
3. Point it at the root of this repo
4. The `eazy-esg` skill will appear automatically — start a new conversation and describe your ESG task

No environment variables or external accounts required.

---

## Project Structure

```
eazy-esg/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata
├── skills/
│   └── eazy-esg/
│       ├── SKILL.md             # Orchestrator — main entry point
│       ├── agents/
│       │   ├── data-intake-agent.md
│       │   ├── framework-mapper-agent.md
│       │   ├── gap-analyser-agent.md
│       │   ├── report-writer-agent.md
│       │   ├── compliance-monitor-agent.md
│       │   ├── report-analyser-agent.md
│       │   ├── report-auditor-agent.md
│       │   └── invoice-processor-agent.md
│       └── references/
│           ├── gri-indicators.md
│           ├── tcfd-pillars.md
│           ├── asrs-requirements.md
│           └── data-schema.md
└── README.md
```

---

## Contributing

Issues and pull requests are welcome. If you're working with a framework not yet supported (ESRS, SASB, CDP, etc.) or have ideas for additional agents, open an issue to discuss.

---

## License

MIT
