# 04 — Report Compiler Spec

**Document Identifier:** DW-T4-04
**Tier:** 4 — Module Spec
**Location:** Google Drive (conceptual sections) + GitHub `docs/modules/` (engineering sections)
**Version:** 1.0
**Date:** May 2026

**Architecture reference:** See `docs/SYSTEM_ARCHITECTURE_MASTER.md` for tier definitions and cross-reference rules.

**What this document owns:** The full report section specification — what each section contains, what data it draws from, and rendering order. The pipeline that consumes the Stage 10 flat array output and renders it into the Deepwalk Discovery Report. The Opportunity Map dual-axis grid UI spec as a rendered report element. The Opportunity Value Map UI spec. The Identified Measures section sort logic and finding card spec. Photo selection and rendering rules. P4 language compliance rules as applied to narrative generation. The Inference Summary / Table of Engineering Assumptions section spec. The appendix section specs.

**What this document does not own:** P4 itself → Studio Design v3.4 (Tier 2). Domain and action path definitions → Studio Design v3.4 (Tier 2). The Stage 10 flat array output contract → `docs/architecture/inference-engine-spec.md` (Tier 3). Measure Catalog wattage assumptions and version lifecycle → `03_Measure_Catalog_Spec` (Tier 4). The Review Workflow and finding exclusion rules → `07_Review_Workflow_Spec` (Tier 2). The Persistence Engine report output → `10_Persistence_Engine_Spec` (Tier 4).

---

## Part 1 — Purpose and Pipeline Position

### 1.1 — What the Report Compiler Does

The Report Compiler consumes the Stage 10 flat array output from the Inference Engine and renders it into the Deepwalk Discovery Report — the primary customer-facing deliverable of a Deepwalk engagement. It is the final transformation step between structured pipeline data and a human-readable document.

The Report Compiler does not perform inference, classification, or savings calculation. All findings, domain assignments, action path assignments, savings values, and confidence scores arrive from the Inference Engine as locked values. The Report Compiler renders these values into the report structure defined in this document. It does not modify them.

### 1.2 — Input: Stage 10 Flat Array

The Report Compiler receives the Stage 10 flat array from the Inference Engine after assumption lock. Each record in the array represents a single finding with all inference metadata resolved. Key fields consumed by the Report Compiler include:

- `finding_id`, `title`, `description`
- `inferred_domain` — Human / Boundary / Mechanical
- `assigned_action_path` — Green / Yellow / Red / Observation
- `equipment_class`, `display_name`
- `annual_energy_savings_kwh`, `s_financial` (annual savings in USD)
- `w_fixture_confidence`, `is_default_applied`, `catalog_version`
- `space_id`, `floor`, `zone` — resolved location fields
- `photo_refs[]` — Context / Subject / Detail slots
- `raw_notes`, `facilitator_review_status`
- `reconciliation_status` — must be `LOCKED` for inclusion in report calculations
- `observation_ids[]`, `cluster_ids[]` — source traceability

Any finding with `reconciliation_status` not equal to `LOCKED` is excluded from savings calculations in the report. It may still appear as an observational note at the Analyst's discretion, disclosed as unresolved.

### 1.3 — Retired Concepts

The following concepts appear in earlier versions of the pipeline and are not implemented in V1. They must not be referenced in report output or system design:

**Guarantee Savings Pool, Added Value Pool, Documented Baseline Pool** — the distribution handoff matrix and pool-based savings classification are retired from V1. The README pipeline diagram references these pools and is outdated; it will be updated during the next harmonization wave.

**Finding inclusion/exclusion flag** — there is no formal mechanism in V1 for excluding findings from the report. The `Exclude from report` action in `07_Review_Workflow_Spec` applies only to genuinely anomalous records (unresolved `"Other"` barrier, unrecovered identity). It is not a general editorial curation tool.

**Guarantee/non-guarantee report distinction** — the guarantee calculation is a back-end financial output. It is not report-visible and does not appear as a label, section, or pool in the customer-facing report.

---

## Part 2 — Report Structure

The Deepwalk Discovery Report is rendered in the following section order. Sections 1–7 are the main report body. Section 8 (Appendices) contains four sub-sections rendered as back matter.

