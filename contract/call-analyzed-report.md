# Call Analyzed Report Contract (Retell)

All three platform branches (Make / n8n / Zapier) consume this shape.

Retell emits several webhook events. This build only processes **`call_analyzed`**:
`call_ended` fires earlier but does **not** contain `call_analysis`, so summaries and
custom fields are missing there. Configure the agent's `webhook_events` to
`["call_analyzed"]` unless you also need lifecycle events elsewhere.

## Endpoint
- Method: `POST`, `Content-Type: application/json`
- Header: `x-retell-signature` (verify against your Retell API key in production)
- Timeout: 10 seconds. Non-2xx responses are retried up to 3 times, so handlers must be
  **idempotent** on `call_id`.

## Payload

```json
{
  "event": "call_analyzed",
  "call": {
    "call_id": "<unique-call-id>",
    "agent_id": "<retell-agent-id>",
    "from_number": "+639171234567",
    "to_number": "+16315550123",
    "direction": "inbound",
    "call_status": "ended",
    "transcript": "Agent: ... Caller: ...",
    "call_analysis": {
      "call_summary": "One-line call summary",
      "call_successful": true,
      "user_sentiment": "Positive",
      "custom_analysis_data": {
        "caller_name": "Maria Santos",
        "callback_number": "+639171234567",
        "intent": "book_appointment | question | complaint",
        "service": "general_consultation | follow_up | annual_physical | vaccination | null",
        "preferred_slot": "2026-08-27T14:00:00+08:00 or null"
      }
    }
  }
}
```

`custom_analysis_data` keys match the Post Call Extraction field names configured on
the Retell agent. Retell does not populate them for calls that never connected, so the
handler must tolerate missing or null values.

## Branch behavior (all platforms identical)
1. Ignore if `event != "call_analyzed"`.
2. Deduplicate on `call.call_id` (n8n and Make look it up in the call log; the archived
   Zapier branch relies on its filter and should add a lookup before production use).
3. Extract `call.call_analysis.custom_analysis_data`; fall back to `call.from_number`
   when `callback_number` is empty.
4. If `intent == book_appointment` AND `preferred_slot` is present and in the future:
   - Conflict-check the calendar.
   - If free: create event `Potaterrr Clinic: {service} — {caller_name}`.
   - If taken: log status `conflict`, alert the owner with a fallback note.
5. Else: log status `no_booking` (question/complaint routing).
6. Always: append a row to the call log and send the owner alert via the configured channel.
7. Acknowledge with `2xx` as soon as possible. The n8n branch responds `200` before
   extraction so Retell does not time out; the rest of the workflow continues after the
   response.
8. Alert channel config:
   - Make: Data Store record (`channel`, `chat_id`, `alert_email`)
   - n8n: Set node `Config` (`channel` = `telegram | email`)
   - Zapier: Sheets `Config` tab lookup

## Field mapping from the old Vapi contract

| Vapi | Retell |
|---|---|
| `message.type == "end-of-call-report"` | `event == "call_analyzed"` |
| `message.call.id` | `call.call_id` |
| `message.call.customer.number` | `call.from_number` |
| `message.artifact.structuredData` | `call.call_analysis.custom_analysis_data` |
| `message.artifact.transcript` | `call.transcript` |
| `message.summary` | `call.call_analysis.call_summary` |
| `message.endedReason` | `call.disconnection_reason` |
