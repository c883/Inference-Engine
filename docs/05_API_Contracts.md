# 05 — API Contracts

**Document Identifier:** DW-T3-05  
**Tier:** 3 — Engineering Track  
**Location:** GitHub `docs/05_API_Contracts.md` (canonical)  
**Version:** 1.0  
**Date:** May 2026

**Architecture reference:** See `docs/SYSTEM_ARCHITECTURE_MASTER.md` for tier definitions and cross-reference rules.

**What this document owns:** All API endpoint definitions, request/response payload contracts, HTTP method and path conventions, validation rules, error codes, and authentication notes for the Deepwalk Discovery Studio backend.

**What this document does not own:** Field schemas are defined in `01_Discovery_Studio_Config_Spec`. Spatial model contract lives in `06_Spatial_Model_Spec`. Inference Engine stage logic lives in `docs/architecture/inference-engine-spec.md`. Classification and gate logic lives in Studio Design v3.4 Part 1 (Tier 2).

**Dependency note:** All field names, enum values, and data types in this document derive strictly from `01_Discovery_Studio_Config_Spec`. If a discrepancy exists between this document and `01`, `01` is correct. Raise a Tier 1 Decision Register entry to resolve it.

---

## Part 1 — General Conventions

### Base URL
```
/api/v1
```

All endpoints are versioned. V1 is the current production version.

### Authentication
All endpoints require a valid session token issued at session creation. Token is passed in the `Authorization` header:
```
Authorization: Bearer {session_token}
```

Authentication implementation details are outside the scope of this document.

### Content type
All requests and responses use `application/json`.

### Timestamps
All timestamps are ISO 8601 format in UTC: `2026-05-24T03:45:00Z`. The server clock is authoritative — client-submitted timestamps are accepted only for the `timestamp` field on `ObservationNode` and `BulldogObservation`, where the device clock governs. Device clock drift is flagged during Stage 6.5 Enrichment Review if it exceeds ±5 minutes of server-adjacent observations.

### Immutability
Raw observation payloads are immutable once accepted. The backend does not expose update or delete endpoints for submitted observations. Corrections are handled through the Enrichment Review workflow. See `07_Review_Workflow_Spec` (Tier 2).

---

## Part 2 — Session Management

### POST /sessions — Create Session

Creates a new Discovery Session and returns the `session_id` and `session_token` used for all subsequent submissions.

**Request body:**
```json
{
  "site_id":                    "STRING  — Required. Identifier for the engagement site.",
  "facilitator_id":             "STRING  — Required. Identifier for the Facilitator creating the session.",
  "scheduled_start":            "TIMESTAMP — Required. Expected session start time.",
  "catalog_version":            "STRING  — Required. Measure Catalog version to lock for this session.",
  "signal_dictionary_version":  "STRING  — Required. Signal dictionary version to lock for this session.",
  "inference_ruleset_version":  "STRING  — Required. Inference Engine ruleset version to lock.",
  "space_id_options":           "ARRAY OF STRINGS — Required. Pre-loaded room codes from site directory.",
  "bulldog_timeout_overrides":  "OBJECT  — Optional. Site-specific timeout overrides per bulldog_type.",
  "enabled_fields":             "ARRAY OF STRINGS — Optional. Subset of ObservationNode fields to activate.",
  "required_fields":            "ARRAY OF STRINGS — Optional. Fields required for submission beyond the always-required set."
}
```

**Response — 201 Created:**
```json
{
  "session_id":     "STRING  — UUID. Use in all subsequent submissions for this session.",
  "session_token":  "STRING  — Bearer token. Expires at session close.",
  "created_at":     "TIMESTAMP",
  "status":         "active"
}
```

**Error responses:**

| Code | Error | Meaning |
|------|-------|---------|
| 400 | `ERR_SESSION_MISSING_REQUIRED_FIELD` | One or more required fields absent from request body |
| 400 | `ERR_SESSION_INVALID_CATALOG_VERSION` | `catalog_version` does not match a released catalog version |
| 400 | `ERR_SESSION_INVALID_RULESET_VERSION` | `inference_ruleset_version` does not match a released ruleset |
| 409 | `ERR_SESSION_ALREADY_ACTIVE` | An active session already exists for this `site_id` |