| # | Section | Nature |
|---|---------|--------|
| 1 | Executive Summary | Narrative + key metrics |
| 2 | Opportunity Summary + ROI | Financial summary table + action path breakdown |
| 3 | Discovery Team | Participant roster |
| 4 | Floor Summary | Per-floor observation summary |
| 5 | Performance Map | Dual-axis 3×3 grid — findings by domain × action path |
| 6 | Opportunity Value Map | Dual-axis 3×3 grid — savings value by domain × action path |
| 7 | Identified Measures | Finding cards, sorted by measure class then savings value |
| 8 | Appendices | A1: Energy Analysis · A2: Weather Data · A3: Inference Summary · A4: Utility Rebate Forms |

---

## Part 3 — Executive Summary

### 3.1 — Purpose

The Executive Summary provides a concise overview of the engagement: building context, total savings opportunity, and the dominant finding pattern. It is written for a non-technical reader and must comply with P4 — classification describes, not assesses (see Studio Design v3.4 Part 1 P4).

### 3.2 — Content Elements

- Building name, address, gross square footage, engagement date
- Total estimated annual savings (sum of `s_financial` across all `LOCKED` findings)
- Total kWh reduction (sum of `annual_energy_savings_kwh` across all `LOCKED` findings)
- Finding count by domain (Human / Boundary / Mechanical)
- Finding count by action path (Green / Yellow / Red)
- 2–4 sentence narrative characterizing the dominant operational pattern observed. Written by the Analyst. Must not include language that assesses building management performance — per P4, findings describe conditions, not judgments.
- Session date, team size, area covered

### 3.3 — P4 Language Compliance

Narrative text in this section must comply with P4. Permitted framing: "Lighting was observed operating in unoccupied zones." Not permitted: "Lighting management is poor" or "The facility team is not following shutdown protocols." Every narrative statement must be traceable to a specific observation or finding record. Unsourced characterizations are not permitted.

---

## Part 4 — Opportunity Summary + ROI

### 4.1 — Purpose

Presents the total savings opportunity in business terms — total annual value, breakdown by action path, and simple payback where applicable. This section appears early in the report to anchor the reader's understanding of scale before findings detail follows.

### 4.2 — Content Elements

- Total estimated annual savings (USD) — sum of `s_financial` across all `LOCKED` findings
- Total kWh reduction
- Savings breakdown table by action path:

| Action Path | Finding Count | Est. Annual Savings | Notes |
|-------------|--------------|---------------------|-------|
| Green | n | $X | Immediate correction — no capital required |
| Yellow | n | $X | Engineering coordination required |
| Red | n | $X | Asset action required |

- Savings breakdown by domain (Human / Boundary / Mechanical) — finding count and aggregate savings value per domain
- Simple payback where cost estimates are available (sourced from `rough_cost_value` on `MeasureCandidate` records). Where cost estimates are not available, payback is disclosed as not calculated.
- Confidence disclosure: where one or more findings carry `w_fixture_confidence = LOW` or `is_default_applied = TRUE`, a footnote discloses the number of findings affected and the direction of the assumption (conservative).

### 4.3 — ROI Presentation Rules

Savings figures presented in this section are estimates derived from the inference pipeline using the catalog version and assumptions locked at the Analysis Boundary Gate. They are not guarantees of performance. This disclosure must appear in the section, either inline or as a footnote.

---

## Part 5 — Discovery Team

### 5.1 — Purpose

Documents the participants in the Discovery Session — the Third Switch team and the customer participants. Establishes the observational basis of the report and provides contact attribution.

### 5.2 — Content Elements

- Third Switch team members: name, role (`surveyor_role` enum value with display name), and function during session
- Customer participants: name, title, department
- Session date, start time, end time
- Building area covered (floors surveyed, zones accessed)
- Any access limitations or areas not surveyed, drawn from `InaccessibleArea` records (see `06_Spatial_Model_Spec`)

---

## Part 6 — Floor Summary

### 6.1 — Purpose

Provides a per-floor breakdown of observation activity and finding distribution. Gives the reader spatial context for where opportunities were identified.

### 6.2 — Content Elements

