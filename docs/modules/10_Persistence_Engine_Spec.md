# 10 — Persistence Engine Spec

**Document Identifier:** DW-T4-10
**Tier:** 4 — Module Spec
**Location:** Google Drive (conceptual sections) + GitHub `docs/modules/` (engineering sections)
**Version:** 2.0
**Date:** May 2026
**Supersedes:** `10_Deepwalk_Persistence_Engine_Spec` v1

**Architecture reference:** See `docs/SYSTEM_ARCHITECTURE_MASTER.md` for tier definitions and cross-reference rules.

**What this document owns:** The Persistence Engine product definition — purpose, scope, and boundaries. The follow-up observation capture form spec. The data backend object schemas. The signal review and validation layer. The aggregation and status logic. The lightweight action tracking layer. The checklist export layer. The dashboard presentation spec. The reporting cadence definitions. The annual true-up and rewalk targeting framework. The business model.

**What this document does not own:** The original Discovery Session findings and inference pipeline — those are immutable inputs to the PE, owned by `docs/architecture/inference-engine-spec.md` (Tier 3). The Facilitator's in-session software loop — that ends at session close per V4 and is defined in `07_Review_Workflow_Spec` (Tier 2). P-principles → Studio Design v3.4 (Tier 2). Measure Catalog wattage assumptions → `03_Measure_Catalog_Spec` (Tier 4). Dashboard calculation methodology dependent on IE or Measure Catalog — open item, not yet specified (see Part 10).

---

## Part 1 — Purpose and Product Definition

### 1.1 — What the Persistence Engine Is

The Persistence Engine (PE) is a longitudinal engagement product delivered as an annual subscription. It extends the value of the Deepwalk Discovery Session by enabling ongoing observation, verification, and status tracking of building operational conditions over time.

The PE is a software product consisting of three components:

- **Web form** — follow-up observation capture interface for field participants
- **Data backend** — stores persistence signals, linking them to original Discovery Session findings where applicable
- **Dashboard** — surfaces persistence status, signal categories, counts, and engagement metrics; exports to CSV

The PE is a CMMS-lite tool. It is not a full task management platform, a building control system, or a replacement for the Discovery Session. It does not recalculate or replace original findings. It tracks what happens to those findings over time and captures new operational signals as they emerge.

### 1.2 — Two Signal Types

The PE handles two distinct signal types. Both are first-class objects in the data model.

**Type 1 — Discovery-linked signals:** Follow-up observations that relate to a known `finding_id` from the original Discovery Session. Status tracks what happened to that finding over time.

**Type 2 — New observations:** Signals that do not link to any original Discovery finding. These may represent conditions that emerged after the session, areas not covered in the original session, or new operational issues surfaced by ongoing participant engagement. In V1, new observations are tracked as signals with category and count. Promotion to formal finding status is deferred — see Part 12, Open Item 1.

Some customers may use the PE to manage their own ongoing energy program without requiring a rewalk. The PE data model supports this use pattern without requiring a new Discovery Session to validate new observations.

### 1.3 — What the PE Does Not Do

- Does not replace or modify original Discovery Session findings
- Does not mutate or overwrite the original Inference Engine output
- Does not function as a building control or automation system
- Does not serve as a full task management or workflow platform
- Does not directly calculate or attribute energy savings in V1 (see Part 10)
- Does not perform a new Inference Engine pipeline pass on follow-up observations in V1

### 1.4 — Architectural Position

The PE is a downstream overlay to the original Discovery Session. The Discovery Session findings are the fixed baseline. The PE tracks the as-maintained state of the building against that baseline and captures new signals beyond it.

```
Discovery Session
→ Inference Engine (Stages 1–10)
→ Report Compiler → Discovery Report (baseline)
→ Persistence Engine (longitudinal overlay)
    → Follow-up observations (Discovery-linked and new)
    → Signal review and validation
    → Status aggregation
    → Dashboard + reporting
    → Annual true-up
```

The PE does not feed back into the Inference Engine in V1. The relationship between PE signals and IE calculations is an open item (see Part 12).

### 1.5 — SEM Positioning

The PE is positioned as SEM-style engagement infrastructure — it provides structured evidence of ongoing operational attention, consistent with Strategic Energy Management program principles. This is a positioning description, not a structural or technical requirement in V1. No specific SEM program documentation format is required by this spec.

---

## Part 2 — Role Boundaries in the PE

### 2.1 — V4 Role Boundary

The Facilitator's software loop ends at session close per V4 (see `07_Review_Workflow_Spec` Part 1). This boundary applies to the PE. The Facilitator does not have a software role in the PE's review, validation, or reporting pipeline.

The Facilitator may participate in PE activities as a **field participant** — attending follow-up walks, capturing follow-up observations using the PE web form, and providing context during annual consultation. This is a business engagement action, not a software role. It is distinct from the in-session software loop and must not be conflated with it in system design, UI role definitions, or documentation.

