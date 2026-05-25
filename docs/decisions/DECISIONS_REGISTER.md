# Deepwalk Architecture Decisions Register

*Canonical reference — maintained in GitHub `docs/decisions/`*
*Tier 1 — owned by Third Switch leadership*
*Migrated from Google Drive working document, May 2026*
*Source: Architecture review conversations, May 2025 – May 2026*

---

## How to Use This Document

This register records architectural decisions that govern the Deepwalk system. Decisions are immutable once marked ✅ RESOLVED — they may only be superseded by a new entry. Open items are marked 🔄 PENDING or 🅿️ PARKED.

Each entry records what was decided, why, and where the canonical implementation lives. Cross-references to spec documents are provided where applicable.

---

## Foundational Principles

### P1 — Triage as the Collection Model

Distributed, low-expertise observers cover ground that trained specialists couldn't cover economically. Speed and coverage over depth. Value emerges from patterns across many fast observations, not from depth of any single observation.

### P2 — Observed Violation Persistence

Any observed violation is assumed constant (full annual frequency) unless specific facts are supplied to adjust it. The burden of proof runs toward correction, not toward finding. This is the default annualization assumption for all savings calculations.

### P3 — Turn Over Every Stone

*"If you're making promises to customers, you have to turn over every stone."*

The 10% guarantee is a commitment, not a hedge. The analysis must be exhaustive within the observable energy boundary — not selective. Every observation either contributes to the guarantee or is documented as to why it doesn't.

**Implications:**
- Nothing observed is silently dropped from the report
- Observations that can't generate savings estimates (regulated, out of scope, unidentifiable) still appear with a documented explanation
- The client sees everything observed and understands exactly why each item is or isn't in the savings calculation
- Every observation has a disposition — finding, coverage record, unresolved observation, or out-of-scope — and that disposition is visible in the output
- Regulatory deductions, scope exclusions, and annualization adjustments are shown transparently, not applied silently

---

## Section A — Core Principles and Pipeline Architecture

### A1 — Coverage-Weighted Session Confidence ✅ RESOLVED

**Resolution:** Coverage is a pre-event boundary condition on project scope, not a runtime variable in the inference engine.

**Key decisions:**
- Unobserved space is excluded from the savings guarantee
- Project energy boundary is derated based on observable space
- Derating uses space-type-adjusted EUI (CBECS reference — methodology TBD) or total EUI as absolute fallback
- EUI methodology source is 🅿️ PARKED — not a showstopper, multiple solutions exist

**New pipeline stage established: Analysis Boundary Gate**

Sits between Discovery Session submission and inference execution. Inference engine does not run until this object is locked.

```
AnalysisBoundary {
  observable_sqft
  total_sqft
  coverage_ratio
  excluded_spaces[]         // each with reason
  space_type_mix            // if available
  eui_source                // enum: CBECS_adjusted | total_EUI_fallback
  project_energy_boundary   // calculated
  adjusted_savings_target   // derated from contract guarantee
  boundary_locked_by        // Facilitator (executor, not decision-maker)
  boundary_locked_at        // timestamp
  added_value_threshold     // findings above this = upside, not guarantee
}
```

**Commercial logic:**
- System restricts final output to meet guarantee (no more, no less)
- Findings exceeding adjusted_savings_target flagged as added value
- Added value findings either reserved for future scope or folded in explicitly

**Role boundary:**
- Facilitator locks the Analysis Boundary as executor
- Contractual and commercial interpretation is a Third Switch upstream decision — not Facilitator judgment

**Pre-session guarantee feasibility estimate (new requirement):** Must be produced before the session is scheduled.

Initial feasibility variables:
- Occupancy density
- Work schedule
- HVAC type
- Customer-supplied mechanical, lighting, and equipment inventories (primary input)

Inventory quality as negotiation lever: absence or poor quality of customer-supplied inventory is a scoping negotiation point re: guarantee or product scope — handled commercially, not as a system requirement.

---

### A2 — Facilitator Review Bottleneck at Triage Scale ✅ RESOLVED

