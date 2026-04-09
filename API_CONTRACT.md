# CACS Decision Engine — API Contract

## Endpoint (planned)
POST /api/v1/decision

## Request Body
```json
{
  "participant_id": "string — required",
  "org_id": "string — required",
  "funding_amount": "number — required (replaces hardcoded 4800)",
  "docs": {
    "photoId": { "status": "string — verified | uploaded | pending | missing | expired | invalid" },
    "ssn": { "status": "string — verified | uploaded | pending | missing | expired | invalid" },
    "birth": { "status": "string — verified | uploaded | pending | missing | expired | invalid" },
    "cert": { "status": "string — verified | uploaded | pending | missing | expired | invalid" },
    "eligibility": { "status": "string — verified | uploaded | pending | missing | expired | invalid" }
  }
}
```

## Response Body
```json
{
  "clearance_id": "string — generated",
  "participant_id": "string",
  "org_id": "string",
  "decision": "CLEARED | BLOCKED | ACTION REQUIRED | PENDING",
  "reason": "string",
  "consequence": "string",
  "ownership": "string",
  "funding_status": "released | locked | hold",
  "funding_amount": "number — 0 if not CLEARED",
  "requested_at": "ISO timestamp",
  "decided_at": "ISO timestamp"
}
```

## Decision Rules
- BLOCKED: photoId, ssn, or birth is missing/invalid OR cert is invalid
- ACTION REQUIRED: any document is expired OR any non-critical document is missing
- PENDING: all documents present but not all verified
- CLEARED: all documents verified or uploaded, no flags

## Status Values
verified | uploaded | pending | missing | expired | invalid
