# CACS Decision Lane — Pilot Onboarding Guide
*For Lead Engineers at pilot partner organizations*

## 5-Minute Quickstart (Postman)
1. Download the file: CACS_PILOT_POSTMAN.json
2. Open Postman → click Import → select the file
3. The collection "CACS Decision Lane — Pilot" appears with 3 requests
4. Open "Success (Cleared)" → click Send
5. You will receive a 200 response with decision: "CLEARED"
6. Open "Fail (Blocked)" → click Send
7. You will receive a 200 response with decision: "BLOCKED"
8. Open "Unauthorized (Wrong Key)" → click Send
9. You will receive a 401 response with error: "Unauthorized"
Quickstart complete. Your integration is verified.

## Endpoint
POST https://cacs-decision-api.onrender.com/api/v1/decision

## Authentication
All requests require a Bearer token:
Authorization: Bearer YOUR_API_KEY

Contact donceletta@getcacs.com to receive your API key.

## Request Format
```json
{
  "participant_id": "string",
  "org_id": "string",
  "funding_amount": 4800,
  "docs": {
    "photoId":     { "status": "verified" },
    "ssn":         { "status": "verified" },
    "birth":       { "status": "verified" },
    "cert":        { "status": "verified" },
    "eligibility": { "status": "verified" }
  },
  "callback_url": "https://your-system.com/webhook"
}
```

## Document Status Values
verified | uploaded | pending | missing | expired | invalid

## Response Format
```json
{
  "clearance_id":   "CLEAR-2026-XXXXXXXXX",
  "participant_id": "string",
  "org_id":         "string",
  "decision":       "CLEARED | BLOCKED | ACTION REQUIRED | PENDING",
  "reason":         "string",
  "consequence":    "string",
  "ownership":      "string",
  "funding_status": "released | locked | hold",
  "funding_amount": 4800,
  "requested_at":   "ISO timestamp",
  "decided_at":     "ISO timestamp"
}
```

## Decision Rules
- BLOCKED: photoId, ssn, or birth is missing/invalid OR cert is invalid → funding locked
- ACTION REQUIRED: any document expired OR non-critical document missing → funding locked
- PENDING: all documents present but not all verified → funding on hold
- CLEARED: all documents verified, no flags → funding released

## Copy-Paste: Node.js
```js
const fetch = require('node-fetch');

async function runClearanceCheck(participantId, orgId, docs) {
  const response = await fetch('https://cacs-decision-api.onrender.com/api/v1/decision', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer YOUR_API_KEY'
    },
    body: JSON.stringify({
      participant_id: participantId,
      org_id: orgId,
      funding_amount: 4800,
      docs: docs
    })
  });
  const decision = await response.json();
  console.log('Decision:', decision.decision);
  console.log('Funding Status:', decision.funding_status);
  return decision;
}

// Example usage
runClearanceCheck('PARTICIPANT-001', 'org_001', {
  photoId:     { status: 'verified' },
  ssn:         { status: 'verified' },
  birth:       { status: 'verified' },
  cert:        { status: 'verified' },
  eligibility: { status: 'verified' }
});
```

## Copy-Paste: Python
```python
import requests

def run_clearance_check(participant_id, org_id, docs):
    response = requests.post(
        'https://cacs-decision-api.onrender.com/api/v1/decision',
        headers={
            'Content-Type': 'application/json',
            'Authorization': 'Bearer YOUR_API_KEY'
        },
        json={
            'participant_id': participant_id,
            'org_id': org_id,
            'funding_amount': 4800,
            'docs': docs
        }
    )
    decision = response.json()
    print(f"Decision: {decision['decision']}")
    print(f"Funding Status: {decision['funding_status']}")
    return decision

# Example usage
result = run_clearance_check('PARTICIPANT-001', 'org_001', {
    'photoId':     {'status': 'verified'},
    'ssn':         {'status': 'verified'},
    'birth':       {'status': 'verified'},
    'cert':        {'status': 'verified'},
    'eligibility': {'status': 'verified'}
})
```

## Webhook Support
Include callback_url in the request body to receive the decision
result via POST to your endpoint after it is logged.

## Security & Auditability
- Every request requires a Bearer token validated server-side
- Invalid or missing tokens return 401 Unauthorized immediately
- Every decision is permanently written to a Supabase PostgreSQL
  database with clearance_id, timestamps, and full decision output
- The audit log cannot be edited or deleted via the API
- Each clearance_id is unique and timestamped to the millisecond

## Support
Email: donceletta@getcacs.com
Documentation: https://cacs-decision-api.onrender.com
