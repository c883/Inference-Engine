# Operational Load Taxonomy

**Document Identifier:** DW-REF-OLT
**Type:** Reference Library
**Location:** GitHub `docs/reference/Operational_Load_Taxonomy.md`
**Version:** 1.1
**Date:** May 2026
**Supersedes:** Operational Load Taxonomy v1.0 (Studio Design v3.4 embedded section)

**Architecture reference:** See `docs/SYSTEM_ARCHITECTURE_MASTER.md` for tier definitions and cross-reference rules.

**What this document contains:** The canonical categories of energy use evaluated during a Deepwalk Discovery Session. The taxonomy defines what Deepwalk observes, how each load category is treated in the inference pipeline, and the boundary between primary Deepwalk targets and loads that fall outside the standard optimization scope. Evidence classification rules are also defined here.

**What this document does not contain:** Schema definitions — those live in `docs/01_Discovery_Studio_Config_Spec.md` (Tier 3). `equipment_class` taxonomy — that lives in `docs/modules/03_Measure_Catalog_Spec.md` (Tier 4). `exempt_asset` definition — that lives in `docs/reference/Observability_Scope_Boundary_Framework.md`.

**Version 1.1 changes from v1.0:** Process load framing updated to reference `exempt_asset` explicitly and align with the Observability Scope Boundary Framework. Document extracted from Studio Design embedded section and established as a standalone Reference Library file.

---

## Purpose

The Operational Load Taxonomy defines the categories of energy use evaluated during a Deepwalk Discovery Session. It supports:

- Surveyor observation structure and role assignment
- Reporting consistency across engagements
- Inference Engine routing logic
- Scalable data aggregation across buildings and portfolios

The taxonomy distinguishes between:
1. **Operational loads** — primary Deepwalk targets
2. **Operational control layers** — primary Deepwalk targets
3. **Process-driven loads** — outside the standard optimization scope; handled via `exempt_asset` designation

Only the first two categories are primary Deepwalk savings calculation targets.

---

## Category Structure

| # | Category | Description | Primary Evaluation Method |
|---|----------|-------------|--------------------------|
| 1 | Lighting Systems | Interior and task lighting behavior | Direct observation |
| 2 | Plug & Equipment Loads | Distributed electronics and equipment | Direct observation |
| 3 | Operational Controls | Schedules, overrides, control behavior | Observation + inference |
| 4 | Occupant Behavior | Human patterns affecting energy use | Observation |
| 5 | Central System Behavior | HVAC or building systems affecting zones | Observation + inference |

---

## Category 1 — Lighting Systems

Lighting loads are among the most consistently observable operational loads. Direct observation yields high-confidence findings in the majority of Discovery Sessions.

**Typical observations:**
- Lights operating in unoccupied areas
- Override behavior visible at switch or control panel
- Fixture types and wattage class identified (nameplate, Detail photo)
- Perimeter daylight interaction — daylight harvesting functioning or not
- Task lighting usage relative to ambient levels

**Inference engine routing:** Lighting observations feed directly to equipment class matching in Stage 2, wattage lookup against the Measure Catalog, and domain / action path scoring. High-confidence findings in the Human and Boundary domains are the most common output.

**Primary associated signatures:** SIG-001 (Lighting Active in Low-Occupancy Zones), SIG-002 (Lighting Controls Not Aligned with Space Use), SIG-006 (Extended Operating Schedules)

---

## Category 2 — Plug & Equipment Loads

Plug loads represent distributed operational energy consumption across the building. They are highly variable by space type and occupant density.

**Typical observations:**
- Workstations and monitors energized after hours
- Printers, copiers, and AV systems in idle or active state
- Breakroom appliances — coffee machines, microwaves, refrigerators
- Vending machines — lighting and compressor state observable
- Local equipment clusters — count and estimated class recorded

**Observable signals:**
- Idle equipment (powered but not in active use)
- Overnight or after-hours energized state
- Visible power indicator lights (phantom loads)
- Equipment cleanliness as utilization proxy (see Observability Scope Boundary Framework Part 6)

**Inference engine routing:** Plug load observations feed equipment class matching, wattage lookup, and cluster-level pattern recognition. Human domain findings are common; Boundary findings emerge when the pattern is systemic across zones.

**Primary associated signatures:** SIG-003 (Persistent Workstation Plug Loads), SIG-004 (Peripheral Equipment Idle Operation), SIG-007 (Breakroom / Appliance Continuous Operation)

---

## Category 3 — Operational Controls

Operational controls determine how systems behave over time. They are not loads themselves — they govern the runtime patterns of lighting and HVAC loads. Control observations often produce the highest-value findings because they affect annualized hours across many fixtures or zones.

**Typical observations:**
- Lighting schedules — programmed vs. actual operating state
- Manual overrides — override indicators visible at switch, panel, or BAS display
- BAS scheduling patterns — schedule display accessible during session
- Local thermostat adjustments — setpoint and schedule visible
- Equipment timers — timer setting observable on device

**Inference engine routing:** Control observations feed schedule inference logic. A confirmed override extends the annualization assumption under P2 (Observed Violation Persistence). A confirmed functioning schedule provides a schedule ceiling for Stage 2 calculation refinement.

