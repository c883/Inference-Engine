# Deepwalk Documentation Sprint — Handoff Document

**Date:** May 2026  
**Purpose:** Context transfer document for continuing the documentation sprint in a new conversation. Read this before doing anything else.

---

## Part 1 — What This Sprint Accomplished

This session completed a full cross-source reconciliation of all Deepwalk spec documents and produced the first six documents of a structured 14-document spec library. The four-tier architecture model was established and locked. Key blocking decisions were made and recorded.

---

## Part 2 — The Four-Tier Model (Locked)

| Tier | Name | Location | Owner |
|------|------|----------|-------|
| 1 | Decision Register | Google Drive | Third Switch leadership |
| 2 | Conceptual / Operational Track | Google Drive | Product, methodology, ops |
| 3 | Engineering Track | GitHub `docs/` | Engineering |
| 4 | Module Specs | Drive + GitHub `docs/modules/` | Assigned per module |

**Reference Library** — shared, not owned by any tier. Google Drive.

The governing document is `docs/SYSTEM_ARCHITECTURE_MASTER.md` in GitHub. Every new document must open with a tier declaration and cite this document.

---

## Part 3 — Documents Completed This Sprint

All six are final and should be uploaded to their canonical locations if not already done.

| Document | Tier | Location | Version | Notes |
|----------|------|----------|---------|-------|
| `SYSTEM_ARCHITECTURE_MASTER.md` | 2 | GitHub `docs/` | 1.0 | Index document. Read first. |
| `01_Discovery_Studio_Config_Spec.md` | 3 | GitHub `docs/` | 2.1 | Canonical ObservationNode schema. Everything cites this. |
| `06_Spatial_Model_Spec.md` | 3 | GitHub `docs/` | 1.0 | space_id, sticky map, forensic timestamp, InaccessibleArea |
| `05_API_Contracts.md` | 3 | GitHub `docs/` | 1.0 | Draft scaffold — field content sourced, endpoint architecture needs engineering review |
| `08_Surveyor_Roles.md` | 2 | Google Drive | 2.0 | Role definitions, Bulldog quick-capture, domain/action path bias |
| `09_Scoping_Ingestion.md` | 2 | Google Drive | 2.0 | Tier A/B/C input framework, SessionEnrichmentContext object |

---

## Part 4 — Key Decisions Made This Sprint

These decisions are now embedded in the documents above. Summarized here for reference.

### Locked decisions

**`why_not` field — reduced from 6 values to 4:**
- `"No Switch or Control Found"` — replaces both `"No Switch Present"` and `"Control Not Found"`
- `"Requires Permission"` — retained
- `"Occupancy Sensor"` — retained, tightened: requires positive ID of sensor controlling specific asset
- `"Other"` — retained as intentional catch-all for Enrichment Review
- `null` — only when `did_you_turn_it_off` is TRUE
- `"Timer / Schedule"` — **removed from V1**; reserved for V2 with photo requirement and role restriction
- `"Control Not Found"` — **retired**; collapsed into `"No Switch or Control Found"`
- `"No Switch Present"` — **retired**; renamed and collapsed

**`field_state_enum` — demoted to optional:**
- Values: `ON`, `OFF`, `AUTO`, `UNKNOWN`
- Default: `UNKNOWN`
- Photos and `raw_notes` are authoritative state signals; `field_state_enum` is a structured shortcut
- Never required — not in the always-required field set
- `AUTO` requires positive identification of a control device — not a fallback for uncertainty

**`BulldogObservation` — `badge_photo_ref` added:**
- Optional fourth field — URI pointing to photo of worker badge or name tag
- Refusal to be photographed is a signal worth noting in `notes`

**Floor plan overlay — not a V1 feature:**
- The Opportunity Map is a dual-axis Performance Map grid (Human/Boundary/Mechanical × Green/Yellow/Red)
- No floor plan overlay, no pins, no spatial placement on a map in V1
- `map_coordinates` field is reserved in schema but not implemented
- This was incorrectly introduced from a retired pre-v3.0 PDF brief and has been corrected in `06_Spatial_Model_Spec`

**Tier input labels renamed A/B/C** in `09_Scoping_Ingestion` to avoid collision with the document tier model (Tier 1–4).

**"Engineering Track"** is the canonical term. "Programmatic Tier" is retired. Appears in existing docs that still need updating: GitHub README, v3.4 two-tier architecture section.

**Measure Catalog split:**
- `03_Measure_Catalog_Spec` (Tier 4) = translation spec — ingestion rules from external sources (DEER, manufacturer data, cut sheets, ENERGY STAR, DLC, ASHRAE) into Deepwalk catalog entries
- `02_Measure_Catalog` (Reference Library) = living catalog library, starts empty, populated through engagements

