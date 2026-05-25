# Opportunity Signature Library

**Document Identifier:** DW-REF-OSL
**Type:** Reference Library
**Location:** GitHub `docs/reference/Opportunity_Signature_Library.md`
**Version:** 1.1
**Date:** May 2026
**Supersedes:** Opportunity Signature Library v1.0 (Studio Design v3.4 embedded section)

**Architecture reference:** See `docs/SYSTEM_ARCHITECTURE_MASTER.md` for tier definitions and cross-reference rules.

**What this document contains:** The canonical set of Opportunity Signatures — recurring operational energy condition patterns observed during Deepwalk Discovery Sessions. Each signature defines the domain default, action path default, required evidence, exclusion conditions, and candidate measures for a recurring pattern type. The Inference Engine uses signatures to classify observations into normalized patterns before generating findings.

**What this document does not contain:** Inference Engine rule logic — that lives in `docs/architecture/inference-engine-spec.md` (Tier 3). Schema definitions — those live in `docs/01_Discovery_Studio_Config_Spec.md` (Tier 3). Domain and action path principle definitions — those live in Studio Design v3.4 (Tier 2).

**Version 1.1 changes from v1.0:** Action path values updated to canonical V4 terminology throughout — `Operational → Green`, `Coordination → Yellow`, `Capital → Red`. Document extracted from Studio Design embedded section and established as a standalone Reference Library file.

---

## Purpose

Opportunity Signatures define recurring operational energy conditions observed across Deepwalk Discovery Sessions. They represent normalized patterns of inefficiency that appear across many buildings and building types.

Signatures act as the bridge between raw observations and findings. They allow the Inference Engine to:

- Classify recurring operating conditions into recognized pattern types
- Map findings to standardized candidate measures
- Aggregate insights across buildings and engagements
- Improve inference logic over time as the signature library grows

---

## Signature Role in the Deepwalk Architecture

```
Observation Data
    ↓
System Characteristics (equipment_class, w_fixture_w)
    ↓
Operating Pattern Classification
    ↓
Opportunity Signature Assignment  ← this library
    ↓
Finding Generation + Domain / Action Path Scoring
    ↓
Measure Candidate Generation
```

---

## Signature Structure

Each signature contains the following fields:

| Field | Description |
|-------|-------------|
| `signature_id` | Unique identifier |
| `signature_name` | Recurring condition description |
| `category` | Equipment or system category: Lighting / Plug Loads / Controls / HVAC |
| `domain_default` | Default domain: Human / Boundary / Mechanical |
| `typical_action_path` | Default action path: Green / Yellow / Red |
| `required_evidence` | Minimum observation criteria for signature assignment |
| `exclusion_conditions` | Conditions that invalidate or override this classification |
| `candidate_measures` | Linked measure types — not exhaustive |
| `notes` | Interpretation guidance |

---

## Domain Definitions

| Domain | Meaning |
|--------|---------|
| Human | Behavior-driven energy use — correctable through behavioral or procedural change |
| Boundary | Schedule or operational coordination issues — correctable through configuration or cross-team alignment |
| Mechanical | System or control configuration issues — correctable through engineering intervention or equipment action |

Domain classification describes the **origin** of the condition, not the solution difficulty. Action path describes solution difficulty.

---

## Action Path Definitions

| Action Path | Meaning |
|-------------|---------|
| Green | Immediately correctable — no capital required; behavioral, procedural, or simple operational change |
| Yellow | Engineering coordination required — configuration change, schedule adjustment, or cross-team dependency |
| Red | Asset action required — replacement, redesign, or capital investment |

`typical_action_path` is the default assignment for pattern recognition. The Inference Engine may override based on observation-specific evidence. The Facilitator Review Layer allows human override when automated classification is inconsistent with on-site context.

---

## Confidence Logic

Signature classification should consider:

| Level | Meaning | Conditions |
|-------|---------|-----------|
| High | Strong observational support | Multiple observations, consistent zone pattern, photo evidence |
| Moderate | Inference supported by partial evidence | Single observation, no photo, consistent with space type |
| Preliminary | Limited observation, requires confirmation | Single observation, ambiguous context, conflicting signals |

