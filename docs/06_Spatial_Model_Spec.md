# 06 — Spatial Model Spec

**Document Identifier:** DW-T3-06  
**Tier:** 3 — Engineering Track  
**Location:** GitHub `docs/06_Spatial_Model_Spec.md` (canonical)  
**Version:** 1.0  
**Date:** May 2026

**Architecture reference:** See `docs/SYSTEM_ARCHITECTURE_MASTER.md` for tier definitions and cross-reference rules.

**What this document owns:** The complete spatial data contract for Discovery Sessions — `space_id` as the canonical location token, the sticky map mechanic, the forensic timestamp anchoring system for ad-hoc entries, the full location hierarchy resolved post-session, the Opportunity Map data structure, and the Analysis Boundary Gate derating behavior at the spatial level.

**What this document does not own:** The `ObservationNode` schema (which contains `space_id` as a field) lives in `01_Discovery_Studio_Config_Spec`. The M3 role responsibilities that support spatial accuracy live in `08_Surveyor_Roles` (Tier 2). The Analysis Boundary Gate classification logic lives in Studio Design v3.4 Part 1 P8 (Tier 2).

---

## Part 1 — Design Philosophy

### Why spatial data is captured differently from observation data

Every other field in the `ObservationNode` payload describes *what* was observed. `space_id` describes *where*. The two have different reliability profiles in the field.

Observation fields — `observation_type`, `fixture_count`, `did_you_turn_it_off` — require the surveyor to look at an asset and record what they see. They are point-in-time, asset-specific, and relatively immune to team coordination errors.

Location data is different. It is shared across the team — every surveyor in the same room should have the same `space_id`. It depends on M3 announcements, floor plan familiarity, and consistent naming. It is the field most vulnerable to drift during a fast-paced session.

The spatial model is designed around this reality. Rather than requiring surveyors to navigate a multi-level location hierarchy on a mobile device while moving through a building, the system uses a single sticky token (`space_id`) that requires one update per room, persists automatically, and is anchored by timestamp sequencing when entries are imprecise.

The full location hierarchy — building, floor, zone, room — is resolved post-session during Stage 6.5 Enrichment, not in the field.

---

## Part 2 — space_id: The Canonical Location Token

### Definition

`space_id` is a string identifier that maps an observation to a physical location in the building. It is the only location field in the mobile payload.

```json
"space_id": "STRING — Sticky map label or ad-hoc anchor string."
```

### Two entry modes

**Mode 1 — Map label (preferred):** The surveyor enters the room code or label from the pre-loaded floor plan. Labels are loaded into the session by the Facilitator during pre-session configuration from the site's facility map or naming convention. Example: `"NR-1101"`, `"FL3-CONF-B"`, `"MECH-B1"`.

**Mode 2 — Ad-hoc anchor (fallback):** When a surveyor encounters an unmapped space — a room not on the floor plan, a structural pocket, a sub-divided zone — they enter a free-text anchor string. Example: `"Next to 101"`, `"Unmapped Mech Pocket East Wing"`, `"Behind Server Room"`. Ad-hoc entries are resolved post-session by the forensic timestamp engine (see Part 4).

### The sticky rule

Once entered, `space_id` persists across all subsequent observations submitted by that surveyor until they manually clear or update it. A surveyor entering a new room updates `space_id` once — every observation submitted until the next update inherits that value automatically.

**Rationale:** Requiring surveyors to re-enter location data for every observation introduces transcription errors and slows capture speed. The sticky rule reduces location entry to one action per room transition. The M3's verbal room announcements are the coordination mechanism that keeps the team's `space_id` values synchronized.

### Validation

`space_id` is always required. A submission with a null or empty `space_id` fails schema validation and is rejected before Stage 1. The system does not accept blank location tokens — an ad-hoc free-text entry is always preferable to no entry.

---

## Part 3 — The Location Hierarchy

The full location hierarchy is a five-level structure. It is resolved post-session — surveyors never populate more than `space_id` in the field.

```
Site
 └── Building
      └── Floor
           └── Zone
                └── Room / Space
                     └── Waypoint (optional sub-point within a room)
```

### Resolution process

During Stage 6.5 Enrichment Review, the Analyst maps each unique `space_id` value to its position in the full location hierarchy using the site's facility directory. This produces a resolved location record attached to every observation cluster.

**Map label entries** resolve directly — the label maps to a pre-loaded facility directory entry.

**Ad-hoc entries** resolve via the forensic timestamp engine (see Part 4) and Analyst review.

### Waypoint

Waypoint is a sub-room anchor — a specific point within a room where an observation was made. It is optional and is most useful in large open-plan floors, mechanical rooms, or spaces where the room-level `space_id` is insufficiently precise for post-session location resolution.

Waypoints are entered as free text in `raw_notes` when needed (`"NW corner near window"`, `"Above drop ceiling panel 12C"`). They are not a separate schema field in V1. The Analyst assigns waypoint coordinates during Enrichment if the observation warrants Opportunity Map placement.

---

## Part 4 — Forensic Timestamp Anchoring

### Purpose

When a surveyor enters an ad-hoc `space_id` that cannot be directly resolved from the facility directory, the forensic timestamp engine uses the sequential timestamps of surrounding verified observations to anchor it to the correct location geometry.

### How it works

