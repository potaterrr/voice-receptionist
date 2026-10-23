# End-of-Call Report Contract (Vapi-compatible)

All three platform branches (Make / n8n / Zapier) consume this exact shape.
When Vapi goes live later, the assistant is configured to emit exactly this.

## Endpoint
- Method: POST, Content-Type: application/json

## Payload

```json
{
  "message": {
    "type": "end-of-call-report",
    "endedReason": "customer-ended-call",
    "call": {
      "id": "<unique-call-id>",
      "customer": { "number": "+639171234567" }
    },
    "artifact": {
      "transcript": "AI: ... Customer: ...",
      "structuredData": {
        "caller_name": "Maria Santos",
        "callback_number": "+639171234567",
        "intent": "book_appointment | question | complaint",
        "service": "general_consultation | follow_up | annual_physical | vaccination | null",
        "preferred_slot": "2026-08-27T14:00 (ISO local, Asia/Manila) or null"
      }
    },
    "summary": "One-line call summary"
  }
}
```

## Branch behavior (all platforms identical)
1. Ignore if `message.type != "end-of-call-report"`
2. Extract `artifact.structuredData`
3. If `intent == book_appointment` AND `preferred_slot` present:
   - Conflict-check calendar → if free: create event "Clinic: {service} — {caller_name}"
   - If taken: log status `conflict`, alert owner with fallback note
4. Else: log status `no_booking` (question/complaint routing)
5. Always: append row to call log + send owner alert via configured channel
6. Alert channel config:
   - Make: Data Store record (`channel`, `chat_id`, `alert_email`)
   - n8n: Set node "Config" (`channel` = telegram | email)
   - Zapier: Sheets "Config" tab lookup