### 2.2 — Role Assignments in the PE

| Role | PE Function |
|------|-------------|
| Analyst / BD | Signal review and validation; status determination; report review and release; annual true-up coordination; new observation triage |
| Facilitator (field) | Follow-up walk participation; follow-up observation capture via web form; annual consultation engagement |
| Customer participants | Follow-up observation capture via web form; action item ownership; dashboard access |
| Admin | User management; configuration; data export |

---

## Part 3 — Follow-Up Observation Web Form

### 3.1 — Purpose

The web form is the primary data capture interface for post-Discovery operational signals. It is distinct from the Discovery Studio mobile form — it is simpler, web-based, and designed for ongoing use by a broader participant group including customer facilities staff, sustainability leads, and PE-enrolled participants.

### 3.2 — Form Schema

```json
{
  "FollowUpObservation": {
    "observation_id":      "STRING    — UUID. Auto-generated.",
    "session_ref":         "STRING    — Links to the original Discovery Session ID.",
    "finding_ref":         "STRING    — Optional. Links to a specific original finding_id if known.",
    "signal_type":         "ENUM      — DISCOVERY_LINKED / NEW_OBSERVATION.",
    "submitted_by":        "STRING    — Submitter identity.",
    "submitted_at":        "TIMESTAMP — Immutable. System-generated.",
    "space_id":            "STRING    — Room or zone identifier.",
    "equipment_class":     "STRING    — Optional. equipment_class key if known.",
    "observed_state":      "ENUM      — ON / OFF / UNKNOWN.",
    "photo_refs":          "ARRAY     — URIs. Slot names: Context / Subject / Detail.",
    "notes":               "STRING    — Optional. Free text.",
    "owl_count":           "INTEGER   — After-hours occupants observed. Default: 0.",
    "review_status":       "ENUM      — SUBMITTED / APPROVED / REJECTED. Set by Analyst.",
    "review_note":         "STRING    — Optional. Analyst note on review decision."
  }
}
```

### 3.3 — Form Design Constraints

- Web-based. Must be accessible on desktop and mobile browsers. Offline capability is not required.
- `finding_ref` is optional — submitters may not know which original finding their observation relates to. `signal_type` is set by the system or Analyst during review; it is not required from the submitter.
- Photo upload is strongly encouraged but not required for submission.
- The form does not ask submitters for domain or action path classification. Those are backend-resolved properties.

---

## Part 4 — Signal Review and Validation Layer

### 4.1 — Purpose

All incoming follow-up observations pass through a review layer before entering the aggregation pipeline. Only `APPROVED` signals are eligible for status computation and reporting. This prevents the PE data stream from becoming a noisy complaint channel and maintains the integrity of the persistence record.

### 4.2 — Review States

| State | Meaning |
|-------|---------|
| `SUBMITTED` | Observation received; awaiting Analyst review |
| `APPROVED` | Observation accepted into the persistence pipeline |
| `REJECTED` | Observation rejected — duplicate, out of scope, insufficient data, or not interpretable |

### 4.3 — Review Criteria

The Analyst reviews each submitted observation against:

- **Duplicate** — substantially the same observation as a recently approved signal for the same `space_id` and `equipment_class`
- **Relevance** — observation relates to a condition within Deepwalk or PE scope
- **Interpretability** — sufficient information exists to assign a status contribution (photo, notes, or state field)
- **Data completeness** — minimum required fields populated

Rejected observations are retained in the system with the rejection reason recorded in `review_note`. They do not contribute to status calculations or reports.

---

## Part 5 — Status Model

### 5.1 — Finding Status Values

Each Discovery-linked finding tracked in the PE carries a current status derived from the approved signal record. Status is assigned and updated by the Analyst.

| Status | Meaning | Applies to |
|--------|---------|-----------|
| `Resolved` | Condition verified closed; photo evidence supports closure | Discovery-linked findings; PE findings once promoted (V1+) |
| `Persisted` | Condition present; same as originally found or previously reported; never been resolved | Discovery-linked; new observations after first approved signal |
| `Remergent` | Condition returned after a prior `Resolved` status | Discovery-linked findings only; requires at least one prior `Resolved` state |
| `Unverified` | No approved signal received in the current reporting period | All finding types |

### 5.2 — Summary vs. Detail Display

The status set supports two levels of resolution in dashboard and report views:

**Top-line / above the fold (summary):** Three states — `Resolved`, `Active`, `Unverified`. `Persisted` and `Remergent` are collapsed into `Active` for executive-level presentation.

**Detail level:** Full four-state breakdown — `Resolved`, `Persisted`, `Remergent`, `Unverified`. Available in finding lists, trend charts, and recurrence analysis where the distinction between a condition that never resolved and one that returned after resolution is meaningful.