Every `ObservationNode` carries an immutable `timestamp` generated at submission time. During post-session data processing, the engine builds a temporal sequence of all observations for each surveyor, ordered by timestamp.

An ad-hoc entry is bracketed by verified entries — the observation immediately before it (in a known room) and the observation immediately after it (in a known room). The engine assigns the ad-hoc entry to a location range bounded by those two known points.

The Analyst then confirms or adjusts the anchor during Stage 6.5 Enrichment Review, using the surrounding context and any `raw_notes` on the ad-hoc observation.

### Limits

The forensic timestamp engine is an anchoring assist — it narrows the resolution window, it does not replace Analyst judgment. If an ad-hoc entry is surrounded by other ad-hoc entries (a surveyor who diverged from the team for an extended period without updating `space_id`), the engine flags the sequence for Analyst review rather than attempting to resolve it automatically.

### Bulldog and Owl timing

The forensic timestamp also governs Bulldog influence window calculations and Owl room-scope assignments. A Bulldog observation's `timestamp` marks the start of its influence window — all observations with timestamps within the window and within the Bulldog's scope receive the appropriate confidence adjustment. Owl observations are room-scoped: only observations with matching `space_id` and timestamps within the Owl's presence window are affected.

---

## Part 5 — Pre-Session Spatial Configuration

The Facilitator loads spatial data before the session begins as part of pre-session configuration (see `01_Discovery_Studio_Config_Spec` Part 10).

| Configuration item | What it controls |
|-------------------|-----------------|
| `space_id_options` | Pre-loaded list of room codes and labels from the site facility map. Presented as a dropdown or autocomplete in the mobile UI. Surveyors select from this list for Map label entry mode. |
| `site_directory` | The master facility directory mapping each `space_id` to its full location hierarchy (building / floor / zone / room). Used during Stage 6.5 resolution. |

**Offline operation:** `space_id_options` are loaded to the device before the session. The mobile app functions fully offline — no network connection is required during the session. All submissions are queued locally and synced when connectivity is restored.

---

## Part 6 — Spatial Attributes of Finding Records

This document owns the spatial fields attached to each finding record post-session. The Opportunity Map UI — the dual-axis Performance Map grid and its navigation behavior — is specified in the Report Compiler spec (Tier 4). Classification logic that determines `domain` and `action_path` values is owned by Studio Design v3.4 Part 1 P8 (Tier 2).

The spatial contribution of this document is: each finding record carries resolved location fields so that findings can be filtered and grouped by location in downstream tools.

### Resolved location fields on a finding record

```json
{
  "FindingSpatialRecord": {
    "finding_id":  "STRING — UUID linking to the finding in the flat output array.",
    "space_id":    "STRING — Resolved canonical space_id from the observation cluster.",
    "floor":       "STRING — Resolved from location hierarchy during Stage 6.5.",
    "zone":        "STRING — Resolved from location hierarchy during Stage 6.5.",
    "domain":      "ENUM   — Human | Boundary | Mechanical. Assigned by Inference Engine.",
    "action_path": "ENUM   — Green | Yellow | Red. Assigned by Inference Engine."
  }
}
```

**`domain` and `action_path`** are included here as grouping keys only — they allow downstream tools to filter findings by classification alongside location. Their values are assigned entirely by the Inference Engine. This document does not define or constrain them.

**`floor` and `zone`** are resolved post-session during Stage 6.5 Enrichment Review by mapping `space_id` against the site directory. They are not entered in the field.

---

## Part 7 — Inaccessible Area Recording

When a surveyor cannot enter a space, the inaccessible area is recorded as a distinct observation type — not as a gap in coverage.

```json
{
  "InaccessibleArea": {
    "session_id":   "STRING    — UUID. Auto-generated.",
    "timestamp":    "TIMESTAMP — Immutable system clock. Auto-generated.",
    "space_id":     "STRING    — Best available identifier for the inaccessible space.",
    "reason":       "ENUM      — See permitted values below.",
    "notes":        "STRING    — Optional. Additional context.",
    "surveyor_role":"ENUM      — Role that attempted access."
  }
}
```

### Permitted `reason` values

| Value | Meaning |
|-------|---------|
| `"Locked"` | Space was locked; no key or access credential available during the session |
| `"Occupied"` | Space was occupied in a way that prevented survey entry |
| `"Requires Permission"` | Access requires authorization not obtained before the session |
| `"Hazardous"` | Safety concern prevented entry |
| `"Other"` | Reason does not fit above categories; describe in `notes` |

### Why this matters

Inaccessible areas are not coverage failures — they are data points. They are recorded, flagged, and feed into the scoping and enrichment process for follow-up access planning. The M3 role is primarily responsible for recording inaccessible areas as the team routes through the building. See `08_Surveyor_Roles` Part 2.

---

## Part 8 — Retired Spatial Field Names

| Retired name | Canonical replacement | Source of change |
|-------------|----------------------|-----------------|
| `room_id` | `space_id` | V4 Section 2 |
| `space_function` | `space_id` | V4 Section 2 |
| `building / floor / zone / room / waypoint` (as mobile form fields) | `space_id` (single field; hierarchy resolved post-session) | This document v1.0 |

---

*Deep//Walk by Third Switch — Assurance at Scale*  
*© 2026 Third Switch, LLC.*  
*This document is internal reference. Not for external distribution without review.*