**Resolution:** Review layer is type-specific routing, not a single queue. Constant violation principle (P2) prevents low inference confidence from becoming a review trigger on its own. Review is triggered by data quality failure modes only.

| Type | Description | Disposition | Review Layer |
|------|-------------|-------------|--------------|
| 1 | Room-level only | Coverage record | None |
| 2 | Unclear identity | `unresolved_observation` | Post-session: photo review + gap survey if guarantee at risk |
| 3a | Inaccessible for identification | Retained, wattage fallback hierarchy | Gap survey if guarantee at risk |
| 3b | Accessible but undocumented | Lowest confidence signal, not recoverable | Training issue — not recoverable post-session |
| 4 | State uncertain | As-observed stands, `state_confidence` field | Photo record adjudicates |
| 5 | Out of scope | Retained, `out_of_scope` + reason code | Pre-inference scope exclusion pass |
| 6 | Redundant | Retained, `corroborates_observation_id` | Backend only |
| 7 | Fraudulent | All observations from participant discarded | Commercial/contractual — upstream of system |

---

### A3 — Facilitator Review Bottleneck (Triage Scale) ✅ RESOLVED

Resolved as byproduct of A2. Key principle: P2 prevents low inference confidence from triggering review. Review is triggered by data quality failure modes only — not by inferential uncertainty.

---

### A4 — Four-Tier Document Architecture ✅ RESOLVED *(May 2026 sprint)*

**Resolution:** All Deepwalk documentation is organized into four tiers with fixed locations and owners.

| Tier | Name | Location | Owner |
|------|------|----------|-------|
| 1 | Decision Register | GitHub `docs/decisions/` | Third Switch leadership |
| 2 | Conceptual / Operational Track | Google Drive | Product, methodology, ops |
| 3 | Engineering Track | GitHub `docs/` | Engineering |
| 4 | Module Specs | Drive + GitHub `docs/modules/` | Assigned per module |

**Reference Library** — shared, not owned by any tier. Google Drive.

Governing document: `docs/SYSTEM_ARCHITECTURE_MASTER.md`. Every new document must open with a tier declaration and cite this document.

**Canonical term:** Engineering Track. "Programmatic Tier" is retired.

---

### A5 — Savings Pool Architecture Retired ✅ RESOLVED *(May 2026 sprint)*

**Resolution:** Guarantee Savings Pool, Added Value Pool, and Documented Baseline Pool are retired from V1 architecture. The distribution handoff matrix and pool-based savings classification are not implemented.

**Implications:**
- Guarantee/non-guarantee distinction is a back-end financial calculation only — not report-visible
- No finding inclusion/exclusion flag in V1 for guarantee categorization
- README pipeline diagram is outdated and references these pools — scheduled for update in next harmonization wave
- `07_Review_Workflow_Spec` `Exclude from report` action applies only to anomalous records (unresolved `"Other"` barrier, unrecovered identity) — not a general curation tool

**Canonical reference:** `04_Report_Compiler_Spec` Part 1.3.

---

## Section B — Naming Conflicts

### B1 — Measure Type vs. Measure Category/Detail vs. equipment_type ✅ RESOLVED

**Resolution:**
- Field-facing term: `observation_type` — single field, single broad selection, completed by Discovery Team
- Back-end term: `equipment_class` — normalized value assigned post-submission by inference engine
- `Measure Category` + `Measure Detail` dependent pair: retired from all field-facing forms
- `Measure Type` (M3Form): retired
- `equipment_type` (Inference Engine spec): renamed `equipment_class`

**Final observation track structure:**
- `equipment` — discrete device or fixture, nearly all observations, all roles including SS when assisting
- `condition_reading` — location-anchored environmental measurement, SS primary, Facilitator secondary
- Track is inferred by the system from submitted data type — no field-level mode switch required

**Canonical reference:** `01_Discovery_Studio_Config_Spec` Part 4.

---

### B2 — Device vs. Object (photo slot) ✅ RESOLVED

**Resolution:** Canonical name: `Subject`

