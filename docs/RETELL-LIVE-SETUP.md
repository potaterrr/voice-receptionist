# Going Live with Real Calls (Retell) — Plug-in Checklist

Everything downstream is already built and simulated. This doc wires real telephony.
Retell offers trial credit; exact amounts and per-minute pricing change, so check the
dashboard before you start. A Retell number is the fastest path; importing an existing
Twilio number is optional.

## 1. Retell account
- Sign up at `retellai.com` → dashboard.
- Copy your **API key** (used for webhook signature verification and API calls).

## 2. Get a phone number
Two options:

**A. Buy a Retell number (simplest)**
- Dashboard → Phone Numbers → buy a number in your region.

**B. Import an existing Twilio number (optional)**
- Have the Twilio **Account SID**, **Auth Token** and number in E.164 format ready.
- Dashboard → Phone Numbers → import from Twilio.
- Retell points the number's voice webhook at the agent; do not also point Twilio at
  another voice provider.

## 3. Create the receptionist agent
Dashboard → Agents → create a **single-prompt voice agent**.

Key settings:
- **Model:** a low-latency model such as `gemini-2.5-flash` (or your preferred provider).
- **Voice:** any stock voice that fits the clinic.
- **First message:** "Thank you for calling Potaterrr Clinic! How can I help you today?"
- **System prompt:**
  You are the receptionist for Potaterrr Clinic. Hours Mon–Fri 8:00–17:00.
  Services: General consultation (30 min) · Follow-up (15 min) · Annual physical (45 min) · Vaccination (15 min).
  Politely answer questions about services and hours. Do not quote prices or confirm insurance coverage —
  offer to have the front desk call the patient back. If the caller wants an appointment,
  collect: their name, callback number, preferred service and day/time. Confirm details back.
  If asked something you don't know, offer to have the front desk call back.

## 4. Configure Post Call Extraction
In the agent editor, expand **Post call extraction** → **Add** a field for each item:

| Field | Type | Description |
|---|---|---|
| `caller_name` | Text | The caller's name. Empty if not given. |
| `callback_number` | Text | Best callback number. Use the caller's number if they did not give another. |
| `intent` | Selector | One of `book_appointment`, `question`, `complaint`. |
| `service` | Selector | One of `general_consultation`, `follow_up`, `annual_physical`, `vaccination`. Empty if not discussed. |
| `preferred_slot` | Text | The requested appointment start as an ISO 8601 datetime with timezone, or empty if no specific time was agreed. |

Retell also produces `call_summary`, `call_successful`, `user_sentiment` and
`in_voicemail` automatically. The workflow reads the custom fields from
`call.call_analysis.custom_analysis_data`.

> Retell does not populate custom fields for calls that never connected. The workflow
> treats missing values as `no_booking`.

## 5. Register the webhook
Use an **agent-level webhook** so only this agent posts to the branch:

- Agent settings → Webhooks → URL = the branch URL below.
- Events: select **`call_analyzed`** only. This is the event that includes
  `call_analysis`; `call_ended` fires earlier without it.
- The endpoint must be public HTTPS. Retell blocks localhost, private IP ranges and
  cloud metadata addresses.

| Branch | Server URL |
|---|---|
| Make | `<YOUR-MAKE-WEBHOOK-URL>` (your scenario's Custom Webhook URL) |
| n8n* | `https://<public-tunnel>/webhook/voice-receptionist-n8n` |
| Zapier | your Catch Hook URL after building the Zap |

\* n8n needs a tunnel (ngrok/cloudflared) when it runs locally.

> These webhook URLs are unauthenticated endpoints — anyone who has one can POST fake
> bookings to your calendar. Never commit real URLs publicly. For production, verify the
> `x-retell-signature` header against your API key at a proxy or in a Code node before
> trusting the payload.

### Why the n8n branch responds early
Retell times out after **10 seconds** and retries non-2xx responses up to **3 times**.
The n8n workflow sends `200` immediately after the webhook, then continues processing.
It also looks up `call_id` in the call log and skips duplicates, so retries cannot create
two calendar events or two log rows.

## 6. Test ladder (cheapest first)
1. Simulated payloads (free) — already green ✅
2. **Web call** from the Retell dashboard (talk to the agent in the browser).
3. One real dial to the number → confirm the `call_analyzed` webhook:
   calendar event appears, alert lands, and one log row is written.
4. Replay the same payload → the duplicate guard should skip it.

## Cost reality check
- You pay for Retell platform usage, speech-to-text/text-to-speech, the LLM, and the
  phone number. Trial credit and rates vary by region and model — check current pricing.
- Use a cheap model and short test calls while validating.