---

### POST /sessions/{session_id}/close — Close Session

Closes an active session. Configuration locks at this point — no further observations are accepted. Triggers the post-session pipeline (spatial resolution, Inference Engine queue, Enrichment Review initialization).

**Request body:** Empty.

**Response — 200 OK:**
```json
{
  "session_id":   "STRING",
  "closed_at":    "TIMESTAMP",
  "status":       "closed",
  "observation_count":       "INTEGER — Total ObservationNode submissions accepted.",
  "bulldog_observation_count":"INTEGER — Total BulldogObservation submissions accepted.",
  "inaccessible_area_count": "INTEGER — Total InaccessibleArea submissions accepted."
}
```

**Error responses:**

| Code | Error | Meaning |
|------|-------|---------|
| 404 | `ERR_SESSION_NOT_FOUND` | `session_id` does not exist |
| 409 | `ERR_SESSION_ALREADY_CLOSED` | Session is already closed |

---

## Part 3 — Observation Submission

### POST /sessions/{session_id}/observations — Submit Observation

Submits a single `ObservationNode`. Accepted immediately — pipeline processing is asynchronous.

**Request body — full field set:**
```json
{
  "session_id":          "STRING    — Required. Must match the session_id in the path.",
  "space_id":            "STRING    — Required. Sticky map label or ad-hoc anchor. See 06_Spatial_Model_Spec.",
  "timestamp":           "TIMESTAMP — Required. Device clock at point of submission. ISO 8601 UTC.",
  "observation_type":    "STRING    — Required. UI triage key. Must map to a valid equipment_class in the active catalog version.",
  "fixture_count":       "INTEGER   — Required. Minimum: 1.",
  "field_state_enum":    "STRING    — Optional. Default: UNKNOWN. Permitted: ON | OFF | AUTO | UNKNOWN.",
  "did_you_turn_it_off": "BOOLEAN   — Required.",
  "why_not":             "STRING    — Conditional. Required when did_you_turn_it_off is false. See permitted values below.",
  "surveyor_role":       "STRING    — Required. Permitted: M3 | LO | TV | SS | Facilitator.",
  "owl_count":           "INTEGER   — Optional. Default: 0. Minimum: 0.",
  "raw_notes":           "STRING    — Optional. Free text. Max 2000 characters.",
  "photo_refs":          "ARRAY     — Optional. Max 3 entries. Each entry: slot-prefixed URI string."
}
```

**`why_not` permitted values:**
```
"No Switch or Control Found"
"Requires Permission"
"Occupancy Sensor"
"Other"
null  (only when did_you_turn_it_off is true)
```

**`photo_refs` format:**
Each entry must be a slot-prefixed URI: `"context:https://..."`, `"subject:https://..."`, `"detail:https://..."`. Maximum one entry per slot.

**Response — 202 Accepted:**
```json
{
  "observation_id": "STRING  — Server-generated UUID for this observation.",
  "session_id":     "STRING",
  "received_at":    "TIMESTAMP — Server receipt time.",
  "status":         "queued"
}
```

**Error responses:**

| Code | Error | Meaning |
|------|-------|---------|
| 400 | `ERR_STAGE1_INVALID_UI_STRING` | `why_not` value does not match permitted enum |
| 400 | `ERR_OBS_MISSING_WHY_NOT` | `did_you_turn_it_off` is false and `why_not` is null |
| 400 | `ERR_OBS_MISSING_REQUIRED_FIELD` | One or more always-required fields absent |
| 400 | `ERR_OBS_INVALID_OBSERVATION_TYPE` | `observation_type` does not map to a valid `equipment_class` in the active catalog version |
| 400 | `ERR_OBS_INVALID_FIXTURE_COUNT` | `fixture_count` is less than 1 |
| 400 | `ERR_OBS_INVALID_ROLE` | `surveyor_role` not in permitted enum |
| 400 | `ERR_OBS_INVALID_FIELD_STATE` | `field_state_enum` value not in permitted enum |
| 400 | `ERR_OBS_INVALID_PHOTO_REF` | Photo ref entry missing slot prefix or malformed URI |
| 400 | `ERR_OBS_SESSION_MISMATCH` | `session_id` in body does not match path parameter |
| 409 | `ERR_SESSION_CLOSED` | Session is closed — no further observations accepted |
| 404 | `ERR_SESSION_NOT_FOUND` | `session_id` does not exist |

