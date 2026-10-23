# Zapier Branch — Manual Build Spec (10 min)

Zapier Zaps can't be created via API, so build this once by hand. Exact steps below.

## Prereq
- Zapier plan must include **Webhooks by Zapier → Catch Hook** (most paid plans; check yours)

## Zap: "Voice Receptionist - Potaterrr Clinic"

### Step 1 — Trigger: Webhooks by Zapier → Catch Hook
- Copy the generated **custom webhook URL** → this is the branch's Server URL

### Step 2 — Filter (only proceed if…)
- `message type` (Text) Exactly equals `end-of-call-report`

### Step 3 — Google Calendar → Find event(s)
- Calendar: your primary
- Search From: `message artifact structuredData preferred_slot`
- Search To: same + 1 hour (use Formatter Date/Time add 1h if needed)
- ✅ "Create Google Calendar event if it doesn't exist yet?" = false

### Step 4 — Paths
**Path A "Free & bookable"** (only continue if Step 3 found nothing AND intent = book_appointment)
1. Google Calendar → Create detailed event
   - Title: `Potaterrr Clinic: {{service}} — {{caller_name}}`
   - Start/End from structuredData slot
   - Description: caller name + number + call id + "Booked by AI Receptionist (Zapier branch)"
2. Google Sheets → Create spreadsheet row(s)
   - Spreadsheet: create "Receptionist Calls (Zapier)" with headers:
     call_id | caller_name | callback_number | intent | service | preferred_slot | status | summary | platform
   - status = `booked`, platform = `zapier`
3. Gmail → Send email to `kristiandyanbusiness@gmail.com`
   - Subject: `📞 Potaterrr Clinic — BOOKED`
   - Body: caller, service, slot, summary + calendar link

**Path B "Conflict"** (Step 3 found an event AND intent = book_appointment)
1. Sheets row: status = `conflict`
2. Gmail: subject `⚠️ Slot taken — needs follow-up`

**Path C "Other intents"** (intent is not book_appointment)
1. Sheets row: status = `no_booking`
2. Gmail: subject `💬 Inquiry — no booking`

## Config-for-clients tip
Add a "Config" tab in the sheet (alert_email, business_hours…) and read it with a
Sheets lookup step as Step 2 — that's the sellable settings screen.