**Final photo slot names — locked:**
- Slot 1: `Context` — wide shot, all observations, all roles
- Slot 2: `Subject` — full view of observation subject, equipment and condition readings
- Slot 3: `Detail` — any close-up technical or identifying information, equipment observations only

SS photo protocol: maximum 2 photos per condition reading (Context + Subject). Detail slot not applicable to condition readings.

**Canonical reference:** `01_Discovery_Studio_Config_Spec` Part 8.

---

### B3 — Nameplate vs. Technical (photo slot) ✅ RESOLVED

**Resolution:** Canonical name: `Detail`

Covers: nameplates, model labels, control panel settings, thermostat readings, breaker labels, circuit IDs, timer configurations, sensor type labels, any close-up identifying or diagnostic information. Equipment observations only.

**Retired names:** `Device Photo`, `Nameplate/Closeup`

**Canonical reference:** `01_Discovery_Studio_Config_Spec` Part 8.

---

### B4 — `site` vs. `building_name` ✅ RESOLVED

**Resolution:** Canonical term: `site`

- `site` is the top-level engagement scope — may contain one or more buildings
- `building_name` retired from ingestion layer — replaced by `site` + `building` child

**Location hierarchy — locked:**
```
Site
  └─ Building
       └─ Floor
            └─ Zone
                 └─ Room
                      └─ Waypoint
```

Analysis Boundary Gate operates at `site` level. Derating applied at `building` level when coverage is partial within a multi-building site.

---

### B5 — `location` as string vs. Location object ✅ RESOLVED

**Resolution:** Hybrid input model — simple field-facing input, structured back-end object.

**Field-facing input:**
- Primary: pre-loaded room list from customer-supplied floor plans
- Fallback: short free text entry when room isn't on list or no list exists

**Back-end:** Free text entries normalized post-session. Location object is a back-end construct — not dependent on surveyor providing structured input.

**Location object — canonical structure:**
```
Location {
  site_id
  building_id
  floor
  zone
  room
  waypoint          // optional
  raw_input         // preserved — original surveyor text entry
}
```

**Canonical reference:** `06_Spatial_Model_Spec`.

---

### B6 — "Ingestion" vs. "Pre-Session Confidence Inputs" ✅ RESOLVED

**Resolution:** Two names coexist by design.
- **Client-facing name: `Scoping`** — lightweight intake process
- **Internal system name: `Ingestion`** — processing of submitted materials into guarantee feasibility estimate and session blueprint
- **"Pre-Session Confidence Inputs"** retired — replaced by Tier A/B/C materials within Scoping

**Canonical reference:** `09_Scoping_Ingestion`.

---

### B7 — "Data Scrubbing" vs. "Normalize" ✅ RESOLVED

**Resolution:**
- `Data Scrubbing` — named sub-process: deduplication, missing name resolution, data hygiene. Human-assisted.
- `Normalize` — canonical internal term for full Stage 1 of the inference pipeline: includes data scrubbing plus synonym mapping, equipment class assignment, state value standardization, and session metadata attachment.

Data scrubbing is the human-assisted portion of normalization. Both terms are retained — they describe different layers of the same stage.

---

### B8 — Action Path Terminology ✅ RESOLVED *(May 2026 sprint)*

**Resolution:** Action path labels in software output are `Green / Yellow / Red`. `Immediate / Coordination / Capital` are retired as software output labels. `Observation` is retained as a fourth action path.

**Canonical action paths — locked:**
- `Green` — locally correctable, no capital, behavior-driven (previously: Immediate / Operational)
- `Yellow` — cross-team dependency, configuration or schedule issue (previously: Coordination)
- `Red` — engineering or replacement required (previously: Capital)
- `Observation` — insufficient evidence to assign corrective path; signal is real but follow-up required

**Note:** `Immediate / Coordination / Capital` language may still appear in older source documents and Reference Library items scheduled for harmonization. The canonical software output labels are Green / Yellow / Red.

**Canonical reference:** Studio Design v3.4; `08_Surveyor_Roles`; `04_Report_Compiler_Spec`.

---