---

## Core Signature Set — Version 1.1

The initial library focuses on commercial office and CRE environments. Eight stable archetypes are defined. Future versions will expand to retail, laboratory, higher education, and specialized operational environments.

---

### Signature 001 — Lighting Active in Low-Occupancy Zones

| Field | Value |
|-------|-------|
| `signature_id` | `SIG-001` |
| `category` | Lighting |
| `domain_default` | Boundary |
| `typical_action_path` | Yellow |

**Description:** Lighting systems remain active in zones with little or no occupancy during the session window. The persistence of lighting without occupancy typically reflects a scheduling or control configuration gap rather than purely behavioral failure.

**Required evidence:**
- Lights on in observed zone
- Low or no occupant presence confirmed
- No visible automatic control behavior (no sensor cycling, no scheduled dim)

**Exclusion conditions:**
- Spaces designated for continuous lighting (egress, safety, security)
- Spaces where a Bulldog (cleaning, security) explains the lighting state

**Candidate measures:** Lighting Shutdown Discipline, Lighting Control Adjustment, Schedule Realignment

---

### Signature 002 — Lighting Controls Not Aligned with Space Use

| Field | Value |
|-------|-------|
| `signature_id` | `SIG-002` |
| `category` | Lighting Controls |
| `domain_default` | Mechanical |
| `typical_action_path` | Yellow |

**Description:** Lighting control configuration does not match the operational behavior of the space. The control hardware exists but is not functioning as intended — sensors disabled, timer settings inconsistent with occupancy, daylight controls not functioning.

**Required evidence:**
- Control type positively identified
- Observed mismatch between control behavior and occupancy pattern

**Exclusion conditions:**
- Temporary override events with known end time
- Controls actively functioning as programmed (mismatch is a scheduling issue → SIG-006)

**Candidate measures:** Lighting Control Adjustment, Control Access Correction, Sensor Recommissioning

---

### Signature 003 — Persistent Workstation Plug Loads

| Field | Value |
|-------|-------|
| `signature_id` | `SIG-003` |
| `category` | Plug Loads |
| `domain_default` | Human |
| `typical_action_path` | Green |

**Description:** Workstation equipment remains energized during low-occupancy periods. Monitors active, laptops docked and charging, local devices operating continuously. The load is behavior-driven — no hardware or configuration barrier prevents shutdown.

**Required evidence:**
- Equipment clusters present and energized
- Low or no occupancy confirmed in the zone
- `did_you_turn_it_off = TRUE` possible (surveyor could power down) or equipment accessible

**Exclusion conditions:**
- Critical workstation operations requiring continuous uptime (servers, render nodes, process control)
- Equipment on overnight batch processing cycle with documented schedule

**Candidate measures:** Plug Load Shutdown Discipline, Workstation Power Management Policy

---

### Signature 004 — Peripheral Equipment Idle Operation

| Field | Value |
|-------|-------|
| `signature_id` | `SIG-004` |
| `category` | Plug Loads |
| `domain_default` | Boundary |
| `typical_action_path` | Green |

**Description:** Shared peripheral equipment remains powered continuously despite intermittent use. Printers, copiers, AV systems, breakroom equipment. The condition is systemic — the same equipment type appears across multiple zones, suggesting a configuration or policy gap rather than individual behavior.

**Required evidence:**
- Equipment operating with no active use observed
- Pattern appears across multiple rooms or zones (distinguishes from SIG-003)

**Exclusion conditions:**
- Equipment required for continuous readiness (emergency systems, always-on server peripherals)

**Candidate measures:** Equipment Shutdown Discipline, Equipment Timer Implementation, Peripheral Power Policy

---

### Signature 005 — HVAC Operating in Low-Occupancy Zones

| Field | Value |
|-------|-------|
| `signature_id` | `SIG-005` |
| `category` | HVAC |
| `domain_default` | Mechanical |
| `typical_action_path` | Yellow |

**Description:** Conditioned air delivered to zones with little or no occupancy during the session. Active HVAC supply confirmed through thermal conditions or direct airflow observation. Requires engineering coordination to adjust zone schedules or segmentation.

