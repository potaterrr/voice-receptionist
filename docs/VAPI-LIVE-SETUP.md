# Going Live with Real Calls (Vapi + Twilio) — Plug-in Checklist

Everything downstream is already built and simulated. This doc wires real telephony.
No card needed for Vapi trial ($10 credit ≈ 60+ min) or Twilio trial.

## 1. Vapi account
- Sign up → dashboard.vapi.ai → copy **Private Key** (Settings → API Keys)

## 2. Twilio account
- console.twilio.com → note **Account SID** + **Auth Token**
- Get a **trial number** (Phone Numbers → Buy a number — free on trial, US/CA easiest)

## 3. Import the number into Vapi
- Dashboard → Phone Numbers → Create Phone Number → **Import Twilio**
- Paste number (+E.164), SID, Auth Token → Import
  - Vapi auto-points the Twilio number's voice webhook at Vapi

## 4. Create the Receptionist assistant (dashboard → Assistants → Create)
Key settings:
- **Model:** Google `gemini-2.5-flash` + paste a Gemini API key as BYO provider key ($0 LLM cost)
- **Voice:** any stock voice (e.g., 11labs or Azure cheapest tier)
- **First message:** "Thank you for calling Potaterrr Clinic! How can I help you today?"
- **System prompt:**
  You are the receptionist for Potaterrr Clinic. Hours Mon–Fri 8:00–17:00.
  Services: General consultation (30 min) · Follow-up (15 min) · Annual physical (45 min) · Vaccination (15 min).
  Politely answer questions about services and hours. Do not quote prices or confirm insurance coverage —
  offer to have the front desk call the patient back. If the caller wants an appointment,
  collect: their name, callback number, preferred service and day/time. Confirm details back.
  If asked something you don't know, offer to have the front desk call back.
- **End-of-call structured data** (Structured Data Plan / success schema):
```json
{
  "caller_name": "string|null",
  "callback_number": "string",
  "intent": "book_appointment | question | complaint",
  "service": "general_consultation|follow_up|annual_physical|vaccination|null",
  "preferred_slot": "ISO datetime or null"
}
```
- Enable **Summary** in artifact settings
- **Server URL** (assistant-level): set to ONE branch hook below

## 5. Choose the live branch (flip anytime)
| Branch | Server URL |
|---|---|
| Make | `<YOUR-MAKE-WEBHOOK-URL>` (your scenario's Custom Webhook URL) |
| n8n* | `http://<public-tunnel>/webhook/voice-receptionist-n8n` |
| Zapier | your Catch Hook URL after building the Zap |

> ⚠️ These webhook URLs are unauthenticated endpoints — anyone who has one can POST
> fake bookings to your calendar. Never commit real URLs publicly.

*n8n needs a tunnel (ngrok/cloudflared) since it runs locally.

## 6. Test ladder (cheapest first)
1. Simulated payloads (free) — already green ✅
2. Vapi **test call** from dashboard (talk to assistant in browser)
3. One real dial to the Twilio number → watch end-of-call hit your branch:
   calendar event appears, Telegram/Gmail alert lands, log row written

## Cost reality check
- Trial covers ~20–60 test minutes; after that $0.05/min platform + STT/TTS
- BYO Gemini key keeps the brain free — biggest lever