### 5.3 — Status Assignment Rules

- `Resolved` requires at least one approved signal with `observed_state = OFF` and photo evidence, reviewed and confirmed by the Analyst
- `Persisted` is set when an approved signal confirms the condition remains present; applies from the first approved signal for any finding that has never been `Resolved`
- `Remergent` is set when a finding previously marked `Resolved` receives a new approved signal confirming the condition has returned. `Remergent` is not applicable to new observations until they have been formally resolved at least once
- `Unverified` is the default for any finding that has received no approved signals in the current reporting period

### 5.4 — New Observation Status Path

New observations (Type 2 signals) enter the status model as follows:

- Default status on first approved signal: `Persisted` (condition is present; no prior baseline to compare against)
- `Remergent` becomes applicable only after a new observation has been formally resolved at least once
- `Resolved` is available once the Analyst confirms closure with supporting evidence
- `Unverified` applies when no approved signal has been received in the current period

Promotion of a new observation to a formal PE finding — with full status history and dashboard parity with Discovery-linked findings — is deferred to V1+. See Part 12, Open Item 1.

### 5.5 — Recurrence Clustering

The backend groups `Persisted` and `Remergent` signals by `space_id`, `equipment_class`, and time window to identify recurrence patterns — zones or equipment classes where conditions persistently return. Recurrence clusters are surfaced in the dashboard detail view and inform rewalk targeting in the annual true-up (see Part 9).

---

## Part 6 — Lightweight Action Tracking

### 6.1 — Purpose

The PE provides lightweight tracking of action items associated with findings. This is a visibility tool — it tracks whether correction actions have been initiated and verified, not a task management or workflow execution system.

### 6.2 — Action Item Schema

```json
{
  "ActionItem": {
    "action_id":        "STRING    — UUID. Auto-generated.",
    "finding_ref":      "STRING    — Links to original finding_id or PE observation_id.",
    "owner":            "STRING    — Assigned owner name or role.",
    "status":           "ENUM      — OPEN / IN_PROGRESS / CLOSED.",
    "verification_ref": "STRING    — Optional. Links to a FollowUpObservation ID with photo evidence.",
    "created_at":       "TIMESTAMP — Immutable.",
    "updated_at":       "TIMESTAMP — Updated on status change.",
    "notes":            "STRING    — Optional."
  }
}
```

### 6.3 — Explicit Constraints

The action tracking layer does not include:

- Scheduling engine or due date enforcement
- Assignment workflows or escalation logic
- Notification or reminder systems
- Integration with external task management platforms (V1)

The PE tracks operational signal reality, not workflow execution. Checklist export (Part 7) is the mechanism for customer ingestion into their own execution systems.

---

## Part 7 — Checklist Export Layer

### 7.1 — Purpose

The PE generates implementation-ready checklists based on original Discovery Session findings, resolved location data, and current PE status. These checklists are designed for customer ingestion into their internal maintenance or project systems. Deepwalk facilitates the data export; it does not own execution of the checklist items.

### 7.2 — Export Content

Each row represents one finding and contains:

| Field | Content |
|-------|---------|
| Finding ID | `finding_id` from Discovery Session or PE observation_id |
| Signal type | `DISCOVERY_LINKED` / `NEW_OBSERVATION` |
| Title | `title` or category label |
| Location | Resolved `space_id`, `floor`, `zone` |
| Equipment class | `display_name` |
| Action path | Green / Yellow / Red (Discovery-linked findings only) |
| Domain | Human / Boundary / Mechanical (Discovery-linked findings only) |
| Current PE status | `Resolved` / `Persisted` / `Remergent` / `Unverified` |
| Action item owner | From action tracking layer |
| Action item status | `OPEN` / `IN_PROGRESS` / `CLOSED` |
| Notes | Analyst-curated |

### 7.3 — Export Formats

- **CSV** — primary format; designed for ingestion into CMMS, FM tools, and project systems
- **PDF** — optional; formatted for human review and distribution

---

## Part 8 — Dashboard

### 8.1 — Purpose

The dashboard is the primary PE interface for ongoing visibility. It surfaces persistence status, signal categories, counts, and engagement metrics for enrolled participants and Third Switch Analysts.

In V1, the dashboard presents categories and counts. Energy usage and savings potential figures are not displayed in V1 — the calculation methodology for translating PE signals into savings estimates is an open item (see Part 12).

### 8.2 — Dashboard Views

**Finding Status Summary (above the fold)**
- Count of findings by summary status: `Resolved` / `Active` / `Unverified`
- Summary status breakdown by domain (Human / Boundary / Mechanical) where applicable
- Trend over time — summary status distribution across reporting periods