### B9 — Scoping Input Tier Labels ✅ RESOLVED *(May 2026 sprint)*

**Resolution:** Scoping input tiers are labeled A / B / C to avoid collision with the document tier model (Tier 1–4).

- Tier A — highest quality customer-supplied data
- Tier B — partial or secondary data
- Tier C — minimal or no customer data

**Retired labels:** Tier 1 / 2 / 3 in scoping context.

**Canonical reference:** `09_Scoping_Ingestion`.

---

### B10 — Facilitator Post-Session Software Role ✅ RESOLVED *(May 2026 sprint)*

**Resolution:** The Facilitator's software loop ends at session close. Post-session review, enrichment, and analysis are owned by the Analyst/BD.

The Facilitator may participate in post-session activities as a business engagement action (annual consultation, follow-up walks) — not as a software role. These are distinct and must not be conflated in system design, UI, or documentation.

**Canonical reference:** `07_Review_Workflow_Spec` Part 1 and Part 2.

---

## Section C — Schema Gaps

### C1 — Session Enrichment Context Object ✅ RESOLVED

**Resolution:** Named object in the pre-event ingestion layer, updateable through the full engagement lifecycle until the Analysis Boundary Gate locks.

```
SessionEnrichmentContext {
  site_id
  space_ownership[]
  macro_occupancy_factors {
    construction_active
    adds_moves_changes
    annual_downtime_periods[]
    company_routine_events[]
    holidays_observed[]
    divestitures_pending
    acquisitions_pending
  }
  operational_policies[] {
    policy_type
    description
    affected_zones[]
    inference_effect
  }
  regulations[] {
    regulation_type
    description
    affected_zones[]
    inference_effect
  }
  captured_by
  last_updated
}
```

**Pipeline behavior:** During Scoping, policies and regulations recorded as facts only. Post-session, Third Switch analyst assigns `inference_effect` with full visibility of actual observations. Protects against pre-session suppression decisions that could create re-walk liability.

**Canonical reference:** `09_Scoping_Ingestion`.

---

### C2 — space_ownership_type ✅ RESOLVED

**Resolution:** `space_ownership_type` retired. Replaced by `space_function` — a client-familiar taxonomy mapped to inference behavior on the back end.

**`space_function` taxonomy — canonical list:** See `09_Scoping_Ingestion` for full list.

---

### C3 — Macro Occupancy Factors ✅ RESOLVED

**Resolution:** Macro occupancy factors are recorded as facts during Scoping. Their effect is managed through a dedicated **Adjustments Module** — a focused analytical tool for Third Switch analysts.

```
Adjustment {
  adjustment_id
  session_id
  adjustment_type
  description
  affected_scope { site_id, building_id, floor, zone }
  hours_adjustment
  direction
  confidence
  evidence
  created_by
  created_at
  report_element
}
```

All adjustments are logged with evidence and analyst attribution — full audit trail. Human analyst judgment governs every adjustment.

---

### C4 — Operational Policies ✅ RESOLVED

**Resolution:** Operational policies have two distinct inference effects:

- `confidence_amplifier` — strengthens constant assumption (e.g., poor cleanliness / maintenance discipline)
- `confidence_suppressor` — reduces annualization assumption (e.g., holiday decor, client visit protocols)
- `neutral_annotate` — noted in report but does not affect inference scoring

Custodial presence explains ambient/overhead loads only. It does not explain occupant-specific loads (monitors, task lights, personal devices). Those violations stand regardless of who else is in the room.

---

### C5 — Regulatory Constraint Layer ✅ RESOLVED

**Resolution:** Regulations define a floor below which correction paths cannot go. They do not suppress findings — they constrain the correctable portion of the savings estimate. All findings are reported; the regulatory deduction is shown transparently.

**Report format:** Full potential savings → regulatory deduction (with code reference) → **net correctable savings (bold)**

Regulatory deductions are calculated and applied in the Adjustments Module alongside macro occupancy factor adjustments.

---

### C6 — Restricted Area Flag ✅ RESOLVED

**Resolution:** Restricted areas are never silently dropped. Two scenarios:

**Scenario A — Restriction discovered at the door:** Added to `excluded_spaces[]` with reason code `access_denied_at_event`. Savings guarantee adjusted. Must be addressed in client contracts.

**Scenario B — Loads observable from outside:** Grouped as `restricted_area_observation`. Valid under P2. Analyst determines correctable fraction post-session.

---

### C7 — Food Service as Distinct Inference Driver ✅ RESOLVED

Addressed within C4 cleanliness dimensions and cold storage findings. Food service cleanliness is partially expected — do not apply office-environment operational discipline amplifier. Equipment-specific cleanliness (coils, gaskets, strip curtains) remains a strong Mechanical finding regardless of space type.

---

### C8 — Bulldog Capture Field ✅ RESOLVED

**Resolution:** Bulldog capture is a dedicated quick-capture mechanism — separate from the standard observation form.

```
BulldogObservation {
  session_id
  timestamp
  space_id
  bulldog_type    // enum: Security | Cleaning | Construction | Contractor | Unknown
  count           // default 1
  notes           // optional
  badge_photo_ref // optional — URI pointing to badge/name tag photo
}
```

`Unknown` Bulldogs generate a conservative influence window until type is resolved post-session. UI: persistent bottom bar or floating button — always visible, never buried.

**Canonical reference:** `01_Discovery_Studio_Config_Spec` Part 2 and Part 9.

---

### C9 — `why_not` Field — Reduced to 4 Values ✅ RESOLVED *(May 2026 sprint)*

**Resolution:** `why_not` field reduced from 6 values to 4 permitted string values plus null.

**Canonical values — locked:**
- `"No Switch or Control Found"` — replaces both `"No Switch Present"` and `"Control Not Found"`
- `"Requires Permission"` — retained
- `"Occupancy Sensor"` — retained; requires positive ID of sensor controlling the specific asset
- `"Other"` — retained as intentional catch-all for Enrichment Review
- `null` — only when `did_you_turn_it_off` is TRUE

**Retired values:** `"No Switch Present"`, `"Control Not Found"`, `"Timer / Schedule"`

**`"Timer / Schedule"` — removed from V1.** Reserved for V2 with additional controls: will require a `Detail` photo of the control device and will be restricted to `Facilitator` or verified facilities-staff attribution.

**Canonical reference:** `01_Discovery_Studio_Config_Spec` Part 6.

---

### C10 — `field_state_enum` Demoted to Optional ✅ RESOLVED *(May 2026 sprint)*

**Resolution:** `field_state_enum` is optional. Default: `UNKNOWN`. Photos and `raw_notes` are authoritative state signals; `field_state_enum` is a structured shortcut.

**Values:** `ON` / `OFF` / `AUTO` / `UNKNOWN`

`AUTO` requires positive identification of a control device governing the specific asset. Not a fallback for uncertainty — use `UNKNOWN` for that.

Long-term fate (Option A: keep optional vs. Option B: remove entirely) is an open decision. Do not change without a Decision Register entry.

**Canonical reference:** `01_Discovery_Studio_Config_Spec` Part 5.

---

### C11 — Measure Catalog Architecture ✅ RESOLVED *(May 2026 sprint)*

**Resolution:** Two distinct deliverables:
- `03_Measure_Catalog_Spec` (Tier 4) — translation spec: ingestion rules, schema, version lifecycle, fallback hierarchy, IE interface contract
- `02_Measure_Catalog` (Reference Library) — living catalog: starts empty, populated through engagements and maintenance processes

**Catalog is the single runtime wattage lookup target.** Pipeline never queries external sources (DEER, ENERGY STAR, DLC, ASHRAE) directly at runtime. Those sources feed a catalog maintenance process that runs separately.

**Version lifecycle:** `Draft → Commit → Release → Deprecated`

**Component handling (V1):** Catalog unit is the observable equipment class. Sub-component relationships are not modeled in V1. `component_resolution_required` flag triggers Enrichment Review when component detail materially affects the wattage assumption. Option C (parent-child component registry) is noted as a planned future capability.

