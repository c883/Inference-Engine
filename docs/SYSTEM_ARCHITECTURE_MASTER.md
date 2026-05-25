# System Architecture Master

**Document Identifier:** DW-MASTER
**Tier:** 3 — Engineering Track (governing document)
**Location:** GitHub `docs/SYSTEM_ARCHITECTURE_MASTER.md` (canonical)
**Version:** 1.1
**Date:** May 2026
**Supersedes:** Version 1.0

**Purpose:** This is the index document for the entire Deepwalk documentation library. Every spec document must open with a tier declaration and cite this document. If a discrepancy exists between this document and any other source, raise a Tier 1 Decision Register entry to resolve it — do not implement the discrepancy.

---

## Part 1 — The Four-Tier Model

All Deepwalk documentation is organized into four tiers with fixed locations and owners.

| Tier | Name | Location | Owner | Format |
|------|------|----------|-------|--------|
| 1 | Decision Register | GitHub `docs/decisions/` | Third Switch leadership | `.md` |
| 2 | Conceptual / Operational Track | Google Drive | Product, methodology, ops | Google Docs |
| 3 | Engineering Track | GitHub `docs/` | Engineering | `.md` |
| 4 | Module Specs | GitHub `docs/modules/` | Assigned per module | `.md` |

**Reference Library** — shared, not owned by any tier. GitHub `docs/reference/`. Version controlled alongside the Engineering Track. See Part 5.

The governing principle: the tier a document belongs to determines its location, format, and update cadence. Documents do not move between tiers without a Tier 1 Decision Register entry.

---

## Part 2 — Tier 1: Decision Register

**Location:** GitHub `docs/decisions/`
**Owner:** Third Switch leadership
**Canonical file:** `docs/decisions/DECISIONS_REGISTER.md`

The Decision Register records all architectural decisions that govern the Deepwalk system. Decisions are immutable once resolved. Open and parked items are tracked in the same document.

Every new architectural decision — schema changes, terminology changes, pipeline modifications, document location changes — requires a Decision Register entry before implementation.

---

## Part 3 — Tier 2: Conceptual / Operational Track

**Location:** Google Drive
**Owner:** Product, methodology, operations
**Format:** Google Docs

Tier 2 documents define the operational framework, role definitions, and methodology that govern how Deepwalk is executed. They are written for product owners, methodology leads, and field operations — not for engineers implementing the system.

**Tier 2 documents do not own schemas, field names, or enum values.** Those are owned by Tier 3 and Tier 4 documents. When a Tier 2 document references a field name or enum value, it cites the canonical Tier 3 or Tier 4 source.

### Tier 2 document index

| Document | File | Version | Notes |
|----------|------|---------|-------|
| Surveyor Roles | `08_Surveyor_Roles` | 2.0 | Role definitions, Bulldog quick-capture, domain/action path bias |
| Scoping + Ingestion | `09_Scoping_Ingestion` | 2.0 | Tier A/B/C input framework, SessionEnrichmentContext object |
| Review Workflow Spec | `07_Review_Workflow_Spec` | 2.0 | Facilitator review layer; post-session Analyst/BD ownership; resolution state machine |
| Enrichment Review + Technical Confirmation | `11_Deepwalk_Enrichment_Review_and_Technical_Confirmation_Spec` | — | P9 principle; customer-facing workflow |
| Studio Design | `Studio_Design_v3.4` | 3.4 | P-principles; domain and action path definitions; Classification Gate logic |

---

## Part 4 — Tier 3: Engineering Track

**Location:** GitHub `docs/`
**Owner:** Engineering
**Format:** `.md`

Tier 3 documents are the canonical source of truth for all schema definitions, field names, data types, permitted values, and pipeline logic. All code, API contracts, and database schemas derive from Tier 3 documents. If a discrepancy exists between a Tier 3 document and any other source, the Tier 3 document is correct.

### Tier 3 document index

| Document | File | Version | What it owns |
|----------|------|---------|-------------|
| Discovery Studio Config Spec | `docs/01_Discovery_Studio_Config_Spec.md` | 2.1 | Canonical `ObservationNode` schema; all field names, types, and permitted values; `BulldogObservation` schema; enum value registries; photo slot definitions; Facilitator pre-session configuration |
| Spatial Model Spec | `docs/06_Spatial_Model_Spec.md` | 1.0 | `space_id`; sticky map system; forensic timestamp engine; `InaccessibleArea` |
| API Contracts | `docs/05_API_Contracts.md` | 1.0 | API endpoint architecture (draft scaffold — engineering review required) |
| Inference Engine Spec | `docs/architecture/inference-engine-spec.md` | 3.1 | Full pipeline stage pseudocode; Stage 6.5 Enrichment Review; human ingestion state machine; Owl and Bulldog inference logic |
| System Architecture Master | `docs/SYSTEM_ARCHITECTURE_MASTER.md` | 1.1 | This document — library index and tier governance |

---

## Part 5 — Tier 4: Module Specs

**Location:** GitHub `docs/modules/`
**Owner:** Assigned per module
**Format:** `.md`

Tier 4 documents specify individual software modules — their inputs, outputs, schemas, business rules, and interface contracts with other modules. They are written for engineers implementing those specific modules.

### Tier 4 document index

