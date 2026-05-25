# Observability & Scope Boundary Framework

**Document Identifier:** DW-REF-OSBF
**Type:** Reference Library
**Location:** GitHub `docs/reference/Observability_Scope_Boundary_Framework.md`
**Version:** 1.1
**Date:** May 2026
**Supersedes:** Observability & Scope Boundary Framework v1.0 (Studio Design v3.4 embedded section)

**Architecture reference:** See `docs/SYSTEM_ARCHITECTURE_MASTER.md` for tier definitions and cross-reference rules.

**What this document contains:** The observability boundary framework — what can and cannot be assessed during a Discovery Session, how non-observable systems are handled, exception categories, scope exclusion reason codes, the `exempt_asset` definition, and standardized inference logic protocols for indirectly observable systems.

**What this document does not contain:** Session schema and field definitions — those live in `docs/01_Discovery_Studio_Config_Spec.md` (Tier 3). Inference Engine stage logic — that lives in `docs/architecture/inference-engine-spec.md` (Tier 3). Analysis Boundary Gate logic — Tier 3. P-principles — Studio Design v3.4 (Tier 2).

**Version 1.1 changes from v1.0:** `necessary_baseload` replaced with `exempt_asset` throughout. Exception category and scope exclusion reason codes aligned with V4 architecture. Document extracted from Studio Design embedded section and established as a standalone Reference Library file. Open item noted for five sanctioned reason codes — see Part 6.

---

## Part 1 — Core Principle: Observability

Deepwalk findings are derived from observable operational behavior. The Discovery Session does not depend on engineering analysis during the survey — engineering analysis occurs afterward, using structured inference logic.

Observable operational behavior includes:
- Lighting operation and control state
- Plug loads and equipment idle conditions
- Control overrides and schedule indicators
- Occupant behavior and space use patterns
- Distributed HVAC behavior visible from occupied spaces
- Equipment condition signals accessible during the session

The boundary between what is and is not observable during a session is not a failure of the method — it is a structural property of the approach. This framework defines that boundary and establishes how non-observable systems are handled.

---

## Part 2 — Observability Categories

Every system or load encountered during a Discovery Session falls into one of three categories:

| Category | Description | Treatment |
|----------|-------------|-----------|
| **Directly Observable** | Systems visible, operating, and interpretable during the session window | Fully analyzed — observations feed the Inference Engine directly |
| **Indirectly Observable** | Systems partially visible but not directly measurable; reasonable inference is possible | Inference modeling using standardized protocols (see Part 5) |
| **Non-Observable** | Systems inaccessible, inactive, or outside the session scope | Exception documented — see Part 3 |

### Directly observable systems — typical examples
- Interior and task lighting
- Plug loads and distributed electronics
- Office equipment idle conditions
- Lighting control override indicators
- HVAC terminal behavior visible in occupied zones
- Equipment condition signals (cleanliness, nameplate, operating state)

### Indirectly observable systems — typical examples
- Air handling equipment serving observed zones (inferred from terminal behavior)
- Lighting circuits (inferred from fixture type and zone pattern)
- Equipment schedules (inferred from observed operating state and BAS indicators)
- Plug loads (inferred from equipment cluster composition)

### Non-observable systems — typical examples
- Locked mechanical rooms and roof equipment
- Equipment not operating during the session window
- Tenant-controlled equipment in restricted areas
- Specialized process systems
- Central plant equipment without accessible indicators

---

## Part 3 — Exception Categories

When a system or space cannot be assessed, the exception is documented — never silently dropped. Per P3 (Turn Over Every Stone), every observation has a disposition in the output record.

Four exception categories are defined:

| Exception Type | Description | Disposition |
|----------------|-------------|-------------|
| **Access Exception** | System or space inaccessible during session — locked, restricted, or physically unreachable | `excluded_spaces[]` entry with reason code; contributes to coverage record; savings guarantee adjusted at Analysis Boundary Gate |
| **Operating State Exception** | Equipment not running during observation window — cannot confirm operating state | Retained as coverage record; `field_state_enum = UNKNOWN`; Facilitator notes in `raw_notes` |
| **Process Exception** | System driven by a specialized operational process — see `exempt_asset` definition in Part 4 | `exempt_asset` flag applied; observation retained; excluded from savings calculations per sanctioned reason code |
| **Control Boundary Exception** | System controlled by a third party or tenant — Deepwalk cannot modify or attribute the load | `out_of_scope` flag with `boundary_exclusion` reason code; retained in data record |

---

## Part 4 — exempt_asset Definition

`exempt_asset` is the canonical term for any load that is observed but excluded from Deepwalk savings calculations because it is operationally necessary, process-driven, or outside the correctable boundary.

**Replaces:** `necessary_baseload` (retired — do not use)

### What qualifies as an exempt_asset

An asset is marked `exempt_asset` when the Analyst confirms, post-session, that the load meets one or more of the following conditions:

- **Operationally required** — the load must remain energized to support a regulated, safety, or continuous operational requirement (e.g., egress lighting, minimum ventilation, cold storage compressors)
- **Process-driven** — the load is governed by a specialized operational process outside standard building operations (e.g., commercial kitchen equipment, laboratory equipment, manufacturing loads)
- **Tenant-controlled** — the load is within the tenant's exclusive control and outside the building operator's authority