**Canonical reference:** `03_Measure_Catalog_Spec`.

---

### C12 — PE Finding Status Set ✅ RESOLVED *(May 2026 sprint)*

**Resolution:** Persistence Engine finding status values:

| Status | Meaning |
|--------|---------|
| `Resolved` | Condition verified closed; photo evidence supports closure |
| `Persisted` | Condition present; same as originally found; never been resolved |
| `Remergent` | Condition returned after a prior `Resolved` status |
| `Unverified` | No approved signal received in the current reporting period |

`In Progress` dropped — insufficient use case identified.

**Display rules:** `Persisted` and `Remergent` collapse to `Active` in top-line summary views. Full four-state breakdown available in detail views.

`Remergent` is not applicable to new observations until they have been formally resolved at least once.

**Canonical reference:** `10_Persistence_Engine_Spec` Part 5.

---

### C13 — PE New Observation Promotion Path ✅ RESOLVED *(May 2026 sprint)*

**Resolution:** Option C — deferred. In V1, new PE observations (Type 2 signals) are tracked as signals with category and count only. Promotion to formal finding status requires VOC before spec. Decision deferred to post-first-PE-engagement.

**Canonical reference:** `10_Persistence_Engine_Spec` Part 12, Open Item 1.

---

## Section D — Specification Gaps

### D1 — Annualization + ROI Calculation Methodology ✅ RESOLVED

**Resolution:** Lighting savings calculation is the canonical example. Three-stage model established from field practice.

**Stage 1 — Reduction Hours:**
1. Establish occupied hours from customer input
2. Check automation schedule — if functioning as programmed, apply schedule ceiling; if not, apply P2 (constant assumption)
3. Apply macro occupancy factor adjustments via Adjustments Module
4. Apply regulatory deductions if applicable

**Stage 2 — Wattage:** See wattage fallback hierarchy (D3 below).

**Stage 3 — Annual Savings:**
```
kWh savings = (watts ÷ 1000) × reduction_hours
$ savings = kWh savings × blended_utility_rate
```

**Two-stage calculation model:**
- Stage 1 (field-based estimate): P2 applied to all observations where BAS state is unknown. Conservative floor. Sufficient to finalize the report if Tier 3 data never arrives.
- Stage 2 (analyst-refined estimate): For observations where BAS confirmed functioning as programmed, schedule ceiling replaces constant assumption. Stage 2 only improves the estimate — never reduces it below Stage 1 floor.

BAS data is the client's responsibility. If it doesn't arrive or is incomplete, Stage 1 stands as the final calculation.

---

### D2 — Weather Data as Modeling Input ✅ RESOLVED

**Resolution:** Weather data serves two purposes:

**Purpose 1 — Whole building energy regression:** Weather is the independent variable for correlating building energy consumption patterns. Establishes baseline model against which other load impacts are estimated as residuals. Data source: nearest NOAA weather station or TMY3.

**Purpose 2 — Envelope and window heat transfer inference:** SS IR temperature readings at exterior walls and windows compared against concurrent outdoor conditions at time of observation.

```
WeatherContext {
  site_id
  session_date
  weather_station_id
  hourly_conditions[] { timestamp, outdoor_temp_f, humidity_pct, solar_radiation }
  annual_hdd
  annual_cdd
  data_source
}
```

Not used in: plug load, lighting, or standard equipment savings calculations.

---

### D3 — Wattage Fallback Hierarchy ✅ RESOLVED *(May 2026 sprint)*

**Resolution:** Formally specified as a 4-step lookup sequence. Catalog is the single runtime target — external sources feed the catalog maintenance process, not the runtime pipeline.

**Step 1 — Catalog exact match:** `equipment_class` + `manufacturer` + `model` against `RELEASE`-status entry in `02_Measure_Catalog`.

**Step 2 — Catalog class-level match:** `equipment_class` only against `RELEASE`-status entry. `COMMIT`-status entry permitted when no `RELEASE` entry exists, subject to exclusion from guarantee pool calculations.