**Finding Status Detail**
- Full four-state breakdown: `Resolved` / `Persisted` / `Remergent` / `Unverified`
- Filterable by domain, action path, floor, zone, and equipment class

**New Observations**
- Count of Type 2 (new observation) signals by category and `space_id`
- Approved vs. submitted vs. rejected signal counts
- Not mixed with Discovery-linked finding status in V1

**Recurrence Hotspots**
- Zones and equipment classes with the highest `Persisted` and `Remergent` signal density
- Used for rewalk targeting and follow-up prioritization

**Signal Density**
- Observation submission volume over time
- Approved vs. rejected signal ratio
- Active submitter count — engagement health indicator

**Action Item Tracker**
- Open / In Progress / Closed action item counts
- Items by owner
- Items with verification evidence vs. unverified closures

**Savings Persistence** *(placeholder — V1+)*
- Reserved view for future savings persistence calculation
- Not implemented in V1

### 8.3 — CSV Export

All dashboard views are exportable to CSV. Export is available to Analyst, BD, and Admin roles. Customer participant export access is configurable.

---

## Part 9 — Reporting Cadences

### 9.1 — Monthly Pulse

Lightweight engagement summary distributed to customer participants and Third Switch Analyst.

- Signal submission volume
- New findings resolved or status-changed in the period
- Immediate interventions flagged for quick action
- Engagement health indicator (active submitter count)

### 9.2 — Quarterly Review

Operational persistence summary reviewed in a structured session with customer stakeholders.

- Finding status distribution and trend since last quarter
- Recurrence patterns and hotspot identification
- Action item progress summary
- System friction patterns — findings consistently `Persisted` or `Unverified` without action

### 9.3 — Annual Persistence Narrative

Full longitudinal summary. Anchor document for the annual consultation and true-up.

- `Resolved` vs. `Active` vs. `Unverified` distribution across all findings
- `Persisted` vs. `Remergent` breakdown within `Active`
- Implementation effectiveness — findings with verified closure evidence vs. unverified
- New observation volume and category distribution
- Residual opportunity framing — findings still `Unverified` or `Active` at year end
- Rewalk targeting recommendation

---

## Part 10 — Annual True-Up and Rewalk Targeting

### 10.1 — True-Up Principle

PE observations provide SEM-style engagement evidence of operational persistence. They are not the authoritative source for energy savings determination. Normalized whole-building energy data remains the authoritative basis for final savings quantification.

The annual true-up reconciles:

- PE finding status distribution (what the operational record shows)
- Normalized building energy data (what the meters show)
- Original Discovery Session savings estimates (what was projected)

### 10.2 — Rewalk Targeting

The annual true-up produces a rewalk target list — a prioritized set of zones, floors, or equipment classes recommended for the next Discovery Session or follow-up walk. Rewalk targets are derived from:

- Findings with `Persisted` or `Remergent` status at year end
- Recurrence hotspot clusters
- Findings marked `Unverified` across multiple reporting periods
- New observations that have accumulated signal density without formal finding status

---

## Part 11 — Business Model

The PE is delivered as an annual subscription. The subscription includes:

- Data storage for follow-up observations, photos, and status records
- Dashboard access for enrolled participants
- Reporting capability (Monthly Pulse, Quarterly Review, Annual Persistence Narrative)
- Annual consultation — Third Switch advisory engagement including true-up coordination and rewalk targeting

The subscription does not include a new Discovery Session. Rewalks are scoped and priced separately.

---

## Part 12 — Open Items

| # | Item | Target |
|---|------|--------|
| 1 | New observation promotion path — mechanism for Analyst to promote a Type 2 signal to a formal PE finding with full status history and dashboard parity; requires VOC before spec | After first PE engagements |
| 2 | Dashboard savings persistence calculation — methodology for translating finding status into savings persistence estimate; IE and Measure Catalog dependency | Before first PE subscription renewal requiring savings true-up |
| 3 | Customer participant access controls — which dashboard views and export capabilities are available to customer roles vs. Analyst/BD | Before first PE deployment |
| 4 | `finding_ref` auto-resolution — backend logic for linking follow-up observations with no `finding_ref` to the correct original finding | Engineering review before production deployment |
| 5 | External system integration — CSV export is V1; V2 candidate is direct integration with CMMS and FM platforms | Post first PE engagement |
| 6 | SEM program documentation alignment — if a specific SEM program requires structured evidence format, PE reporting templates will need to conform | Before first SEM-enrolled PE engagement |
| 7 | Notification and reminder system — explicitly excluded from V1; V2 candidate | Post first PE engagement |
| 8 | README pipeline diagram update — add PE as downstream overlay; retire pool references | Next harmonization wave |

---

*Deep//Walk by Third Switch — Assurance at Scale*
*© 2026 Third Switch, LLC.*
*This document is internal reference. Not for external distribution without review.*