### What exempt_asset does not mean

- An asset is not automatically `exempt_asset` because it is in continuous operation — continuous operation may be the finding
- An asset is not `exempt_asset` because the surveyor could not turn it off — that is a `did_you_turn_it_off = FALSE` / `why_not` record, not an exemption
- The `exempt_asset` designation is a post-session Analyst determination, not a field-level flag

### Reporting treatment

`exempt_asset` findings appear in the report with their operating condition documented and their exemption reason stated. They do not contribute to savings totals. Per P3, they are not silently omitted.

---

## Part 5 — Scope Exclusion Reason Codes

When an observation is excluded from savings calculations during the post-session scope exclusion pass at the Analysis Boundary Gate, the following reason codes are applied:

| Reason Code | Meaning |
|-------------|---------|
| `dead_circuit` | Device plugged in or connected but not drawing power; circuit confirmed dead or device confirmed non-functional |
| `boundary_exclusion` | Load powered outside the project energy boundary — separate tenant meter, separate building feed, construction power |
| `temporary_load` | Truly temporary load — vendor performing a short-term task, event equipment, construction equipment not permanent to the site |
| `access_denied_at_event` | Space or system access was denied during the session after it was expected to be accessible — triggers contract escalation process |

**Open item:** The Decisions Register referenced "five sanctioned reason codes" as a target for this framework. The four codes above represent the complete set currently defined. A fifth code has not been formally specified. This is noted as an open item — any additional reason code requires a Tier 1 Decision Register entry before addition. See `docs/decisions/DECISIONS_REGISTER.md`.

Excluded observations are not deleted. They remain in the data record with their reason code, contribute to the coverage record, and appear in the report with a documented explanation per P3.

---

## Part 6 — Specialized Process Loads

Process-driven loads require a distinct inference lens. They are observable but typically not optimizable within the standard Deepwalk scope without separate engineering analysis.

**Operational definition:** A load is process-driven when its operating pattern is governed by an operational requirement — production schedule, food safety standard, laboratory protocol, or similar — rather than building occupancy patterns.

**Treatment:**
- Observed and documented during the session
- `exempt_asset` flag applied post-session by Analyst if confirmed process-driven
- Operational context recorded in `raw_notes`
- Opportunities may be noted in the report (e.g., equipment condition findings, scheduling gaps relative to process windows)
- Savings from process load optimization excluded from the primary guarantee calculation unless a specific correctable fraction can be isolated

**Examples by type:**
- Commercial kitchen equipment — food safety temperature requirements constrain shutdown options; equipment condition findings (coil cleanliness, gasket integrity) are valid findings regardless
- Laboratory equipment — minimum ACH and temperature requirements constrain HVAC; specific equipment on independent circuits may still be addressable
- Cold storage compressors — continuous operation required; strip curtain integrity, gasket condition, and coil cleanliness are addressable findings
- Manufacturing and production loads — process schedule governs runtime; scheduling alignment opportunities may exist at shift boundaries

---

## Part 7 — Standard Inference Logic for Indirectly Observable Systems

When systems are indirectly observable, the Inference Engine applies the following standardized protocols. These protocols do not override direct observation — they supplement it when direct evidence is unavailable.

### Load Pattern Inference
Observed equipment clusters used to estimate plug loads. A workstation cluster of known composition is mapped to average workstation load profiles from the Measure Catalog. Applied when individual nameplate data is unavailable.

### Schedule Inference
Observed occupant activity and system state used to estimate runtime patterns. Lights operating in an unoccupied zone → extended schedule assumption applied under P2 (Observed Violation Persistence).

### Control Override Inference
Manual override indicators or override inferred from control behavior inconsistent with expected schedule. Override present → extended operating hours assumption applied.

### HVAC Behavior Inference
Thermal conditions and airflow patterns used to infer scheduling or control behavior. Conditioned zones with no occupancy and no visible override → extended HVAC schedule assumption applied.

**Governing principle:** Inference modeling supplements observation. Observation-based inference under P2 is always the floor — sensor data and BAS records refine the estimate but do not replace it.

---

## Part 8 — Engineering Assumption Protocols

When inference modeling is required, standardized assumptions are applied. All assumptions must be:

- Documented in the Inference Summary (report Appendix A3)
- Sourced from the Measure Catalog (`02_Measure_Catalog`) or named reference standard
- Consistent across engagements using the same `catalog_version`
- Transparent in reporting — disclosed with source and confidence level

Assumption sources in priority order follow the wattage fallback hierarchy defined in `docs/modules/03_Measure_Catalog_Spec.md` Part 7.

---

## Version Notes

Version 1.0 established the initial observability and scope boundary framework.
Version 1.1 replaced `necessary_baseload` with `exempt_asset`, aligned exception codes with V4 architecture, and established this as a standalone Reference Library file.

---

*Deep//Walk by Third Switch — Assurance at Scale*
*© 2026 Third Switch, LLC.*
*This document is internal reference. Not for external distribution without review.*
