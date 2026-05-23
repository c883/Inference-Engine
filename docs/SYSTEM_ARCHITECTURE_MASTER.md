# Deepwalk — System Architecture Master

**Document Identifier:** DW-ARCH-MASTER  
**Version:** 1.0  
**Date:** May 2026  
**Status:** Authoritative — all new and updated documents must cite this document for tier assignment and cross-reference rules.

**Canonical location:** `docs/SYSTEM_ARCHITECTURE_MASTER.md` (GitHub — this is the authoritative version)  
**Operational reference copy:** Google Drive, linked from the V4 Decision Register. If the two copies ever conflict, GitHub is correct.

**Supersedes:** Deepwalk Discovery Studio Design v1 (Part 1 + Part 2 combined). Those two documents are retired. Their content has been distributed into the tier structure defined here.

---

## Purpose

This document defines the complete documentation architecture for the Deepwalk platform. It assigns every current and planned specification document to its tier, states what each tier owns and does not own, and establishes the rules that prevent content from drifting across boundaries.

**If you are a developer new to this codebase, start here.** This document tells you what every spec file owns, where it lives, and what it is allowed to touch. It is intentionally written to be readable by both operators and engineers. Read Part 1 (the tier model) and Part 5 (terminology) before opening any other spec.

Every document in the Deepwalk spec library must open with a reference to this document and declare its tier. When any document is edited, the author checks whether the content being added belongs in that tier. If it does not, it becomes a cross-reference, not a reproduction.

---

## Part 1 — The Four-Tier Model

### Tier 1 — Decision Register

**Location:** Google Drive  
**Document:** Discovery Studio Project Reconciliation — Master Design & Architecture Specification (currently V4)  
**Owner:** Third Switch leadership

**What it owns:**
- Architectural decisions that resolve conflicts between the Conceptual/Operational Track and the Engineering Track
- Schema name locks and canonical field registry decisions
- Stage-level design choices (e.g., Stage 6 deterministic rule tree vs. scoring matrix)
- Output format decisions (e.g., flat array vs. named pools)
- Terminology harmonization rulings

**Rules:**
- Stays short. Decision-only. No content duplication from other tiers.
- Any decision touching both tracks is logged here first, then propagated. Never the other way around.
- Versioned with a letter suffix on major decisions (V4 → V5).
- Is the enforcement point for disputes between documents.

---

### Tier 2 — Conceptual / Operational Track

**Location:** Google Drive  
**Owner:** Third Switch — product, methodology, and operations

**What it owns:**
- Foundational principles (P1–P9)
- The P8 classification decision tree (four gates: Asset Condition, Control Layer, Access, Barrier Nature)
- Domain definitions (Human / Boundary / Mechanical) and action path definitions (Green / Yellow / Red)
- P5 (Domain and Action Path Independence)
- P4 (Classification Describes, Not Assesses — language rules for all documents and reports)
- P6 (Time of Reference)
- P7 (Access and Normal Operational Reach)
- Role definitions, responsibilities, domain/action path biases, team sizing
- Full engagement pipeline (Pre-session through Report Compiler and downstream modules)
- Owl and Bulldog definitions, scope, and inference effects
- Analysis Boundary Gate logic and scope exclusion reason codes (including `exempt_asset`)
- Session Enrichment Context — tiers, objects, pipeline behavior
- Observation Quality Taxonomy (Types 1–7)
- Wattage Fallback Hierarchy (as a formal 4-step lookup rule)
- Confidence model and priority scoring framework
- Facilitator Review Layer — mandatory triggers, available actions, audit trail rules
- Open items log

**What it does not own:**
- JSON schemas, field name definitions, or enum value lists → Tier 3
- Error codes → Tier 3
- Stage pseudocode or cascade logic → Tier 3
- API endpoint definitions → Tier 3
- Module-specific pipeline logic → Tier 4

**Cross-reference rule:** When a Tier 2 document references a schema field, it uses the canonical name from Tier 3 (`01_Discovery_Studio_Config_Spec`) and does not redefine it.

