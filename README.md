# AI Voice Receptionist for Clinics

**Answer every missed and after-hours call. Book the appointment. Alert the front desk.**

An AI receptionist for clinics and appointment-based practices — built on **Make, n8n or Zapier** over one shared **Vapi + Twilio** voice layer. It picks up when the front desk can't, captures structured intake, checks the calendar, books the slot, logs the call and alerts your team.

<p align="left">
  <a href="#compliance--data-handling">
    <img src="https://img.shields.io/badge/%F0%9F%94%92_HIPAA--compliant_build_available-2ea886?style=for-the-badge" alt="HIPAA-compliant build available" />
  </a>
  <img src="https://img.shields.io/badge/Vapi_%2B_Twilio-voice_layer-6D00CC?style=for-the-badge" alt="Vapi plus Twilio" />
  <img src="https://img.shields.io/badge/Make_%7C_n8n_%7C_Zapier-three_builds-2088FF?style=for-the-badge" alt="Make, n8n and Zapier" />
</p>

> **Demo status:** simulated end-to-end today on Make and n8n. Live phone calls are a [documented Vapi + Twilio setup](docs/VAPI-LIVE-SETUP.md) away, and a pilot configures a real number.

---

## The problem

A clinic's phone is a revenue and care-access channel. When it rings out:

- New patients call the next practice on their list.
- After-hours callers leave no message and don't call back.
- The front desk juggles check-ins, walk-ins and the phone at the same time.
- Staff re-enter the same details from voicemail into the calendar.
- Nobody can tell you how many calls were missed, or why.

An AI receptionist doesn't replace your team. It covers the calls that would otherwise go to voicemail — instantly, consistently, and with a written record.

## What patients experience

```text
Patient calls
   │  no answer / after hours / front desk busy
   ▼
[ Vapi voice agent + Twilio number ]
   │  natural conversation · name · callback · intent · preferred slot
   ▼
[ Make  |  n8n  |  Zapier ]
   │  validate → conflict-check calendar → book → log → alert staff
   ▼
Appointment on the calendar  +  call summary for the front desk
```