**Step 3 — Auto-populate from external source:** No catalog match. System queries configured external sources in engagement-specified priority order (DEER, ENERGY STAR, DLC, ASHRAE — priority order is engagement-configurable, not ranked in this register). Result written to catalog as `DRAFT` entry with `auto_populated = TRUE` and `ingestion_timestamp`. Routed to catalog maintenance queue.

**Step 4 — Global catch-all:** No catalog match and no external source match. Internal defaults table minimum applied. `is_default_applied = TRUE`, `w_fixture_confidence = LOW`. Routed to Enrichment Review. Not written to catalog until manually confirmed.

**External source query priority** is engagement-configurable — negotiated with customer or sponsor. This register does not rank external sources.

**Canonical reference:** `03_Measure_Catalog_Spec` Part 7.

---

### D4 — Sensor/Counter Data Validation Gate ✅ RESOLVED

**Resolution:** Three documented failure modes: incorrect installation, unexpected load coverage, data gaps.

```
SensorValidationRecord {
  sensor_id
  session_id
  data_source
  coverage_confirmed
  coverage_notes
  installation_plausibility    // enum: plausible | suspect | failed
  plausibility_basis
  data_gap_pct
  gap_disposition              // enum: acceptable | interpolated | excluded
  validated_by
  validated_at
  approved_for_use             // boolean — final gate decision
}
```

`approved_for_use = false` → data logged but cannot influence savings calculations or override observation inference. Observation-based inference (Stage 1) always runs first — sensor data refines, never replaces. Validated by Third Switch analysts post-session — not Facilitator, not field team.

---

### D5 — Pilot Tracking Metrics ✅ RESOLVED

**Resolution:** Pilot tracking metrics belong in the SessionContext and AnalysisBoundary objects — not a separate tracking system.

```
SessionMetrics {
  session_id
  total_observations
  observations_by_role {}
  flagged_observations
  unresolved_observations
  coverage_sqft
  coverage_ratio
  session_duration_minutes
  observations_per_hour
  rooms_visited
  bulldog_events
  owl_count_total
  photos_submitted
}
```

Not client-facing in V1.

---

### D6 — Report Section Order ✅ RESOLVED *(May 2026 sprint)*

**Resolution:** Canonical Deepwalk Discovery Report section order:

| # | Section |
|---|---------|
| 1 | Executive Summary |
| 2 | Opportunity Summary + ROI |
| 3 | Discovery Team |
| 4 | Floor Summary |
| 5 | Performance Map |
| 6 | Opportunity Value Map |
| 7 | Identified Measures |
| 8 | Appendices |
| A1 | Energy Analysis |
| A2 | Weather Data |
| A3 | Inference Summary / Table of Engineering Assumptions |
| A4 | Utility Rebate Forms *(conditional)* |

**Performance Map vs. Opportunity Value Map:** Same 3×3 structure, same domain × action path groupings, same underlying finding assignments. Different display logic — Performance Map shows finding names; Opportunity Value Map shows aggregate savings values. These are two distinct rendered report elements and must not be conflated.

**Identified Measures sort order:** By measure class (`equipment_class` group) ranked by aggregate annual savings value descending, then within each class by individual `s_financial` descending. Computed at report compile time — not editorially configured.

**Canonical reference:** `04_Report_Compiler_Spec`.

---

## Section E — Architectural Blind Spots

### E1 — Facilitator Flag Queue vs. Facilitator Review Layer ✅ RESOLVED

**Resolution:** Two mechanisms serve different purposes:

- **Flag queue (prototype):** Real-time session tool. Surveyors flag observations during the session for Facilitator awareness. Operational.
- **Facilitator review layer (inference spec):** Post-session analytical tool. Analyst reviews low-confidence findings before report inclusion. Analytical.

Sequential, not competing: flags during session → inform Facilitator awareness → feed into post-session review queue → analyst resolves before Analysis Boundary Gate locks.

---

### E2 — Concurrent Observation Conflict Resolution ✅ RESOLVED

**Resolution:** Under P2, conflicting observations from different roles about the same device default to the more conservative (higher violation) record.