---

## Part 4 — Bulldog Observation Submission

### POST /sessions/{session_id}/bulldogs — Submit Bulldog Observation

Submits a single `BulldogObservation`. Accepted immediately.

**Request body:**
```json
{
  "session_id":      "STRING    — Required. Must match path.",
  "timestamp":       "TIMESTAMP — Required. Device clock. ISO 8601 UTC.",
  "space_id":        "STRING    — Required. Location where Bulldog was observed.",
  "bulldog_type":    "STRING    — Required. Permitted: Security | Cleaning | Construction | Contractor | Unknown.",
  "count":           "INTEGER   — Required. Default: 1. Minimum: 1.",
  "notes":           "STRING    — Optional. Max 1000 characters.",
  "badge_photo_ref": "STRING    — Optional. URI pointing to photo of worker badge or name tag."
}
```

**Response — 202 Accepted:**
```json
{
  "bulldog_observation_id": "STRING  — Server-generated UUID.",
  "session_id":             "STRING",
  "received_at":            "TIMESTAMP",
  "status":                 "queued"
}
```

**Error responses:**

| Code | Error | Meaning |
|------|-------|---------|
| 400 | `ERR_BULLDOG_MISSING_REQUIRED_FIELD` | One or more required fields absent |
| 400 | `ERR_BULLDOG_INVALID_TYPE` | `bulldog_type` not in permitted enum |
| 400 | `ERR_BULLDOG_INVALID_COUNT` | `count` is less than 1 |
| 409 | `ERR_SESSION_CLOSED` | Session is closed |
| 404 | `ERR_SESSION_NOT_FOUND` | `session_id` does not exist |

---

## Part 5 — Inaccessible Area Submission

### POST /sessions/{session_id}/inaccessible — Submit Inaccessible Area

Submits a single `InaccessibleArea` record. See `06_Spatial_Model_Spec` Part 7 for schema definition and reason enum values.

**Request body:**
```json
{
  "session_id":    "STRING    — Required. Must match path.",
  "timestamp":     "TIMESTAMP — Required. Device clock. ISO 8601 UTC.",
  "space_id":      "STRING    — Required. Best available identifier for the inaccessible space.",
  "reason":        "STRING    — Required. Permitted: Locked | Occupied | Requires Permission | Hazardous | Other.",
  "notes":         "STRING    — Optional. Max 1000 characters.",
  "surveyor_role": "STRING    — Required. Permitted: M3 | LO | TV | SS | Facilitator."
}
```

**Response — 202 Accepted:**
```json
{
  "inaccessible_area_id": "STRING  — Server-generated UUID.",
  "session_id":           "STRING",
  "received_at":          "TIMESTAMP",
  "status":               "queued"
}
```

**Error responses:**

| Code | Error | Meaning |
|------|-------|---------|
| 400 | `ERR_INACC_MISSING_REQUIRED_FIELD` | Required field absent |
| 400 | `ERR_INACC_INVALID_REASON` | `reason` not in permitted enum |
| 409 | `ERR_SESSION_CLOSED` | Session is closed |
| 404 | `ERR_SESSION_NOT_FOUND` | `session_id` does not exist |

---

## Part 6 — Findings Retrieval

### GET /sessions/{session_id}/findings — Retrieve Session Findings

Returns the flat tagged output array for a completed session. Available only after the Inference Engine pipeline has completed for the session. See `docs/architecture/inference-engine-spec.md` Section 5.10 for the full output schema.