| Document | File | Version | What it owns |
|----------|------|---------|-------------|
| Measure Catalog Spec | `docs/modules/03_Measure_Catalog_Spec.md` | 1.0 | `equipment_class` taxonomy; `CatalogEntry` schema; version lifecycle; wattage fallback hierarchy; external source ingestion rules; IE interface contract |
| Report Compiler Spec | `docs/modules/04_Report_Compiler_Spec.md` | 1.0 | Report section spec; finding pool rendering; Performance Map and Opportunity Value Map UI spec; photo rendering rules; P4 language compliance; Inference Summary spec |
| Persistence Engine Spec | `docs/modules/10_Persistence_Engine_Spec.md` | 2.0 | PE product definition; follow-up observation form; signal review and validation; status model; dashboard spec; reporting cadences; annual true-up |
| Implementation Support Spec | `docs/modules/Implementation_Support_Spec_v1.1.md` | 1.1 | Checklist spec; SOP framework; customer system mapping; CSV format spec; future library plan |

---

## Part 6 — Reference Library

**Location:** GitHub `docs/reference/`
**Owner:** Shared — maintained by Third Switch with engineering review
**Format:** `.md`

The Reference Library contains structured reference material that is consumed by multiple tiers but not owned by any single tier. It is version controlled in GitHub alongside the Engineering Track, enabling diff history, pull request review, and consistent access patterns.

Reference Library documents are not spec documents — they do not define pipeline logic or schema. They define the vocabulary, taxonomies, and recurring pattern libraries that the inference engine and report compiler consume.

### Reference Library document index

| Document | File | Version | What it contains |
|----------|------|---------|-----------------|
| Opportunity Signature Library | `docs/reference/Opportunity_Signature_Library.md` | 1.1 | 8 recurring operational condition patterns; domain defaults; action path defaults; evidence requirements; candidate measures |
| Observability Scope Boundary Framework | `docs/reference/Observability_Scope_Boundary_Framework.md` | 1.1 | Observability categories; exception types; `exempt_asset` boundary definition; scope exclusion reason codes; inference logic protocols |
| Operational Load Taxonomy | `docs/reference/Operational_Load_Taxonomy.md` | 1.1 | 5 operational load categories; evidence classification; `exempt_asset` framing for process loads |
| Measure Catalog | `docs/reference/02_Measure_Catalog.md` | living | Living catalog of equipment class entries — starts empty, populated through engagements per `03_Measure_Catalog_Spec` |

**Note on `02_Measure_Catalog`:** This is a living document that grows with every engagement. It is governed by `03_Measure_Catalog_Spec` (Tier 4). It does not have a fixed version — individual entries are versioned per the `Draft → Commit → Release` lifecycle defined in the spec.

---

## Part 7 — Cross-Reference Rules

**Rule 1 — Tier 3 is authoritative for all schema.** No other document may define or modify field names, data types, permitted enum values, or validation rules. If a Tier 2 or Tier 4 document references a field name, it is citing Tier 3 — not defining it.

**Rule 2 — Tier 2 is authoritative for methodology and roles.** Domain definitions, action path definitions, P-principles, and role responsibilities are defined in Tier 2 (Studio Design v3.4 and Surveyor Roles). Tier 3 and Tier 4 documents cite these definitions but do not redefine them.

**Rule 3 — The Reference Library is consumed, not authored, by the pipeline.** The Inference Engine and Report Compiler read from the Reference Library. They do not write to it except where explicitly specified (e.g., `03_Measure_Catalog_Spec` Part 8.2 defines the one circumstance where the IE writes a new catalog entry).

**Rule 4 — Changes to canonical locations require a Decision Register entry.** Moving a document between tiers, changing a file path, or changing location from Drive to GitHub requires a Tier 1 Decision Register entry before implementation.

**Rule 5 — Every document opens with a tier declaration.** The header of every spec and reference document must include Document Identifier, Tier, Location, and Version, and must cite `docs/SYSTEM_ARCHITECTURE_MASTER.md` as the architecture reference.

---

## Part 8 — Canonical Terminology Quick Reference

| Use this | Not this |
|----------|----------|
| `exempt_asset` | `necessary_baseload` |
| `did_you_turn_it_off` | `did_you_turn_off` |
| `fixture_count` | `field_count` |
| `observation_type` | `asset_sub_class`, `asset_type_enum`, `Measure Type`, `Measure Category` |
| `space_id` | `room_id`, `space_function` |
| `field_state_enum` | `state` |
| `why_not` | `why_not_enum` |
| `"No Switch or Control Found"` | `"No Switch Present"`, `"Control Not Found"` |
| Engineering Track | Programmatic Tier |
| Green / Yellow / Red | Immediate / Coordination / Capital (as software output labels) |
| Analyst / BD | Facilitator (for post-session software tasks) |
| Context / Subject / Detail | Device Photo, Nameplate/Closeup (photo slot names) |
| Tier A / B / C | Tier 1 / 2 / 3 (in scoping context) |
| `docs/reference/` | Google Drive (for Reference Library) |

---

## Part 9 — Documents Requiring Harmonization

The following documents contain outdated terminology or references. They are not yet updated. Until harmonization is complete, this document and the Tier 3 specs take precedence.

| Document | Issue | Priority |
|----------|-------|----------|
| GitHub README pipeline diagram | References retired savings pools; outdated pipeline structure | Next harmonization wave |
| `inference-engine-spec.md` Section 6.1 | Uses retired field names: `did_you_turn_off`, `field_count`, `asset_sub_class` | Next harmonization wave |
| Studio Design v3.4 two-tier architecture section | Uses "Programmatic Tier" — replace with "Engineering Track" | Next harmonization wave |

---

## Part 10 — Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | May 2026 | Initial release — four-tier model established |
| 1.1 | May 2026 | Reference Library relocated from Google Drive to GitHub `docs/reference/`; Tier 4 module specs relocated to GitHub `docs/modules/`; Decisions Register relocated to GitHub `docs/decisions/`; document indexes updated; harmonization table added |

---

*Deep//Walk by Third Switch — Assurance at Scale*
*© 2026 Third Switch, LLC.*
*This document is internal reference. Not for external distribution without review.*
