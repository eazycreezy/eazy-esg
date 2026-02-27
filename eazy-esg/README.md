# Eazy ESG Plugin

Comprehensive ESG management assistant for Corporate ESG Managers. Covers the full sustainability reporting workflow — from ingesting raw data to drafting compliant disclosures across GRI, TCFD, and ASRS frameworks.

## Overview

Eazy ESG acts as a knowledgeable ESG colleague: approachable, precise, and always focused on getting disclosures right. It routes requests to the appropriate specialist workflow based on what you're trying to accomplish.

**Supported Frameworks:**
- GRI (Global Reporting Initiative) — Universal + Topic Standards
- TCFD (Task Force on Climate-related Financial Disclosures) — 4 pillars
- ASRS (Australian Sustainability Reporting Standards) — AASB S1 & S2, aligned with IFRS ISSB

## Components

### Skill: eazy-esg

The main entry point and orchestrator. Triggers automatically when you mention ESG-related topics and routes to the right internal workflow.

**Internal Workflows:**

| Workflow | What it does |
|----------|-------------|
| Data Intake | Ingests ESG data from spreadsheets, CSVs, or emails; validates and normalises fields |
| Framework Mapper | Maps your data to GRI, TCFD, or ASRS disclosure requirements |
| Gap Analyser | Scores completeness and flags missing disclosures with remediation guidance |
| Report Writer | Drafts disclosure sections, board summaries, and narrative statements |
| Compliance Monitor | Tracks the latest regulatory requirements and updates for ASRS, TCFD, and GRI |
| Report Analyser | Audits and scores an existing sustainability report against framework requirements |
| Report Auditor | Deep-checks draft reports for accuracy, consistency, and standard alignment |

**Reference files loaded on demand:**
- GRI indicators (Universal + common Topic Standards)
- TCFD 4-pillar structure with recommended disclosures
- ASRS key disclosure requirements (AASB S1 & S2)
- Standard ESG data field schema for intake validation

## Setup

No external integrations or environment variables required. The plugin works entirely within your conversation.

## Usage

Just describe what you need — Eazy ESG will figure out which workflow to run:

| What you say | What happens |
|---|---|
| "Here's our emissions spreadsheet" | Data intake and validation |
| "Map this to GRI" | Framework mapping workflow |
| "What's missing from our disclosure?" | Gap analysis with scoring |
| "Draft our TCFD report section" | Report writing workflow |
| "What are the latest ASRS requirements?" | Compliance monitoring |
| "Audit our existing sustainability report" | Report analysis and scoring |

You can also chain steps: upload data → map to framework → identify gaps → draft report.

**Key behaviours:**
- Always asks which framework (GRI/TCFD/ASRS) if you haven't specified
- Marks missing data as `[DATA REQUIRED]` — never estimates or invents metrics
- Cites specific indicator codes (e.g. GRI 305-1, TCFD Metrics & Targets, ASRS S2.29)
- Flags gaps early before proceeding to drafting