**Query parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `domain` | STRING | Filter by domain: `Human`, `Boundary`, `Mechanical` |
| `action_path` | STRING | Filter by action path: `Green`, `Yellow`, `Red` |
| `is_guaranteed_saving` | BOOLEAN | Filter to guarantee pool only |
| `is_exempt_baseline` | BOOLEAN | Filter to exempt baseline records only |
| `floor` | STRING | Filter by resolved floor identifier |
| `zone` | STRING | Filter by resolved zone identifier |

**Response — 200 OK:**
```json
{
  "session_id":          "STRING",
  "pipeline_status":     "STRING — completed | in_progress | pending | error",
  "findings_count":      "INTEGER",
  "findings": [
    {
      "cluster_id":                 "STRING",
      "space_id":                   "STRING",
      "floor":                      "STRING",
      "zone":                       "STRING",
      "observation_type":           "STRING",
      "equipment_class":            "STRING",
      "assigned_domain":            "STRING  — Human | Boundary | Mechanical",
      "assigned_action_path":       "STRING  — Green | Yellow | Red",
      "annual_energy_savings_kwh":  "FLOAT",
      "annual_financial_savings_usd":"FLOAT",
      "is_guaranteed_saving":       "BOOLEAN",
      "is_exempt_baseline":         "BOOLEAN",
      "is_default_applied":         "BOOLEAN",
      "validation_status":          "STRING"
    }
  ],
  "portfolio_validation": {
    "total_calculated_savings_kwh": "FLOAT",
    "portfolio_spend_cap_kwh":      "FLOAT",
    "boundary_check_passed":        "BOOLEAN"
  },
  "isolated_review_bucket": [
    {
      "cluster_id":        "STRING",
      "why_not":           "Other",
      "validation_status": "FACILITATOR_REVIEW_REQUIRED"
    }
  ]
}
```

**Error responses:**

| Code | Error | Meaning |
|------|-------|---------|
| 404 | `ERR_SESSION_NOT_FOUND` | `session_id` does not exist |
| 409 | `ERR_PIPELINE_NOT_COMPLETE` | Inference Engine pipeline has not yet completed for this session |

---

## Part 7 — Session Status

### GET /sessions/{session_id} — Get Session Status

Returns the current status and summary counts for a session. Useful for monitoring pipeline progress after session close.

**Response — 200 OK:**
```json
{
  "session_id":              "STRING",
  "site_id":                 "STRING",
  "facilitator_id":          "STRING",
  "status":                  "STRING — active | closed | pipeline_complete | error",
  "created_at":              "TIMESTAMP",
  "closed_at":               "TIMESTAMP | null",
  "catalog_version":         "STRING",
  "inference_ruleset_version":"STRING",
  "observation_count":       "INTEGER",
  "bulldog_observation_count":"INTEGER",
  "inaccessible_area_count": "INTEGER",
  "pipeline_status":         "STRING — pending | in_progress | completed | error",
  "pipeline_completed_at":   "TIMESTAMP | null"
}
```

---

## Part 8 — Error Code Registry

All error codes used across all endpoints. Grouped by origin stage.

### Session errors
| Code | HTTP status | Description |
|------|-------------|-------------|
| `ERR_SESSION_MISSING_REQUIRED_FIELD` | 400 | Required session creation field absent |
| `ERR_SESSION_INVALID_CATALOG_VERSION` | 400 | Catalog version not found |
| `ERR_SESSION_INVALID_RULESET_VERSION` | 400 | Ruleset version not found |
| `ERR_SESSION_ALREADY_ACTIVE` | 409 | Active session exists for site |
| `ERR_SESSION_ALREADY_CLOSED` | 409 | Session already closed |
| `ERR_SESSION_NOT_FOUND` | 404 | session_id not found |
| `ERR_SESSION_CLOSED` | 409 | Submission rejected — session is closed |

