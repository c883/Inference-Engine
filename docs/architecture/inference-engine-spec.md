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
    In[Raw Field Session Observations] --> S1[Stage 1: Ingestion & Normalization]
    S1 --> S2[Stage 2: Global Portfolio Context]
    S2 --> S3[Stage 3: Localized Telemetry Context Filter]
    S3 --> S4[Stage 4: Structural Spatial Clustering]
    S4 --> S5[Stage 5: Asset Capability Resolution]
    
    S5 --> G1{Gate 1 Check}
    G1 -->|Damaged| SC_Red[Short-Circuit: Assign Red Token]
    G1 -->|Baseload| SC_Base[Hardcode Hours to 0]
    G1 -->|Passed| S6[Stage 6: Control Domain Allocation]
    
    S6 --> G2{Gate 2 Matrix}
    G2 --> S7[Stage 7: Operational Reach Evaluation]
    
    S7 --> G3{Gate 3 Check}
    G3 -->|Unknown Vendor| S8[Stage 8: Barrier Nature Assignment]
    G3 -->|Reach Yes| Path_Green[Assign Green Token]
    
    S8 --> G4{Gate 4 Check}
    G4 -->|Reconfiguration| Path_Yellow[Assign Yellow Token]
    G4 -->|Capital Auth| Path_Red[Assign Red Token]
    
    SC_Red --> S9[Stage 9: Savings Calculation]
    Path_Green --> S9
    Path_Yellow --> S9
    Path_Red --> S9
    SC_Base --> S10[Stage 10: Finding Pool Distribution]
    S9 --> S10
    
    S10 --> Val{Physics Check}
    Val -->|Exceeds Cap| Err[Trigger Calibration Error]
    Val -->|Passed| Output[Output Pools: Guarantee / Added Value / Baseline]