**Primary associated signatures:** SIG-002 (Lighting Controls Not Aligned), SIG-006 (Extended Operating Schedules)

---

## Category 4 — Occupant Behavior

Human behavior drives a significant portion of operational energy use. Behavior observations are typically the fastest to capture and the fastest to correct — they feed directly into Green action path findings.

**Typical observations:**
- Occupancy patterns — who is present, where, and when
- Personal device use — laptops, monitors, chargers left on
- Equipment left operating — shutdown discipline observable directly
- Space usage patterns — rooms occupied vs. unoccupied relative to system state
- Behavioral overrides — occupant-controlled adjustments to thermostats, lighting, equipment

**Inference engine routing:** Behavior observations feed Human domain scoring directly. `did_you_turn_it_off = TRUE` is a strong Human / Green signal. Patterns across multiple zones elevate confidence.

**Primary associated signatures:** SIG-003 (Persistent Workstation Plug Loads), SIG-007 (Breakroom / Appliance Continuous Operation)

---

## Category 5 — Central System Behavior

Some building system behaviors are visible through occupied spaces without direct access to mechanical equipment. These observations produce indirectly observable findings that require inference modeling.

**Typical observations:**
- HVAC operation in unoccupied zones — airflow, thermal conditions, thermostat state
- Temperature conditions indicating active conditioning — warm or cool zones relative to session time and occupancy
- Airflow patterns — diffuser activity observable from occupied spaces
- Conditioning of unused areas — storage, secondary lobbies, low-use zones actively conditioned

**Inference engine routing:** Central system observations feed HVAC inference rules in Stage 4 (envelope and zone behavior). They produce Boundary and Mechanical domain findings. Findings require inference modeling rather than direct calculation — engineering assumptions from the Measure Catalog are applied.

**Primary associated signatures:** SIG-005 (HVAC Operating in Low-Occupancy Zones), SIG-008 (Conditioning of Unused Spaces)

---

## Process-Driven Loads and exempt_asset

Certain loads are process-driven rather than building-driven. They are operationally specialized and outside the standard Deepwalk optimization scope.

**Examples:**
- Commercial kitchen equipment — food safety requirements govern operating parameters
- Laboratory equipment — minimum ACH, temperature, and humidity requirements apply
- Manufacturing and industrial process equipment — production schedules govern runtime
- Specialized refrigeration systems tied to operational processes
- Data center IT loads — uptime requirements govern power state

**Treatment:** Process-driven loads are observed and documented during the session. Post-session, the Analyst applies the `exempt_asset` designation when the load is confirmed process-driven. `exempt_asset` observations are:

- Retained in the data record — never silently dropped (P3)
- Excluded from primary savings calculations
- Reported with operating condition documented and exemption reason stated
- Eligible for condition-based findings (equipment cleanliness, gasket integrity, coil condition) that fall within the Deepwalk scope regardless of process designation

For the full `exempt_asset` definition, designation criteria, and reporting treatment, see `docs/reference/Observability_Scope_Boundary_Framework.md` Part 4.

---

## Evidence Classification

Each finding is classified by the evidence type that supports it. This classification is recorded in the observation record and surfaced in the Inference Summary (report Appendix A3).

| Evidence Type | Definition | Confidence effect |
|---------------|------------|-------------------|
| **Observed** | Directly witnessed condition — surveyor confirmed the operating state during the session | High confidence baseline |
| **Inferred** | Derived from standardized inference logic and Measure Catalog assumptions — not directly measured | Moderate confidence; assumption source disclosed |
| **Exception** | Outside Discovery Session scope — access, operating state, process, or control boundary exception | No savings calculation; documented with exception type |

This classification maintains transparency between observation, interpretation, and scope limits. It enables the Inference Summary to be forensically examinable — every savings number is traceable to its evidence type and assumption source.

---

## Relationship to the Deepwalk Architecture

This taxonomy ensures that Deepwalk findings remain:

- **Operationally focused** — findings describe conditions that affect operational energy use, not capital asset condition alone
- **Consistent across buildings** — the same five categories apply regardless of building type or vintage
- **Scalable for portfolio analytics** — consistent category tagging enables cross-engagement analysis

The taxonomy is the foundation for:
- Surveyor observation categories and role-based domain assignment (`08_Surveyor_Roles`)
- Report section structure (`04_Report_Compiler_Spec`)
- Inference Engine equipment class routing (`docs/architecture/inference-engine-spec.md`)
- Long-term portfolio pattern analysis

---

## Version Notes

Version 1.0 established the foundational operational categories.
Version 1.1 updated process load framing to reference `exempt_asset` explicitly, aligned with the Observability Scope Boundary Framework, and established this as a standalone Reference Library file.

Future versions may incorporate:
- Expanded load subclasses by building type
- Standardized inference rules per category
- Quantified assumption libraries per `equipment_class`
- Aggregated dataset structures for portfolio benchmarking

---

*Deep//Walk by Third Switch — Assurance at Scale*
*© 2026 Third Switch, LLC.*
*This document is internal reference. Not for external distribution without review.*
