# Implementation Support Spec

**Document Identifier:** DW-T4-IS
**Tier:** 4 — Module Spec
**Location:** Google Drive (conceptual sections) + GitHub `docs/modules/` (engineering sections)
**Version:** 1.1
**Date:** May 2026
**Supersedes:** `Implementation_Support_Spec_v1.0`

**Architecture reference:** See `docs/SYSTEM_ARCHITECTURE_MASTER.md` for tier definitions and cross-reference rules.

**What this document owns:** The implementation support layer concept and scope. The initial delivery checklist spec — source data, format, and content. The SOP framework — authorship model, structure, catalog, and delivery. Customer system mapping — how Deepwalk outputs map to common customer execution environments. Output format specs (CSV and PDF). The relationship between the initial delivery checklist and the PE ongoing checklist export. The future library plan for SOPs and customer system format variants.

**What this document does not own:** The PE ongoing checklist export — that is defined in `10_Persistence_Engine_Spec` Part 7, which uses the same CSV format spec defined here. The Report Compiler finding sort order and card content — defined in `04_Report_Compiler_Spec`. The Stage 10 flat array output contract — defined in `docs/architecture/inference-engine-spec.md` (Tier 3). SOP content itself — owned by Third Switch methodology; this spec defines the structure and delivery model, not the content.

---

## Part 1 — Purpose and Concept

### 1.1 — What the Implementation Support Layer Is

The Implementation Support layer is the bridge between Deepwalk report delivery and customer execution. It takes the locked finding set from the Discovery Report and produces two artifacts designed for operational use in the customer's own systems and workflows:

1. **Implementation Checklist** — a structured, actionable list of findings organized for task assignment and tracking in customer systems
2. **Standard Operating Procedures (SOPs)** — Third Switch-authored execution protocols, one per measure category × action path combination, delivered as concise report appendix pages

Deepwalk facilitates the production and delivery of these artifacts. It does not own execution of the items they contain. The customer ingests them into their own CMMS, FM tools, work order platforms, or operational workflows.

### 1.2 — Two Delivery Moments

The Implementation Support layer has two delivery moments with different source data:

**Initial delivery** — at Discovery Report delivery. Source: Stage 10 flat array (locked findings from the current engagement). Produces the full checklist and SOP set for the engagement. Owned by this spec.

**Ongoing delivery** — via the Persistence Engine. Source: PE signal record (updated finding status over time). Produces an updated checklist reflecting current PE status. Owned by `10_Persistence_Engine_Spec` Part 7, which uses the CSV format defined in Part 5 of this document.

### 1.3 — Scope Boundary

Deepwalk produces the implementation artifacts. Deepwalk does not:

- Own or manage execution of checklist items
- Assign tasks in customer systems
- Track completion outside the PE action tracking layer
- Author or maintain customer-internal SOPs (though customer SOPs are collected opportunistically — see Part 4)

---

## Part 2 — Pipeline Position

The Implementation Support layer sits downstream of the Report Compiler and is triggered at report delivery.

```
Discovery Session
→ Inference Engine (Stages 1–10)
→ Report Compiler → Discovery Report
→ Implementation Support Layer
    → Implementation Checklist (CSV + PDF)
    → SOPs (report appendix)
    → Customer system mapping (Analyst-configured)
→ Customer execution systems (CMMS, FM tools, work order platforms)
→ Persistence Engine (ongoing — updated checklist via PE export)
```

The Implementation Support layer does not run until the Report Compiler has produced a locked report. It consumes the same Stage 10 flat array as the Report Compiler, filtered to `LOCKED` findings only.

---

## Part 3 — Implementation Checklist

### 3.1 — Purpose

The implementation checklist is a structured, system-ingestible list of all findings from the Discovery Report, organized for task assignment and tracking. It is the primary handoff artifact for customer execution teams.

### 3.2 — Source Data

The checklist is generated from the Stage 10 flat array, filtered to findings with `reconciliation_status = LOCKED`. Fields pulled directly from the pipeline — no manual data entry required for the structured fields.

### 3.3 — Checklist Content

Each row represents one finding:

| Field | Content | Source |
|-------|---------|--------|
| Finding ID | `finding_id` | Pipeline |
| Title | `title` | Pipeline |
| Measure category | `display_name` (`equipment_class` group) | Catalog |
| Domain | Human / Boundary / Mechanical | Pipeline |
| Action path | Green / Yellow / Red | Pipeline |
| Location — floor | `floor` | Pipeline |
| Location — zone | `zone` | Pipeline |
| Location — room | `space_id` | Pipeline |
| Fixture count | `fixture_count` | Pipeline |
| Est. annual savings | `s_financial` (USD) | Pipeline |
| Est. kWh reduction | `annual_energy_savings_kwh` | Pipeline |
| Confidence | `w_fixture_confidence` | Pipeline |
| SOP reference | SOP document name for this measure category × action path | Analyst-assigned |
| Owner | Blank — for customer completion | — |
| Status | Blank — for customer completion | — |
| Notes | Analyst-curated from `raw_notes` | Analyst |

### 3.4 — Sort Order

Matches the Identified Measures sort order from the Report Compiler: measure category ranked by aggregate annual savings value descending, then within each category by individual `s_financial` descending. This ensures the checklist and report are navigable together without re-sorting.

### 3.5 — Red Path Items

Findings with `assigned_action_path = Red` are included in the checklist. The SOP reference field for Red path items reads "Refer to capital planning process" rather than referencing an operational SOP. Red path findings represent asset actions — replacement, redesign, or engineering intervention — which are outside the scope of operational SOPs.

---

## Part 4 — Standard Operating Procedures

### 4.1 — Purpose

SOPs are Third Switch-authored execution protocols that tell the customer *how* to act on each finding type. Where the checklist tells the customer *what* to do, the SOP provides the step-by-step operational procedure for doing it.

SOPs are delivered as concise appendix pages within the Discovery Report. They are not standalone documents in V1.

### 4.2 — SOP Structure: One Per Measure Category × Action Path

Each SOP covers one measure category × action path combination where findings exist in the engagement. An engagement with Green and Yellow lighting findings produces two lighting SOPs. An engagement with Green plug load findings and Yellow plug load findings produces two plug load SOPs.

This matrix approach ensures:
- Each SOP is targeted — the execution protocol for a Green (immediate behavioral correction) is different from a Yellow (coordination with facilities or BAS)
- Red path items are handled separately (see Part 3.5) — they do not generate operational SOPs

**SOP naming convention:** `[Measure Category] — [Action Path] SOP`
Examples:
- `Lighting Shutdown Discipline — Green SOP`
- `Lighting Controls Alignment — Yellow SOP`
- `Plug Load Management — Green SOP`

### 4.3 — SOP Content Elements

Each SOP page contains:

- **Title** — measure category + action path
- **Applies to** — list of finding titles from the engagement that this SOP covers
- **Who acts** — role responsible for execution (e.g., facilities staff, BAS administrator, department manager)
- **Procedure** — numbered steps. Concise. Plain language. Maximum 8 steps in V1.
- **Verification** — how the responsible party confirms the action was completed (e.g., photo of powered-down equipment, BAS schedule screenshot)
- **Recurrence note** — what to do if the condition returns (brief — typically a reference to the PE web form submission process)
- **SOP version** — version number and date; enables catalog maintenance over time

### 4.4 — Delivery Format and Presentation

SOPs are delivered as appendix pages within the Discovery Report — specifically within or alongside the Identified Measures section. Each SOP page is one page maximum. The checklist references the SOP by name for each finding, creating a navigational link between the two artifacts.

**Presentation principle:** The SOP set should not feel overwhelming. Mitigations:
- Only SOPs relevant to findings in the current engagement are included — no generic library dump
- One page per SOP maximum
- SOPs are grouped by measure category in the appendix with a simple index
- The checklist SOP reference column allows the reader to navigate directly to the relevant SOP without reading the full set

### 4.5 — SOP Authorship Model

**Third Switch-authored:** The canonical SOP library is authored and maintained by Third Switch. SOPs are drawn from this internal library and selected for each engagement based on the finding set.

**Customer-authored SOPs:** Where a customer has their own existing SOPs for a finding type (e.g., a facilities team with an existing lighting shutdown protocol), Third Switch collects these opportunistically during the engagement. Collected customer SOPs are added to the internal catalog with attribution. They may be referenced or adapted in future engagements.

**Collection process (V1):** Manual. The Analyst or Facilitator requests existing customer SOPs during Scoping or the Discovery Session. No formal collection workflow is specified in V1.

### 4.6 — SOP Catalog and Future Library Plan

**V1:** Third Switch maintains an internal SOP library organized by measure category × action path. Selection and delivery are manual — the Analyst selects the relevant SOPs from the library and includes them in the report.