For each floor surveyed:

- Floor identifier
- Rooms / zones accessed (count and list of `space_id` values)
- Observation count
- Finding count by action path (Green / Yellow / Red)
- Dominant equipment class observed
- Notable conditions drawn from `raw_notes` flagged by the Analyst as floor-level context

### 6.3 — Sort Order

Floors rendered in building order (lowest to highest, or as labeled in the site floor plan). Mechanical floors and roof levels follow occupied floors.

---

## Part 7 — Performance Map

### 7.1 — Purpose

Renders the Deepwalk Performance Map — the dual-axis 3×3 grid classifying findings by energy control domain (vertical axis) and action path (horizontal axis). This is the primary classification visualization of the report.

### 7.2 — Grid Structure

The Performance Map is a 3×3 grid with fixed axes.

**Vertical axis — Domain (rows):**
- Mechanical (M)
- Boundary (B)
- Human (H)

**Horizontal axis — Action Path (columns):**
- Green — Immediate correction
- Yellow — Engineering coordination required
- Red — Asset action required

Each cell contains the finding names (`title` field) assigned to that domain × action path combination. Cells with no findings are rendered empty — they are not omitted.

### 7.3 — Rendering Rules

- Finding titles in each cell are listed, not summarized
- Cell contents are ordered by `priority_score` descending from the Inference Engine output
- The grid is rendered as a visual table, not a prose description
- Domain and action path labels use the canonical terms defined in Studio Design v3.4. Abbreviated icons (M, B, H) and color coding (Green / Yellow / Red) are used per brand guidelines
- `Observation`-path findings are not placed on the Performance Map grid. They are disclosed separately below the grid with a note that they represent conditions observed but not yet classifiable as optimization opportunities

### 7.4 — Distinction from Opportunity Value Map

The Performance Map shows **what** the findings are — finding names populate the cells. The Opportunity Value Map (Part 8) shows **how much** each region of the same grid is worth in annual savings. They share the same 3×3 structure, the same domain × action path groupings, and the same finding assignments. They are rendered as two separate report elements with different display logic. They must not be conflated in design or described as the same element.

---

## Part 8 — Opportunity Value Map

### 8.1 — Purpose

Renders the financial distribution of the savings opportunity across the Performance Map grid. Shows where annual savings value concentrates by domain and action path. Uses the same underlying groupings as the Performance Map but displays aggregate savings values rather than finding names.

### 8.2 — Grid Structure

Same 3×3 structure as the Performance Map — Mechanical / Boundary / Human rows, Green / Yellow / Red columns. Each cell displays the aggregate `s_financial` (sum of annual savings across all findings in that cell). Cells with no findings are rendered as `$—`, not blank or zero.

### 8.3 — Supporting Summary Elements

**Savings Distribution by Domain** — a table showing for each domain:
- Total savings value
- Number of opportunity categories (distinct `equipment_class` groups with findings in that domain)
- Average value per category

**Savings Concentration narrative** — one or two sentences describing where value concentrates. Must comply with P4. Example: "Value concentrates in the Boundary and Human domains, indicating that most recoverable savings are accessible without capital investment." Must not editorialize about root cause or management behavior.

### 8.4 — Calculation Rules

All values in this section are summed from `s_financial` on `LOCKED` findings only. Findings with `reconciliation_status` not equal to `LOCKED` are excluded from all displayed totals. Where excluded findings exist, a footnote discloses the count and notes that their values are pending resolution.

---

## Part 9 — Identified Measures

### 9.1 — Purpose

Presents individual finding cards in detail — the full enumeration of identified opportunities with supporting data, location, calculation basis, and photo evidence. This is the operationally actionable core of the report.

### 9.2 — Sort Order

Findings are sorted by measure class (equipment class group) ranked by aggregate annual savings value descending, then within each class by individual `s_financial` descending.

The sort is computed at report compile time from locked savings figures:

1. Compute aggregate `s_financial` for each `equipment_class` group across all `LOCKED` findings
2. Rank `equipment_class` groups by aggregate value, highest first
3. Within each group, sort individual findings by `s_financial` descending
4. Render all findings in that group before moving to the next group