---

### Tier 3 — Engineering Track

**Location:** GitHub (`c883/Inference-Engine`, `docs/` directory)  
**Owner:** Third Switch — engineering

**What it owns:**
- Canonical `ObservationNode` schema with all required fields, data types, and validation rules
- All field name canonical registry (the locked names from V4 Section 2)
- `why_not` enum values (the six permitted strings + null)
- `field_state_enum` values (`ON`, `OFF`, `AUTO`)
- `surveyor_role` enum values (`M3`, `LO`, `TV`, `SS`, `Facilitator`)
- `out_of_scope_reason` enum values (five sanctioned codes)
- Error codes (`ERR_` prefix system)
- 10-stage pipeline pseudocode and stage-level input/output schemas
- Stage 2 cascade fallback logic (Table A / Table B / Appendix A / Global Catch-All)
- Stage 6 deterministic rule tree (locked per V4)
- Stage 10 flat array output contract with boolean flags (`is_guaranteed_saving`, `is_exempt_baseline`)
- `portfolio_spend_cap_kwh` as the canonical portfolio cap field (loaded Stage 2)
- API endpoint definitions, payload contracts, validation constraint logic
- Spatial model contract (space_id, sticky map mechanic, forensic timestamp sequencing)
- Data dictionary and immutability matrix
- Centralized Control Assumption Defaults Matrix (Appendix A)
- Engineering design principles: Zero Implicit Inference, Fail-Fast Pipeline Isolation, Immutable Data Lineage

**What it does not own:**
- P-principles (P1–P9) → Tier 2
- Gate logic (P8 decision tree) → Tier 2
- Domain or action path definitions → Tier 2
- Role descriptions or methodology → Tier 2
- Module pipeline logic (Report Compiler, Persistence Engine, Implementation Support) → Tier 4

**Cross-reference rule:** When a Tier 3 document references classification logic (e.g., why a certain `why_not` value routes to a given stage), it cites Studio Design v3.4 Section P8 and does not restate the gate logic.

---

### Tier 4 — Module Specs

**Location:** Google Drive (conceptual/operational sections) + GitHub `docs/modules/` (engineering sections), as appropriate per module  
**Owner:** Third Switch — assigned per module

**Characteristics:**
- Each module spec is self-contained and owns its own pipeline, data objects, and future libraries
- References up to Tier 2 for principles and classification logic
- References up to Tier 3 for schema field names and data contracts
- Does not duplicate content from either track

**Current modules:**

| Module | Status | Notes |
|--------|--------|-------|
| Report Compiler | Draft exists — needs full spec | Renders Tier 3 flat array output into named finding pools for PDF |
| Measure Catalog Spec (`03_Measure_Catalog_Spec`) | Spec structure exists — needs completion | Translation spec: defines ingestion rules, mapping logic, and normalization process from external sources (DEER, manufacturer data, cut sheets, ENERGY STAR, DLC, ASHRAE) into Deepwalk catalog entries. Owns equipment_class taxonomy, version control system (Draft → Commit → Release), wattage library. The catalog itself is a separate living document in the Reference Library. |
| Implementation Support | In progress | Downstream of Report Compiler; maps report outputs to customer systems and documentation; will have its own libraries |
| Persistence Engine | Draft exists — needs harmonization | Longitudinal engagement layer; standalone CMMS-lite; annual subscription |

**Architectural note on Implementation Support:** This is a contractual engagement module and a planned future engine. The term "implementation" in this module name refers to the customer's operational implementation of report findings — it is not the same as the Engineering Track (Tier 3). These must not be conflated in naming or folder structure.

---

### Reference Library

**Location:** Google Drive  
**Owner:** Third Switch — shared

**Characteristics:**
- Cross-referenced from any tier
- Not owned by any single track
- Updated when engagement data warrants expansion

**Current documents:**

