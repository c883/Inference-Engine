## SECTION 4 — 10-STAGE PIPELINE

The Inference Engine processes observations through a rigid 10-stage sequential pipeline. No stage may be bypassed. Observations must complete each stage in order before advancing.

1. **STAGE 1** — Ingestion and Normalization
2. **STAGE 2** — Global Portfolio Context (Macro Baseline Container)
3. **STAGE 3** — Localized Telemetry Context Filter
4. **STAGE 4** — Structural Spatial Clustering
5. **STAGE 5** — Asset Capability Resolution
6. **STAGE 6** — Control Domain Allocation
7. **STAGE 7** — Operational Reach Evaluation
8. **STAGE 8** — Barrier Nature Assignment
9. **STAGE 9** — Measure Generation and Savings Calculation
10. **STAGE 10** — Finding Pool Distribution and Output Validation

### 4.1 — Pipeline Logic Schematic

Below is the technical visualization of the 10-stage backend pipeline execution paths:

```mermaid
flowchart TD
    %% Custom UI Styles for Clarity
    classDef layer1 fill:#fff5f5,stroke:#e53e3e,stroke-width:2px,color:#9b2c2c;
    classDef layer2 fill:#fffaf0,stroke:#dd6b20,stroke-width:2px,color:#9c4221;
    classDef layer3 fill:#ebf8ff,stroke:#2b6cb0,stroke-width:2px,color:#1a365d;
    classDef layer4 fill:#f7fafc,stroke:#4a5568,stroke-width:2px,color:#2d3748;
    classDef layer5 fill:#f0fff4,stroke:#38a169,stroke-width:2px,color:#22543d;

    %% --- LAYER 1: FIELD DATA ACQUISITION ---
    subgraph L1 [1. FIELD DATA CAPTURE]
        A[Raw Observation Node] --> S1[Stage 1: Normalize Text Strings]
        S1 -->|Validate 'Why Not' Dropdown| FormCheck{Matches Allowed Enums?}
        FormCheck -->|NO| Err_1[Halt: ERR_STAGE1_INVALID_UI_STRING]
    end
    style L1 fill:#fff5f5,stroke:#e53e3e,stroke-width:1px

    %% --- LAYER 2: GLOBAL CONTEXT INGESTION ---
    subgraph L2 [2. MACRO BASELINE CONTAINER]
        FormCheck -->|YES| S2[Stage 2: Inject Global Context]
        D[(Engagement Profile)] --> S2
    end
    style L2 fill:#fffaf0,stroke:#dd6b20,stroke-width:1px

    %% --- LAYER 3: REAL-TIME TELEMETRY FILTERS ---
    subgraph L3 [3. TELEMETRY PROCESSING]
        S2 --> S3[Stage 3: Telemetry Decay Filter]
        S2 -->|Why Not: Occupancy Sensor| S3
        S3 --> S4[Stage 4: Structural Spatial Clustering]
    end
    style L3 fill:#ebf8ff,stroke:#2b6cb0,stroke-width:1px

    %% --- LAYER 4: RESOLUTION ENGINE & PATHS ---
    subgraph L4 [4. CORE RESOLUTION MODULES]
        S4 --> S5[Stage 5: Asset Capability Check]
        
        %% Short Circuits
        S5 -->|Why Not: Requires Permission| SC_Base[Hardcode Reduction Hours to 0]
        
        %% Normal Tree & Logic Splits
        S5 -->|Passed| S6[Stage 6: Control Domain Weights]
        S6 -->|Why Not: No Switch Present / Timer / Schedule| S6
        
        S6 --> S7[Stage 7: Operational Reach Evaluation]
        S7 -->|Why Not: Control Not Found| S8[Stage 8: Barrier Nature Assignment]
        S7 -->|Reach == YES| Path_Green[Assign Green Token]
        
        S8 --> Path_Yellow[Assign Yellow Token]
        S8 --> Path_Red[Assign Red Token]
    end
    style L4 fill:#f7fafc,stroke:#4a5568,stroke-width:1px

    %% --- LAYER 5: VALIDATION & SAVINGS POOLS ---
    subgraph L5 [5. OUTPUT VALIDATION LAYER]
        SC_Base --> S10
        Path_Green --> S9[Stage 9: Run Physics Savings Math]
        Path_Yellow --> S9
        Path_Red --> S9
        
        S2 -->|Why Not: Other| S10[Stage 10: Aggregate Data Verification]
        S9 --> S10
        
        S10 --> Check{Total Savings Exceed Portfolio Cap?}
        Check -->|YES| Err_2[Trigger Calibration Error Flag]
        Check -->|NO| Handoff[/Distribution Handoff Matrix/]
        
        Handoff --> Pool1[(Guarantee Savings Pool)]
        Handoff --> Pool2[(Added Value Pool)]
        Handoff --> Pool3[(Documented Baseline Pool / Metadata Preserved)]
    end
    style L5 fill:#f0fff4,stroke:#38a169,stroke-width:1px
```
## SECTION 5 — STAGE SPECIFICATIONS

### 5.1 — STAGE 1: Ingestion and Normalization

#### 5.1.1 — Purpose
To ingest raw field observations from the mobile discovery interface, validate payload parameters, and normalize arbitrary text entries into strict system enums. If input verification fails, processing halts immediately before affecting data state.

#### 5.1.2 — UI Dropdown Field Enforcements ("Why Not" Rule-Couplet)
When a field observation record indicates that an asset was not deactivated by the surveyor (`did_you_turn_off == "NO"`), the ingestion API contract enforces an exact text string match against one of six allowed values. 

The inference engine executes a deterministic data-routing sequence strictly bounded by these values:

```json
{
  "type": "object",
  "properties": {
    "did_you_turn_off": { "type": "string", "enum": ["YES", "NO"] },
    "why_not_enum": {
      "type": "string",
      "enum": [
        "No Switch Present",
        "Requires Permission",
        "Occupancy Sensor",
        "Timer / Schedule",
        "Control Not Found",
        "Other"
      ]
    }
  },
  "required": ["did_you_turn_off"]
}
```
#### 5.1.3 — Programmatic Routing Paths

The inference engine executes a deterministic data-routing sequence strictly bounded by the validated `why_not_enum` values:

1. **"No Switch Present"**
   * **Target Route:** STAGE 6 — Control Domain Allocation
   * **Action:** Bypasses manual occupant routing; forces classification to a centralized circuit-level panel distribution loop.

2. **"Requires Permission"**
   * **Target Route:** STAGE 5 — Asset Capability Resolution
   * **Action:** Evaluates asset under the critical capability criteria profile. If verified as operational baseload, operational runtime reduction hours ($H_{\text{reduction}}$) are hard-coded to zero.

3. **"Occupancy Sensor"**
   * **Target Route:** STAGE 3 — Localized Telemetry Context Filter
   * **Action:** Injects a local sensor timeout multiplier variable to modulate mathematical runtime baselines.

4. **"Timer / Schedule"**
   * **Target Route:** STAGE 6 — Control Domain Allocation
   * **Action:** Links asset domain metrics directly to a mechanical timeclock or automated building automation schedule profile.

5. **"Control Not Found"**
   * **Target Route:** STAGE 7 — Operational Reach Evaluation
   * **Action:** Triggers the P7 Default operational restriction rule, forcing reach metrics to `UNKNOWN` and pushing the asset directly to Stage 8.

6. **"Other"**
   * **Target Route:** STAGE 10 — Finding Pool Distribution and Output Validation
   * **Action:** Categorizes record as an unresolved outlier anomaly. Affixes an immutable flag forcing a manual engineering review before report compilation.