This sort order is deterministic and calculated — it is not editorially configured.

### 9.3 — Finding Card Elements

Each finding is rendered as a card containing:

| Element | Source | Notes |
|---------|--------|-------|
| Finding title | `title` | |
| Domain | `inferred_domain` | With icon (M / B / H) |
| Action path | `assigned_action_path` | With color (Green / Yellow / Red) |
| Location | `space_id`, `floor`, `zone` | Resolved location fields |
| Equipment class | `display_name` | Human-readable from catalog |
| Fixture count | `fixture_count` | |
| Observed state | `field_state_enum` | Where logged |
| Finding description | `description` | Per P4 — describes condition, not judgment |
| Load profile | `calculated_p_load_kw` | In kW |
| Est. annual savings | `s_financial` | In USD |
| Est. kWh reduction | `annual_energy_savings_kwh` | |
| Calculation basis | `w_fixture_source`, `w_fixture_confidence` | Disclosed per assumption transparency rules |
| Confidence | `w_fixture_confidence` | `LOW` confidence disclosed explicitly |
| Photos | `photo_refs[]` | Per photo rendering rules in Part 10 |
| Analyst notes | Drawn from `raw_notes` | Analyst-curated for report relevance |

### 9.4 — P4 Language Compliance

All `description` text on finding cards must comply with P4. The description states what was observed and what the finding means operationally. It does not assess the quality of building management, assign blame, or make recommendations beyond the action path classification. Recommendations are implied by the action path, not stated in the finding description.

---

## Part 10 — Photo Rendering Rules

### 10.1 — Photo Slot Priority

Photos are rendered from the three named slots in priority order. If a slot is empty, the next available slot is used. An observation with no photos is valid — no placeholder image is rendered.

| Priority | Slot | Purpose |
|----------|------|---------|
| 1 | `Context` | Wide shot — spatial context, where the asset sits in the room |
| 2 | `Subject` | Full view of the observation subject |
| 3 | `Detail` | Close-up of nameplate, control panel, circuit ID, or technically meaningful detail |

### 10.2 — Rendering Rules

- A maximum of two photos are rendered per finding card in the main report body. Where three photos exist, the third is available in the appendix record for that finding.
- Photos are rendered adjacent to the finding card they are linked to — not in a separate photo appendix.
- Photo captions are auto-generated from the slot name and the `space_id` of the observation (e.g., "Context — Room 204").
- Photos must not be rendered without the finding card they belong to.

---

## Part 11 — Appendices

The appendices contain four sub-sections rendered as back matter following the main report body.

### 11.1 — A1: Energy Analysis

**Purpose:** Establishes the weather-dependent whole-building energy baseline against which savings estimates are contextualized.

**V1 Content:**
- Utility account data: rate structure, annual consumption (kWh), annual spend (USD), billing period
- Whole-building regression model screenshots — the regression model establishes weather-correlated energy usage patterns. In V1, this is a screenshot of an externally-produced regression model, not a system-generated calculation
- Brief narrative contextualizing the regression output relative to the findings (e.g., what proportion of whole-building consumption the identified savings represent)

**Note:** Weather-based savings calculations are not produced by the Inference Engine in V1. The Energy Analysis section provides context, not a calculation basis for any specific finding.

### 11.2 — A2: Weather Data

**Purpose:** Provides the weather data used to support the regression model and contextualize seasonal energy patterns for the engagement site.

**V1 Content:**
- Location and climate zone
- Heating degree days (HDD) and cooling degree days (CDD) for the engagement period
- Source of weather data (station ID or service)
- Temperature range summary for the engagement period

**Standard section:** Included in every report. Where weather normalization has not been applied to specific findings, this section still appears with available data and a disclosure that weather normalization was not applied to individual measure calculations in this engagement.

### 11.3 — A3: Inference Summary / Table of Engineering Assumptions

**Purpose:** Provides a human-readable, forensically examinable record of the engineering assumptions underlying every category-level savings calculation in the report. Enables technical reviewers, auditors, and customer engineering staff to examine and challenge the calculation basis.

**Content:** One row per `equipment_class` group represented in the report findings:

| Column | Content |
|--------|---------|
| Equipment class | `display_name` |
| Catalog entry ID | `catalog_entry_id` |
| Catalog version | `catalog_version` used for this session |
| Wattage assumption (`W_fixture`) | `w_fixture_w` in watts |
| Source | `w_fixture_source` |
| Source reference | `w_fixture_source_ref` |
| Confidence | `w_fixture_confidence` |
| Default applied | `is_default_applied` — flagged explicitly if TRUE |
| Fallback step | Which step of the wattage fallback hierarchy produced this value (Step 1–4) |
| Finding count | Number of findings in this class using this assumption |
| Notes | Any `component_resolution_required` flags or Enrichment Review resolutions affecting this class |

**Disclosure rule:** Any row where `is_default_applied = TRUE` or `w_fixture_confidence = LOW` must be visually flagged. The table must include a footer note: "Assumptions marked with [flag] were resolved using fallback values. These are conservative estimates. Contact Third Switch to initiate a data recovery process for improved precision."

### 11.4 — A4: Utility Rebate Forms

**Purpose:** Provides pre-populated or blank utility incentive application forms relevant to the identified measures and the engagement jurisdiction.

**V1 status:** Conditional. Included only when applicable rebate programs have been identified for the engagement jurisdiction. In V1, this section is produced manually — the Report Compiler does not auto-generate or auto-populate rebate forms. The Analyst attaches relevant forms as PDFs or inserts them as formatted pages.

**When included:** The section lists each rebate program identified, the measures it applies to, the estimated incentive value (where calculable), and the form or application reference. Where incentive values cannot be calculated from pipeline data, the field is left blank for Analyst completion.

---

## Part 12 — Narrative Generation Rules

### 12.1 — System-Generated vs. Analyst-Written Content

| Section | Source |
|---------|--------|
| Executive Summary narrative | Analyst-written, reviewed for P4 compliance |
| Opportunity Summary + ROI tables | System-generated from pipeline data |
| Discovery Team roster | System-populated from session metadata |
| Floor Summary per-floor data | System-generated; notable conditions Analyst-curated |
| Performance Map cell contents | System-generated — finding titles from pipeline |
| Opportunity Value Map cell values | System-generated — savings aggregations from pipeline |
| Finding card structured fields | System-generated from pipeline data |
| Finding card description text | Analyst-written or Analyst-reviewed; must comply with P4 |
| Inference Summary table | System-generated from catalog and pipeline metadata |
| Energy Analysis narrative | Analyst-written |
| Weather Data | System-populated from weather data source |

### 12.2 — P4 Compliance Summary

P4 governs all narrative text in the report. The full principle is defined in Studio Design v3.4 Part 1 P4. Applied here:

- Classification describes the observed condition and its operational context — it does not assess management quality or assign fault
- Action path language describes the nature of the correction path — it does not prescribe specific actions or timelines
- Finding titles and descriptions use present-tense observational language: "Lighting observed operating..." not "Lighting left on..." or "Staff failed to turn off..."
- No finding description may include language implying negligence, incompetence, or willful non-compliance

---

## Part 13 — Open Items

| # | Item | Target |
|---|------|--------|
| 1 | README pipeline diagram update — retire pool references, align with V1 architecture | Next harmonization wave |
| 2 | Regression model integration — V1 uses screenshots; V2 candidate is system-generated regression output from utility data | Post first engagement with weather normalization requirement |
| 3 | Utility Rebate Forms automation — V1 is manual; V2 candidate is auto-population from measure class and jurisdiction | After first rebate-eligible engagement |
| 4 | Finding card sort — current rule sorts by measure class aggregate savings then individual savings; confirm after first report whether editorial override is needed for specific engagement types | After first report delivery |
| 5 | `Observation`-path finding presentation — current spec places these below the Performance Map grid; confirm placement and label language | Before first report with Observation-path findings |
| 6 | Photo rendering — two-photo limit per finding card is a V1 constraint; V2 candidate is configurable per engagement type | Post first engagement |

---

*Deep//Walk by Third Switch — Assurance at Scale*
*© 2026 Third Switch, LLC.*
*This document is internal reference. Not for external distribution without review.*