**Future capability (V2+):** A system for organizing and selecting language blocks from the SOP catalog — including both Third Switch-authored and customer-collected SOPs — by engagement type, measure category, action path, and customer system type. This capability enables:
- Rapid SOP assembly for new engagement types
- Customer-specific language variants
- Portfolio-level SOP consistency tracking
- Automated SOP selection based on finding set

This capability is not in scope for V1. It is noted here to ensure the V1 catalog is structured in a way that supports future automation — specifically, SOPs should be versioned, tagged by measure category and action path, and stored with attribution from first authorship.

---

## Part 5 — Output Format Spec

### 5.1 — CSV (Primary)

The CSV format is the primary export format for the implementation checklist. It is designed for direct ingestion into customer CMMS, FM tools, work order platforms, and project management systems.

**CSV structure:** One row per finding. Columns match the checklist content defined in Part 3.3. Column headers use plain English labels — not field names from the pipeline schema. This ensures the file is immediately usable by customer operations staff without technical translation.

**Encoding:** UTF-8. No special characters in field names.

**Blank fields:** Owner and Status fields are included as blank columns — the customer populates these in their own system. They are not omitted from the export.

**This format spec is canonical for both:**
- The initial delivery checklist (this spec)
- The PE ongoing checklist export (`10_Persistence_Engine_Spec` Part 7)

The PE export uses the same column structure with the addition of a `Current PE Status` column (`Resolved` / `Persisted` / `Remergent` / `Unverified`) and a `Last Signal Date` column. All other columns are identical.

### 5.2 — PDF (Optional)

A formatted PDF version of the checklist is available for engagements where the customer needs a human-readable deliverable rather than a system-ingestible file. The PDF presents the same content as the CSV in a formatted table layout consistent with the Discovery Report brand guidelines.

PDF generation is manual in V1 — the Analyst formats the checklist for PDF output. Automated PDF generation is a V2 candidate.

---

## Part 6 — Customer System Mapping

### 6.1 — Purpose

Different customers use different execution systems. The customer system mapping layer configures the checklist export to match the import requirements of the customer's target system, reducing the friction of ingestion.

### 6.2 — Supported System Types (V1)

In V1, customer system mapping is an Analyst-configured manual step — the Analyst reviews the customer's target system and adjusts the CSV column order, field labels, or field values to match the system's import template.

Common target systems encountered:

| System Type | Import Format | Common Mapping Notes |
|-------------|--------------|---------------------|
| CMMS (generic) | CSV with work order fields | Map finding title → work order description; action path → priority; location → asset location |
| FM platform | CSV or Excel | May require floor/zone split into separate columns; status values may need to match platform enum |
| Work order platform | CSV | Priority field often required — map Green → High, Yellow → Medium, Red → Low (or per customer convention) |
| Project management tool | CSV | May require date fields — leave blank or populate with engagement date |
| No target system | PDF checklist | Customer operates manually; PDF is the primary deliverable |

### 6.3 — Future System-Specific Format Library

As engagements accumulate, Third Switch will build a library of system-specific CSV templates — pre-configured column mappings for common CMMS and FM platforms. This library reduces Analyst configuration time and improves ingestion reliability.

In V1, this library does not exist. The Analyst performs mapping manually and documents the mapping for future reference. Documentation of manual mappings is the seed data for the future library.

---

## Part 7 — Open Items

| # | Item | Target |
|---|------|--------|
| 1 | SOP catalog structure — formal tagging schema for measure category, action path, version, and attribution to support future automated selection | Before first engagement using the catalog for a second time |
| 2 | Customer SOP collection process — formal workflow for requesting, storing, and attributing customer-authored SOPs | Before first engagement where customer SOPs are collected |
| 3 | Red path document type — capital planning brief or scope of work template for Red path findings; currently noted as "refer to capital planning process" in the checklist | Before first engagement with significant Red path findings |
| 4 | PDF checklist generation — V1 is manual; V2 candidate is automated PDF output from the same data source as CSV | Post first engagement |
| 5 | Customer system format library — pre-configured CSV templates for common CMMS and FM platforms | After 3+ engagements with distinct customer system types |
| 6 | SOP language block system — catalog-based assembly of SOPs from reusable language blocks, including customer-collected variants | V2+ — after sufficient SOP catalog accumulation |
| 7 | SOP maximum step count — 8 steps is a V1 constraint; review after first engagement delivery for readability | After first report delivery |

---

*Deep//Walk by Third Switch — Assurance at Scale*
*© 2026 Third Switch, LLC.*
*This document is internal reference. Not for external distribution without review.*