| Document | Version | Status |
|----------|---------|--------|
| Opportunity Signature Library | v1.0 | Exists — needs action path language harmonization |
| Observability Scope Boundary Framework | v1.0 | Exists — needs `exempt_asset` terminology update |
| Operational Load Taxonomy | v1.0 | Exists — minimal changes needed |

---

## Part 2 — Full Document Registry

Every current and planned document in the Deepwalk spec library, its tier, its location, and its status.

### Tier 1

| Document | Location | Version | Status |
|----------|----------|---------|--------|
| Master Design & Architecture Specification | Drive | V4 | Current |

### Tier 2 — Conceptual / Operational Track

| Document | Location | Version | Status |
|----------|----------|---------|--------|
| Studio Design (Discovery Studio Design Spec) | Drive | v3.4 | Current — authoritative |
| Performance Map / Brief | Drive | v4 | Current — authoritative |
| Enrichment Review & Technical Confirmation Spec | Drive | Revised draft | Current — needs P9, version number |
| `08_Surveyor_Roles` | Drive | v1 draft | Needs role harmonization with v3.4 |
| `09_Scoping_Ingestion` | Drive | v1.4 draft | Needs Tier A/B/C alignment with v3.4 |
| `07_Review_Workflow_Spec` | Drive | v1 draft | Needs V4 role boundary update |
| System Architecture Master (this document) | Drive → GitHub | v1.0 | New |

### Tier 3 — Engineering Track

| Document | Location | Version | Status |
|----------|----------|---------|--------|
| Inference Engine Spec | GitHub `docs/architecture/inference-engine-spec.md` | v3.1 Production | Current — authoritative |
| `01_Discovery_Studio_Config_Spec` | GitHub `docs/` | Needs rewrite | V4 schema not yet reflected |
| `05_API_Contracts` | GitHub `docs/` | Skeleton only | Needs full V4 ObservationNode contract |
| `06_Spatial_Model_Spec` | GitHub `docs/` | Skeleton only | Needs sticky map + forensic timestamp spec |

### Tier 4 — Module Specs

| Document | Location | Version | Status |
|----------|----------|---------|--------|
| `04_Report_Compiler_Spec` | Drive + GitHub | Skeleton only | Needs full section spec + flat array rendering logic |
| `03_Measure_Catalog_Spec` | Drive + GitHub | Structure exists | Translation spec — needs version control, wattage hierarchy, taxonomy registry, external source mapping rules |
| `10_Persistence_Engine_Spec` | Drive | Draft exists | Needs role language harmonization (V4) |
| `Implementation_Support_Spec` | Drive | v1.0 exists; v1.1 planned | Needs v1.1 update |

### Reference Library

| Document | Location | Version | Status |
|----------|----------|---------|--------|
| Opportunity Signature Library | Drive | v1.0 | Needs action path language update |
| Observability Scope Boundary Framework | Drive | v1.0 | Needs `exempt_asset` terminology update |
| Operational Load Taxonomy | Drive | v1.0 | Minor updates only |
| Measure Catalog | Drive / TBD | v0 (empty) | Living library — populated through engagements by applying `03_Measure_Catalog_Spec`; starts empty and accumulates over time |

---

## Part 3 — Production Wave Plan

Documents are written and updated in dependency order. Wave 1 must be complete before Wave 2 begins, because Wave 2 documents cite Wave 1 documents for field names, role definitions, and schema contracts.

### Wave 1 — Foundation

These documents define the terms that everything else references. They must be written or updated first.

| # | Document | Tier | Primary changes |
|---|----------|------|----------------|
| 1 | System Architecture Master (this document) | 2 | New — written first |
| 2 | `08_Surveyor_Roles` | 2 | Align role attributes with v3.4; add Bulldog quick-capture; confirm V4 role enums |
| 3 | `01_Discovery_Studio_Config_Spec` | 3 | Full rewrite to V4 canonical ObservationNode schema; correct all field names; remove condition_enum from field form; add field_state_enum [ON/OFF/AUTO]; correct photo slot names |