**Rules:**
- Both observations are retained in the data model
- The observation supporting the violation (device on) takes precedence over the observation suggesting it was off
- Conflict is flagged for analyst review — noted in the finding with both records cited as evidence
- If one observation has photo support and the other doesn't, photo-supported record takes precedence regardless of direction
- Analyst resolves conflict before Analysis Boundary Gate locks

---

### E3 — Location Anchoring Method ✅ RESOLVED

Resolved in B5 — hybrid input model: pre-loaded room list + free text fallback + post-session normalization. Phone telemetry 🅿️ PARKED.

---

### E4 — Implementation Service Layer 🔄 PENDING

Present in legal pad architecture. No spec, no owner. To be addressed as a separate workstream.

---

### E5 — Decisions Register Location ✅ RESOLVED *(May 2026 sprint)*

**Resolution:** Decisions Register migrated from Google Drive to GitHub `docs/decisions/`. Tier 1 classification retained — owned by Third Switch leadership. GitHub provides version control, diff history, and consistent access patterns with other engineering documentation.

---

## Open and Parked Items

| Item | Description | Status |
|------|-------------|--------|
| EUI methodology source | CBECS as reference or internal system — multiple solutions exist | 🅿️ PARKED |
| Guarantee feasibility model | Pre-evaluation approach needed | 🔄 IN DEVELOPMENT |
| Implementation Service Layer | Present in architecture, no spec, no owner | 🔄 PENDING |
| Phone telemetry for location reconciliation | Passive logging is non-starter due to IT approval / privacy risk; evaluate when platform matures | 🅿️ PARKED |
| `field_state_enum` long-term fate | Option A (optional, current V1 decision) vs. Option B (remove entirely, future consideration) | 🔄 OPEN — do not change without Decision Register entry |
| `"Timer / Schedule"` V2 reintroduction | Requires `Detail` photo + Facilitator or verified facilities-staff attribution | 🔄 OPEN — V2 |
| Exempt Asset kWh estimation methodology | Open before first engagement with confirmed exempt assets | 🔄 OPEN |
| Demand charge peak kWh reduction logic | Open before first demand-charge engagement | 🔄 OPEN |
| Stage 6 scoring matrix | Shelved for V2 | 🅿️ PARKED — after first 3 engagements |
| SEM Compliance Column — Willdan engagement | Open | 🔄 OPEN |
| Catalog version increment threshold | 10% threshold for MAJOR version increment is provisional | 🔄 OPEN — confirm after first catalog maintenance cycle |
| PE new observation promotion path | Deferred pending VOC | 🔄 OPEN — post first PE engagement |
| PE dashboard savings persistence calculation | IE and Measure Catalog dependency unresolved | 🔄 OPEN — before first PE renewal requiring savings true-up |
| README pipeline diagram update | Retire pool references; add PE as downstream overlay | 🔄 PENDING — next harmonization wave |

---

## Documents Requiring Harmonization

The following documents contain outdated terminology or references and are scheduled for update:

| Document | Issue | Priority |
|----------|-------|----------|
| GitHub README pipeline diagram | References retired savings pools; outdated pipeline structure | Next harmonization wave |
| `inference-engine-spec.md` Section 6.1 | Uses retired field names: `did_you_turn_off`, `field_count`, `asset_sub_class` | Next harmonization wave |
| Opportunity Signature Library | Uses `Operational / Coordination / Capital` action path labels — replace with `Green / Yellow / Red` | Next harmonization wave |
| Observability Scope Boundary Framework | Uses `necessary_baseload` — replace with `exempt_asset`; align exception codes with v3.4 five sanctioned reason codes | Next harmonization wave |
| Operational Load Taxonomy | Verify process load framing consistent with `exempt_asset` boundary logic | Next harmonization wave |
| Studio Design v3.4 two-tier architecture section | Uses "Programmatic Tier" — replace with "Engineering Track" | Next harmonization wave |

---

*Deep//Walk by Third Switch — Assurance at Scale*
*© 2026 Third Switch, LLC.*
*This document is internal reference. Not for external distribution without review.*