### Stage 1 — Ingestion errors
| Code | HTTP status | Description |
|------|-------------|-------------|
| `ERR_STAGE1_INVALID_UI_STRING` | 400 | `why_not` value does not match permitted enum |
| `ERR_OBS_MISSING_WHY_NOT` | 400 | `did_you_turn_it_off` false with null `why_not` |
| `ERR_OBS_MISSING_REQUIRED_FIELD` | 400 | Always-required field absent |
| `ERR_OBS_INVALID_OBSERVATION_TYPE` | 400 | `observation_type` not in active catalog |
| `ERR_OBS_INVALID_FIXTURE_COUNT` | 400 | `fixture_count` less than 1 |
| `ERR_OBS_INVALID_ROLE` | 400 | `surveyor_role` not in permitted enum |
| `ERR_OBS_INVALID_FIELD_STATE` | 400 | `field_state_enum` not in permitted enum |
| `ERR_OBS_INVALID_PHOTO_REF` | 400 | Photo ref missing slot prefix or malformed URI |
| `ERR_OBS_SESSION_MISMATCH` | 400 | Body session_id does not match path session_id |

### Stage 3 errors
| Code | HTTP status | Description |
|------|-------------|-------------|
| `ERR_STAGE3_MISSING_SENSOR_CONSTANT` | 500 | Occupancy sensor timeout constant absent from portfolio config — pipeline halts for this node |

### Stage 4 errors
| Code | HTTP status | Description |
|------|-------------|-------------|
| `ERR_STAGE4_SPATIAL_CONTEXT_CONFLICT` | 500 | Conflicting `global_context` attributes across observations sharing same `space_id` + `equipment_class` — cluster flagged, pipeline halts for this cluster |

### Stage 6 errors
| Code | HTTP status | Description |
|------|-------------|-------------|
| `ERR_STAGE6_MISSING_PANEL_TOPOLOGY` | 500 | `associated_panel_id` is null when required for `CENTRAL_PANEL_LOOP` routing |
| `ERR_STAGE6_CONTROL_DOMAIN_UNRESOLVED` | 500 | Control domain cannot be resolved — cluster flagged |

### Stage 10 errors
| Code | HTTP status | Description |
|------|-------------|-------------|
| `ERR_STAGE10_PORTFOLIO_CAP_EXCEEDED` | 500 | Total calculated savings exceed `portfolio_spend_cap_kwh` — payload distribution blocked; calibration error flag triggered |

### Bulldog and inaccessible area errors
| Code | HTTP status | Description |
|------|-------------|-------------|
| `ERR_BULLDOG_MISSING_REQUIRED_FIELD` | 400 | Required Bulldog field absent |
| `ERR_BULLDOG_INVALID_TYPE` | 400 | `bulldog_type` not in permitted enum |
| `ERR_BULLDOG_INVALID_COUNT` | 400 | `count` less than 1 |
| `ERR_INACC_MISSING_REQUIRED_FIELD` | 400 | Required inaccessible area field absent |
| `ERR_INACC_INVALID_REASON` | 400 | `reason` not in permitted enum |

### Pipeline errors
| Code | HTTP status | Description |
|------|-------------|-------------|
| `ERR_PIPELINE_NOT_COMPLETE` | 409 | Findings retrieval attempted before pipeline completion |

---

## Part 9 — Endpoint Summary

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/api/v1/sessions` | Create session |
| POST | `/api/v1/sessions/{id}/close` | Close session |
| GET | `/api/v1/sessions/{id}` | Get session status |
| POST | `/api/v1/sessions/{id}/observations` | Submit observation |
| POST | `/api/v1/sessions/{id}/bulldogs` | Submit Bulldog observation |
| POST | `/api/v1/sessions/{id}/inaccessible` | Submit inaccessible area |
| GET | `/api/v1/sessions/{id}/findings` | Retrieve findings (post-pipeline) |

---

*Deep//Walk by Third Switch — Assurance at Scale*  
*© 2026 Third Switch, LLC.*  
*This document is internal reference. Not for external distribution without review.*