**Opportunity Map:**
- Is a dual-axis Performance Map grid — not a floor plan feature
- UI specification belongs in Report Compiler spec (Tier 4)
- Spatial contribution from `06_Spatial_Model_Spec`: resolved location fields (`space_id`, `floor`, `zone`) on each finding record for filtering purposes only

### Open decisions (not yet in Decision Register)

| Item | Status |
|------|--------|
| `field_state_enum` long-term fate — Option A (optional, current) is V1 decision; Option B (remove entirely) is a future consideration | Logged as open — do not change without Decision Register entry |
| `"Timer / Schedule"` V2 reintroduction — requires `Detail` photo + Facilitator or facilities-staff attribution | Noted in `01_Discovery_Studio_Config_Spec` Part 11 |
| Exempt Asset kWh estimation methodology | Open — before first engagement with confirmed exempt assets |
| Demand charge peak kWh reduction logic | Open — before first demand-charge engagement |
| Stage 6 scoring matrix — shelved for V2 | Open — after first 3 engagements |
| SEM Compliance Column — Willdan engagement | Open |

---

## Part 5 — Remaining Documents (7 total)

### Wave 2 — 2 remaining

| # | Document | Tier | Location | Primary work |
|---|----------|------|----------|-------------|
| 1 | `07_Review_Workflow_Spec` | 2 | Drive | V4 role boundary update: Facilitator software loop ends at session close; Analyst/BD owns Stage 6.5; align 6 resolution states with Enrichment spec |
| 2 | `03_Measure_Catalog_Spec` | 4 | Drive + GitHub | Translation spec: ingestion rules from external sources; equipment_class taxonomy; version control (Draft→Commit→Release); wattage fallback hierarchy; DEER/ENERGY STAR/DLC/ASHRAE mapping |

### Wave 3 — 5 remaining

| # | Document | Tier | Location | Primary work |
|---|----------|------|----------|-------------|
| 3 | `04_Report_Compiler_Spec` | 4 | Drive + GitHub | Full section specs; flat array → named pool rendering; finding pool presentation; P4 language rules for narrative; Opportunity Map dual-axis grid UI spec lives here |
| 4 | `10_Persistence_Engine_Spec` | 4 | Drive | Harmonize role language with V4; Facilitator verification in PE is separate from post-session software loop |
| 5 | `Implementation_Support_Spec_v1.1` | 4 | Drive | Update v1.0; downstream of Report Compiler; customer system mapping; future library plan |
| 6 | Opportunity Signature Library | Ref | Drive | Update action path language: replace Operational/Coordination/Capital with Green/Yellow/Red per v3.4; verify P5 alignment |
| 7 | Observability Scope Boundary Framework | Ref | Drive | Replace `necessary_baseload` with `exempt_asset`; align exception codes with v3.4 five sanctioned reason codes |
| 8 | Operational Load Taxonomy | Ref | Drive | Verify process load framing consistent with exempt_asset boundary logic; minor language alignment |

---

## Part 6 — Canonical Terminology (Quick Reference)

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
| Tier A / B / C | Tier 1 / 2 / 3 (in scoping context — avoids collision with document tier model) |

---

## Part 7 — How to Start the New Conversation

1. Open a new conversation with Claude
2. Upload this handoff document
3. Upload `SYSTEM_ARCHITECTURE_MASTER.md` from GitHub (or paste its contents)
4. Upload `01_Discovery_Studio_Config_Spec.md` from GitHub — this is the most-referenced document
5. Say: "We are continuing the Deepwalk documentation sprint. Read the handoff document and the two uploaded specs, then we will start with `07_Review_Workflow_Spec`."

The new conversation does not need the full reconciliation history — the decisions are embedded in the committed documents. The handoff document and `01` together contain enough context to write the remaining seven documents correctly.

---

## Part 8 — Source Documents (for reference)

The following source documents were read during this sprint and informed all decisions. They do not need to be re-read in the new conversation unless a specific section is disputed.

| Document | Drive ID / Location | Version |
|----------|-------------------|---------|
| Studio Design | Drive `16v5ZBYKcdjQi...` | v3.4 |
| IE Spec (Drive) | Drive `19vw7UUHZ8vBCj...` | V3.1 |
| IE Spec (GitHub) | `docs/architecture/inference-engine-spec.md` | v3.1 Production |
| Master Reconciliation Spec | Drive `1e2R8pkfEfoTjkT0MES_xSSTV78GC7GrRP0AfdLEhsN8` | V4 |
| Brief / Performance Map | Drive `1Ktv72zSuADgN5...` | v4 |
| Enrichment Review Spec | Drive `1zpKtz5wl_UzoB...` | Revised draft |
| Attached PDF (retired) | Uploaded in this session | Pre-v3.0 — DO NOT USE |

---

*Deep//Walk by Third Switch — Assurance at Scale*  
*© 2026 Third Switch, LLC.*