**Required evidence:**
- Active HVAC supply confirmed in the observed zone (airflow, thermal delta, thermostat reading)
- Low or no occupancy confirmed

**Exclusion conditions:**
- Spaces requiring constant conditioning regardless of occupancy (server rooms, cold storage, humidity-controlled labs)
- HVAC serving adjacent occupied zones through shared plenum (cannot be isolated)

**Candidate measures:** HVAC Schedule Adjustment, Zone Conditioning Alignment, Segmentation Review

---

### Signature 006 — Extended Operating Schedules

| Field | Value |
|-------|-------|
| `signature_id` | `SIG-006` |
| `category` | Controls |
| `domain_default` | Boundary |
| `typical_action_path` | Yellow |

**Description:** Lighting or HVAC systems operating beyond expected occupancy hours. Systems are functioning as programmed — the schedule itself is misaligned with actual occupancy. The fix is a schedule reconfiguration, not a hardware change.

**Required evidence:**
- Systems operating with minimal occupancy
- Schedule indicators (BAS display, timer setting, thermostat program) visible and inconsistent with occupancy pattern
- Contrast with SIG-002: controls are functioning; the schedule is wrong

**Exclusion conditions:**
- Spaces designated for extended hours operation
- Systems serving multiple zones with genuinely varied occupancy schedules

**Candidate measures:** Lighting Schedule Adjustment, HVAC Schedule Adjustment, BAS Reprogramming

---

### Signature 007 — Breakroom / Appliance Continuous Operation

| Field | Value |
|-------|-------|
| `signature_id` | `SIG-007` |
| `category` | Plug Loads |
| `domain_default` | Human |
| `typical_action_path` | Green |

**Description:** Appliance clusters operating continuously outside normal use periods. Coffee machines, microwaves, toaster ovens, small appliances — energized and ready with no occupants present. Behavior-driven; correctable through shutdown discipline.

**Required evidence:**
- Appliance cluster present and energized
- Continuous energized state observed (not standby — actively powered)

**Exclusion conditions:**
- Refrigeration loads requiring continuous operation (excluded from this signature — address separately as exempt_asset or standalone finding)
- 24-hour operational environments where breakroom is in continuous use

**Candidate measures:** Appliance Shutdown Discipline, Equipment Timer Installation, Breakroom Policy

---

### Signature 008 — Conditioning of Unused Spaces

| Field | Value |
|-------|-------|
| `signature_id` | `SIG-008` |
| `category` | HVAC |
| `domain_default` | Boundary |
| `typical_action_path` | Yellow |

**Description:** HVAC conditioning applied to spaces rarely used during typical operations. Conference rooms, storage areas, secondary lobbies — spaces that are conditioned on the same schedule as occupied zones despite fundamentally different use patterns.

**Required evidence:**
- Low-use space identified and confirmed (no evidence of regular occupancy)
- Conditioned air or active heating observed

**Exclusion conditions:**
- Humidity-controlled spaces where conditioning is required regardless of occupancy
- Spaces with shared HVAC that cannot be independently scheduled

**Candidate measures:** Zone Scheduling Adjustment, Space Consolidation, HVAC Segmentation Review

---

## Signature Aggregation

Over time the signature library enables:

- Cross-building comparison of signature frequency
- Portfolio-level pattern recognition
- Inference model improvement as signature assignment accuracy is validated against outcomes
- Identification of building types or vintages with endemic signature patterns

Signature frequency across a portfolio is an indicator of systemic operational inefficiencies that persist beyond individual buildings.

---

## Version Notes

Version 1.0 established the initial signature set for commercial office environments.
Version 1.1 updated action path terminology to Green / Yellow / Red and established this as a standalone Reference Library file.

Future versions will expand to:
- Retail environments
- Laboratory and clinical spaces
- Higher education buildings
- Specialized operational environments (food service, cold storage, data centers)

---

*Deep//Walk by Third Switch — Assurance at Scale*
*© 2026 Third Switch, LLC.*
*This document is internal reference. Not for external distribution without review.*