1. The patient calls the clinic's existing number.
2. If nobody picks up (or it's after hours), the AI answers instantly and conversationally.
3. It answers questions about services and hours, then captures **name, callback number, intent, service and preferred slot** as structured data.
4. The workflow checks the calendar and books the appointment if the slot is free.
5. Staff get a summary alert; every call is logged — including questions and conflicts that need a human.

## What the clinic gets

- **24/7 coverage** for missed and after-hours calls — no hold music, no voicemail void.
- **Structured intake**, not a scribbled message: name, number, intent, service and preferred time.
- **Conflict-checked bookings** against Google Calendar, with a real event created for free slots.
- **A call log** for every outcome: `booked`, `conflict`, `no_booking`.
- **Staff alerts** through Telegram or email, configurable per platform.
- **One voice layer, three engines** — choose the platform that matches who maintains it.

## See it work (no phone, no PHI)

The simulator ships with three synthetic calls. No real patient data is used.

| Scenario | What happens | Result |
|---|---|---|
| **Bookable** | Caller asks for a general consultation tomorrow at 2 PM | Calendar event created · staff alerted · call logged |
| **Question** | Caller asks how long an annual physical takes | No booking · call logged · staff alerted for follow-up |
| **Past slot** | Caller asks for a follow-up last Monday | Not bookable · logged as conflict · staff alerted |

Run one yourself:

```bash
curl -X POST <BRANCH_WEBHOOK_URL> -H "Content-Type: application/json" \
  -d @simulator/payload-1-bookable.json
```

> Branch webhook URLs are unauthenticated endpoints. Keep them secret and never commit real URLs.

The included persona is **Potaterrr Clinic** (Mon–Fri, 08:00–17:00) with general consultation, follow-up, annual physical and vaccination services. Swap in any practice by editing the assistant prompt and service config per branch.

## Sample build

<p align="center">
  <img src="docs/samples/n8n-workflow.png" alt="n8n workflow canvas: webhook, parse, end-of-call report, calendar check, logging and staff alerts" width="720" />
</p>

The n8n branch end to end: webhook → parse → end-of-call report → calendar check → book/log → staff alert. Captured from the working build; the public demo runs on synthetic data only.

## Pick your engine

All three branches consume the same end-of-call report, so the comparison is apples-to-apples. Only one branch is live at a time — switch by changing one Server URL in Vapi.

| Engine | Best for | Trade-off |
|---|---|---|
| **Make.com** | Fastest build and visual debugging | Op budgets and expression quirks |
| **n8n** (self-hosted) | Data ownership and no per-op costs | You own uptime, updates and security |
| **Zapier** | Handing a readable automation to non-technical owners | Click-ops only; task costs add up |

Full scorecard: [`docs/PROS-AND-CONS.md`](docs/PROS-AND-CONS.md)

## Compliance & data handling

**The public demo is not HIPAA compliant.** It is a portfolio reference built with synthetic data, not a production healthcare system. It has no Business Associate Agreements (BAAs), no PHI safeguards, and **must not be used with real patient data**.

**A HIPAA-compliant build is available as a scoped engagement.** For clinics that handle protected health information, the build is designed around:

- BAAs with every vendor that touches PHI.
- Role-based access controls and least-privilege credentials.
- Audit logging, retention policies and access reviews.
- PHI-safe telephony, storage and alerting.
- A compliance review before go-live.

Compliance is established per deployment and depends on your practice's policies, vendors and BAAs — it is not claimed by this public repository. We confirm what is in scope on the fit call.

## Engagement options

No prices are listed here — scope and quote follow a short discovery call.

### 01 · Pilot — prove it on one number

Best for a single location that wants evidence before rolling out.

- One platform (Make or n8n).
- Google Calendar booking and staff alerts.
- Call logging and three synthetic test scenarios.
- Go-live checklist and handover notes.

**Typical timeline:** 1–2 weeks.

### 02 · Practice — fit the real front desk

Best for a growing practice with multiple services or providers.

- Everything in Pilot.
- Custom intake fields and service menu.
- Provider or multi-calendar routing.
- Escalation rules and a reporting sheet.
- One revision round after the pilot.

**Typical timeline:** 2–4 weeks.

### 03 · HIPAA Build — designed for PHI

Best for healthcare practices that will handle real patient information.

- Everything in Practice.
- Scoped HIPAA-compliant architecture.
- BAAs, access controls, audit logging and retention.
- PHI-safe infrastructure and vendor review.
- Compliance review before go-live.

**Typical timeline:** 4–6 weeks plus compliance review.

> Timelines assume one decision-maker, timely access to calendars and phone systems, and one revision round. Final scope is confirmed after discovery.

## How a rollout works

1. **Fit call (20 min)** — goals, call volume, platforms, compliance needs.
2. **Discovery** — services, hours, intake fields, calendar ownership, alert routing.
3. **Build** — one platform, configured for your practice.
4. **Simulate** — synthetic scenarios reviewed with your team.
5. **Pilot** — real number, limited hours, measured against the old flow.
6. **Go live + handover** — documentation, access, and a support path.

## What I need from you

- Services, hours and locations.
- Calendar access (and who owns it).
- The phone number or forwarding setup.
- Alert channel and escalation contact.
- One decision-maker for sign-off.
- For HIPAA work: policy contacts, vendor constraints and BAA requirements.

## FAQ

**Is this answering real calls today?**  
Not in this public repo. The pipeline is simulated end-to-end on Make and n8n. Live telephony is documented and configured during a pilot.

**Does it replace front-desk staff?**  
No. It covers overflow and after-hours calls, then hands questions and exceptions back to your team with a summary.

**Which platform should we choose?**  
Client-managed → Zapier. Agency-managed → Make. Self-hosted or privacy-sensitive → n8n. The [scorecard](docs/PROS-AND-CONS.md) breaks it down.

**Is it HIPAA compliant?**  
The public demo is not. A HIPAA-compliant build is available as a scoped engagement, with BAAs, access controls and audit logging established per deployment.

**Can it integrate with our EHR?**  
The current build books Google Calendar and alerts staff. EHR or PM-system integrations are scoped per engagement after discovery.

**Who owns the workflow and data?**  
You do. The n8n branch is fully self-hostable, and every branch is exportable.

**What does it cost?**  
Scope and quote follow the fit call. No prices are published because the compliant and multi-location scopes differ substantially.

---

## Technical appendix

<details>
<summary>Repo layout, contract and build docs</summary>

```text
contract/end-of-call-report.md    # JSON contract every branch consumes
simulator/payload-{1,2,3}-*.json  # synthetic calls: bookable / question / past-slot
docs/VAPI-LIVE-SETUP.md           # plug in real calling (Vapi + Twilio trial)
docs/ZAPIER-BUILD-SPEC.md         # click-by-click build for the Zapier branch
docs/PROS-AND-CONS.md             # platform scorecard
README.md                         # you are here
```

- **Contract:** [`contract/end-of-call-report.md`](contract/end-of-call-report.md)
- **Live-call setup:** [`docs/VAPI-LIVE-SETUP.md`](docs/VAPI-LIVE-SETUP.md)
- **Zapier build:** [`docs/ZAPIER-BUILD-SPEC.md`](docs/ZAPIER-BUILD-SPEC.md)
- **Platform scorecard:** [`docs/PROS-AND-CONS.md`](docs/PROS-AND-CONS.md)

The demo persona is synthetic. Never send real patient data to the public simulator or an unauthenticated webhook.

</details>

## Book a fit call

Tell me your call volume, services and compliance needs. I'll tell you whether a pilot makes sense.

<!-- TODO(Christian): replace this placeholder with your real Cal.com/Calendly link before sending to clients. -->
**[Book a fit call →](https://cal.com/your-handle/intro-call)**

Prefer to message first? WhatsApp is fastest.

- 💬 WhatsApp: [@potaiterrr](https://wa.me/potaiterrr) *(preferred)*
- ✈️ Telegram: [@portaterrrr](https://t.me/portaterrrr)
- 📧 Email: [kristiandyanbusiness@gmail.com](mailto:kristiandyanbusiness@gmail.com)
- 📘 Facebook: [Potaterrr](https://www.facebook.com/profile.php?id=61593886946042)
- 💬 Discord: `potaterrr`

---

Built by [@potaterrr](https://github.com/potaterrr) — Python automation dev who mashes the boring stuff so you don't have to. 🥔
