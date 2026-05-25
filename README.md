# Deepwalk Inference Engine

An isolated, deterministic backend repository that ingests raw field observations and outputs validated energy findings, domain scores, and savings metrics, completely decoupled from the Discovery Studio frontend interface.

**Architecture reference:** See `docs/SYSTEM_ARCHITECTURE_MASTER.md` for tier definitions, document index, and cross-reference rules.

---

## Repository Structure

```
Inference-Engine/
├── README.md
├── docs/
│   ├── SYSTEM_ARCHITECTURE_MASTER.md      # Library index and tier governance
│   ├── 01_Discovery_Studio_Config_Spec.md # Canonical ObservationNode schema
│   ├── 06_Spatial_Model_Spec.md           # space_id, sticky map, forensic timestamp
│   ├── 05_API_Contracts.md                # API endpoint architecture (draft)
│   ├── architecture/
│   │   └── inference-engine-spec.md       # Full pipeline stage spec (v3.1 production)
│   ├── decisions/
│   │   └── DECISIONS_REGISTER.md          # Architectural decisions register (Tier 1)
│   ├── modules/
│   │   ├── 03_Measure_Catalog_Spec.md     # equipment_class taxonomy; wattage fallback hierarchy
│   │   ├── 04_Report_Compiler_Spec.md     # Report section spec; Performance Map; finding cards
│   │   ├── 10_Persistence_Engine_Spec.md  # PE product definition; status model; dashboard
│   │   └── Implementation_Support_Spec_v1.1.md  # Checklist; SOPs; customer system mapping
│   └── reference/
│       ├── Opportunity_Signature_Library.md          # 8 recurring operational condition patterns
│       ├── Observability_Scope_Boundary_Framework.md # Observability limits; exempt_asset; exception codes
│       └── Operational_Load_Taxonomy.md              # 5 operational load categories; evidence classification
```

---

## Pipeline Overview

The Inference Engine processes field observations through 10 sequential stages, from normalization through savings calculation, producing a locked flat array consumed by the Report Compiler.

```mermaid
flowchart TD
    classDef layer1 fill:#fff5f5,stroke:#e53e3e,stroke-width:2px,color:#9b2c2c;
    classDef layer2 fill:#fffaf0,stroke:#dd6b20,stroke-width:2px,color:#9c4221;
    classDef layer3 fill:#ebf8ff,stroke:#2b6cb0,stroke-width:2px,color:#1a365d;
    classDef layer4 fill:#f7fafc,stroke:#4a5568,stroke-width:2d3748;
    classDef layer5 fill:#f0fff4,stroke:#38a169,stroke-width:2px,color:#22543d;

    %% --- LAYER 1: FIELD DATA ACQUISITION ---
    subgraph L1 [1. FIELD DATA CAPTURE]
        A[ObservationNode] --> S1[Stage 1: Normalize]
        S1 -->|Validate why_not| FormCheck{Matches V4 Enums?}
        FormCheck -->|NO| Err_1[Halt: ERR_STAGE1_INVALID_UI_STRING]
    end
    style L1 fill:#fff5f5,stroke:#e53e3e,stroke-width:1px

    %% --- LAYER 2: GLOBAL CONTEXT INGESTION ---
    subgraph L2 [2. MACRO BASELINE CONTAINER]
        FormCheck -->|YES| S2[Stage 2: Inject Global Context + Catalog Lookup]
        D[(SessionEnrichmentContext\nMeasure Catalog)] --> S2
    end
    style L2 fill:#fffaf0,stroke:#dd6b20,stroke-width:1px

    %% --- LAYER 3: TELEMETRY FILTERS ---
    subgraph L3 [3. TELEMETRY PROCESSING]
        S2 --> S3[Stage 3: Telemetry Decay Filter\nOwl + Bulldog Influence Windows]
        S3 --> S4[Stage 4: Structural Spatial Clustering]
    end
    style L3 fill:#ebf8ff,stroke:#2b6cb0,stroke-width:1px

    %% --- LAYER 4: RESOLUTION ENGINE ---
    subgraph L4 [4. CORE RESOLUTION MODULES]
        S4 --> S5[Stage 5: Asset Capability Check\nexempt_asset evaluation]

        S5 -->|why_not: Requires Permission| SC_Base[Hardcode Reduction Hours to 0]

        S5 -->|Passed| S6[Stage 6: Control Domain Weights]
        S6 -->|why_not: No Switch or Control Found| S6

        S6 --> S7[Stage 7: Operational Reach Evaluation]
        S7 -->|why_not: No Switch or Control Found\nresolved unfound| S8[Stage 8: Barrier Nature Assignment]
        S7 -->|Reach == YES| Path_Green[Assign Green Token]

        S8 --> Path_Yellow[Assign Yellow Token]
        S8 --> Path_Red[Assign Red Token]
    end
    style L4 fill:#f7fafc,stroke:#4a5568,stroke-width:1px

    %% --- LAYER 5: OUTPUT ---
    subgraph L5 [5. OUTPUT + ASSUMPTION LOCK]
        SC_Base --> S9
        Path_Green --> S9[Stage 9: Physics Savings Calculation\nW_fixture from Measure Catalog]
        Path_Yellow --> S9
        Path_Red --> S9

        S2 -->|why_not: Other| S10[Stage 10: Aggregate Data Verification\nAssumption Lock]
        S9 --> S10

        S10 --> Check{Reconciliation Status?}
        Check -->|Unresolved items| Queue[Enrichment Review Queue\nStage 6.5]
        Check -->|All LOCKED| Output[/Stage 10 Flat Array\nReport Compiler Input/]

        Output --> RC[Report Compiler\ndocs/modules/04_Report_Compiler_Spec.md]
        Output --> PE[Persistence Engine\ndocs/modules/10_Persistence_Engine_Spec.md]
    end
    style L5 fill:#f0fff4,stroke:#38a169,stroke-width:1px
```

---

## Key V4 Terminology

| Use this | Not this |
|----------|----------|
| `exempt_asset` | `necessary_baseload` |
| `did_you_turn_it_off` | `did_you_turn_off` |
| `fixture_count` | `field_count` |
| `observation_type` | `asset_sub_class`, `Measure Type`, `Measure Category` |
| `space_id` | `room_id`, `space_function` |
| `field_state_enum` | `state` |
| `why_not` | `why_not_enum` |
| `"No Switch or Control Found"` | `"No Switch Present"`, `"Control Not Found"` |
| Green / Yellow / Red | Immediate / Coordination / Capital |
| Engineering Track | Programmatic Tier |

**Retired from V1 (do not implement):**
- `"Timer / Schedule"` why_not value — reserved for V2
- Guarantee Savings Pool, Added Value Pool, Documented Baseline Pool
- Distribution Handoff Matrix

---

## Documents Requiring Harmonization

The following files in this repository contain outdated terminology and are scheduled for update:

| File | Issue |
|------|-------|
| `docs/architecture/inference-engine-spec.md` Section 6.1 | Uses retired field names: `did_you_turn_off`, `field_count`, `asset_sub_class` |

---

*Deep//Walk by Third Switch — Assurance at Scale*
*© 2026 Third Switch, LLC. Internal reference. Not for external distribution without review.*