### Wave 2 — Core Specs

Depend on Wave 1 for role definitions (08) and field schema (01).

| # | Document | Tier | Primary changes |
|---|----------|------|----------------|
| 4 | `06_Spatial_Model_Spec` | 3 | Add sticky map mechanic; forensic timestamp sequencing; space_id as canonical token; ad-hoc anchor handling |
| 5 | `05_API_Contracts` | 3 | Full V4 ObservationNode in each endpoint; why_not enum values; error codes; Bulldog endpoint; Stage 10 flat array output |
| 6 | `09_Scoping_Ingestion` | 2 | Align Tier A/B/C with v3.4 Tier structure; reference SessionEnrichmentContext and AnalysisBoundary objects |
| 7 | `07_Review_Workflow_Spec` | 2 | Update: Facilitator software loop ends at session close per V4; Analyst/BD owns Stage 6.5; align resolution states with Enrichment spec |
| 8 | `03_Measure_Catalog_Spec` | 4 | Add version control system; equipment_class taxonomy registry; wattage fallback hierarchy as formal spec; DEER/ENERGY STAR/DLC integration |

### Wave 3 — Downstream Modules and Reference Library

Depend on Wave 2 for report format (05), measure structure (03), and role resolution (07).

| # | Document | Tier | Primary changes |
|---|----------|------|----------------|
| 9 | `04_Report_Compiler_Spec` | 4 | Full section specs; flat array → named pool rendering logic; finding pool presentation; P4 language rules for narrative |
| 10 | `10_Persistence_Engine_Spec` | 4 | Role language harmonization (Facilitator verification in PE is separate from post-session software loop); align with V4 |
| 11 | `Implementation_Support_Spec_v1.1` | 4 | Update v1.0; document downstream position relative to Report Compiler; scope of customer system mapping; future library plan |
| 12 | Opportunity Signature Library | Ref | Action path language: replace Operational/Coordination/Capital with Green/Yellow/Red per v3.4; verify P5 alignment |
| 13 | Observability Scope Boundary Framework | Ref | Replace `necessary_baseload` with `exempt_asset` throughout; align exception codes with v3.4 five sanctioned reason codes |
| 14 | Operational Load Taxonomy | Ref | Verify process load framing consistent with exempt_asset boundary logic; minor language alignment |

---

## Part 4 — Governing Rules

These rules apply to every document in the Deepwalk spec library. They are enforced by the Tier 1 Decision Register.

**Rule 1 — Every document declares its tier.**
The document header states its tier, what it owns, and its primary cross-references. No exceptions.

**Rule 2 — Content belongs to one tier.**
If content belongs in a different tier, it is replaced with a cross-reference. Content is never reproduced across tiers.

**Rule 3 — Decisions flow downward from Tier 1.**
Any decision touching both Tier 2 and Tier 3 is logged in the Decision Register first. It then propagates into the relevant tier documents. Documents do not make decisions — they implement them.

**Rule 4 — Field names are locked in Tier 3.**
The canonical field name registry lives in `01_Discovery_Studio_Config_Spec` (Tier 3). All other documents use the locked names. If a name needs to change, it is logged as a decision in Tier 1 first.

**Rule 5 — Principles are locked in Tier 2.**
P1–P9 live in Studio Design v3.4. No other document restates or redefines a P-principle. References are by citation only: "per P4" or "see Studio Design v3.4 Part 1 P7."

**Rule 6 — The Engineering Track does not explain why.**
Tier 3 documents implement classification and routing logic. They do not explain the analytical reasoning behind it. That reasoning lives in Tier 2. A Tier 3 document says "per P8 Gate 3, route to Stage 8" — it does not restate what Gate 3 means.

**Rule 7 — Module Specs cite both tracks.**
Tier 4 documents reference Tier 2 for principles and Tier 3 for schemas. They never reproduce content from either.

