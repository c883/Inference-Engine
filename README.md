# Deepwalk Inference Engine (Programmatic Tier)

An isolated, deterministic backend repository that ingests raw field observations and outputs validated energy findings, domain scores, and savings metrics, completely decoupled from the Discovery Studio frontend interface.

---

## 🌍 GLOBAL ECOSYSTEM PIPELINE

Below is the end-to-end flow showing how physical field telemetry, macro financial constraints, and our 10 programmatic pipeline stages interact to generate validated client deliverables.

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