**Rule 8 — The Reference Library is stable.**
Reference library documents are updated only when engagement data or new decisions require it. They do not track sprint-level changes.

---

## Part 5 — Canonical Terminology Quick Reference

The following terms are locked. All documents use these exact forms.

| Canonical Term | Retired / Variant Terms | Notes |
|----------------|------------------------|-------|
| `exempt_asset` | `necessary_baseload` | Scope exclusion reason code; locked in v3.4 and IE V3.1 |
| `did_you_turn_it_off` | `did_you_turn_off` | Field name; locked in V4 Section 2 |
| `fixture_count` | `field_count` | Field name; locked in V4 Section 2 |
| `observation_type` | `asset_sub_class`, `asset_type_enum`, `Measure Type`, `Measure Category` | UI triage key; maps to `equipment_class` in backend |
| `space_id` | `room_id`, `space_function` | Canonical location token; backed by sticky map mechanic |
| `field_state_enum` | `state` | Values: `ON`, `OFF`, `AUTO` |
| `why_not` | `why_not_enum` | Six permitted values + null; defined in Tier 3 |
| Green / Yellow / Red | Immediate / Coordination / Capital (as action path labels in software) | Action path colors are the canonical output labels; Immediate/Coordination/Capital are retained as human-readable descriptors in Tier 2 documents and reports |
| Human / Boundary / Mechanical | — | Domain names; unchanged across all documents |
| Conceptual Tier | — | The analytical/business logic layer described in Tier 2 (P8 gates) |
| Engineering Track | Programmatic Tier (retired label) | The executable software pipeline in GitHub (Tier 3). "Programmatic Tier" appears in existing documents (GitHub README, v3.4 two-tier architecture section) and must be updated to "Engineering Track" as each of those documents is harmonized in its respective wave. |
| Implementation Support | — | Contractual module; downstream of Report Compiler; not the same as "Engineering Track" |
| Analyst / BD | Facilitator (for post-session software tasks) | Post-session software loop owner per V4 |
| Photo slots: Context / Subject / Detail | Device Photo, Nameplate/Closeup | Slot names locked in v3.4 |

---

## Part 6 — Open Items Inherited at Architecture Level

The following items are flagged at the architecture level and propagate into the relevant tier documents as open items.

| # | Item | Tier | Target |
|---|------|------|--------|
| 1 | Stage 6 scoring matrix — shelved for V2; calibration target after first 3 engagements | 3 | Post-engagement |
| 2 | Exempt Asset kWh estimation methodology — how Facilitator estimates annual kWh load for boundary deduction | 2 + 3 | Before first engagement with confirmed exempt assets |
| 3 | Demand charge peak kWh reduction logic — Stage 9 references it; calculation not yet specified | 3 | Before first demand-charge engagement |
| 4 | Access Tier field — future parallel descriptor alongside domain and action path | 2 | V2 |
| 5 | ML-assisted inference — V1 is fully rules-based; portfolio ML is a future capability | 3 | Post-portfolio data accumulation |
| 6 | UNCONTROLLED field state vocabulary — Stage 1 normalization keyword list; formal review after first 5 engagements | 3 | Post-engagement |
| 7 | SEM Compliance Column — Willdan engagement requirement; CA SEM Guide SSA and EM number mapping | 2 | Before next SEM engagement |
| 8 | Implementation Support future libraries — scope TBD as module matures | 4 | In progress |
| ~~9~~ | ~~`02_Measure_Catalog` — clarify whether this is the catalog spec (Tier 4) or actual catalog entries (living reference)~~ | ~~4~~ | **Resolved:** `02_Measure_Catalog` is the living catalog library (Reference Library, starts empty, populated through engagements). `03_Measure_Catalog_Spec` is the Tier 4 translation spec that governs how external sources are ingested. These are distinct deliverables. |

---

*Deep//Walk by Third Switch — Assurance at Scale*  
*© 2026 Third Switch, LLC.*  
*This document is internal reference. Not for external distribution without review.*
