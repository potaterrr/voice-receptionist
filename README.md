# AI Voice Receptionist — Potaterrr Salon 💅🥔

> An AI receptionist that answers missed calls, books appointments, and alerts the owner —
> **built three times** (Make / n8n / Zapier) so you can compare platforms before you buy.
> Voice layer: [Vapi](https://vapi.ai) + Twilio. Runs today in simulation; live calls are a plug-in away.

```
Caller → missed/after-hours → [Vapi voice agent + Twilio number]
                    │  end-of-call webhook (structured data)
                    ▼
        ┌───────────────┼──────────────────┐
        ▼               ▼                  ▼
   MAKE branch      n8n branch        ZAPIER branch
   scenario         self-hosted       Catch Hook Zap
        │               │                  │
        └── parse → conflict-check → book Google Cal → log → alert owner ──┘
```

**Only one branch is live at a time** — flip by changing a single Server URL in Vapi.

## What it does

1. Caller dials; nobody picks up (or after hours) → AI receptionist answers instantly
2. Converses naturally: answers service/pricing questions, captures name, callback number, intent, preferred slot
3. End of call → structured report hits your platform's webhook
4. Pipeline checks calendar conflicts → **books the slot** → logs the call → Telegram/Gmail alert with summary

## The three implementations

| | [Make](https://make.com) | [n8n](https://n8n.io) (self-hosted) | [Zapier](https://zapier.com) |
|---|---|---|---|
| Trigger | Custom Webhook | Webhook node | Catch Hook |
| Config pattern | Data Store record | `Config` Set node (`channel`: telegram/email) | Sheets "Config" tab |
| Booking | Google Calendar freebusy + create | Google Calendar availability + create | Calendar double-book guard |
| Log | Make Data Store | Built-in Data Table | Google Sheet |
| Owner alert | Telegram bot | Telegram bot / Gmail switch | Gmail |

Full comparison with real-world findings: [`docs/PROS-AND-CONS.md`](docs/PROS-AND-CONS.md)

## Repo layout

```
contract/end-of-call-report.md   # the JSON contract every branch consumes
simulator/payload-{1,2,3}-*.json # test calls: bookable / question / past-slot
docs/VAPI-LIVE-SETUP.md          # plug in real calling (Vapi + Twilio trial, no card)
docs/ZAPIER-BUILD-SPEC.md        # click-by-click manual build for the Zapier branch
docs/PROS-AND-CONS.md            # platform scorecard for your pitch deck
README.md                        # you are here
```

## Simulate a call (free, no phone needed)

```bash
curl -X POST <BRANCH_WEBHOOK_URL> -H "Content-Type: application/json" \
  -d @simulator/payload-1-bookable.json
```

Expect: calendar event created · owner alerted · call logged.

> ⚠️ Branch webhook URLs are unauthenticated endpoints. Keep them secret.

## Demo persona

**Potaterrr Salon** 💅 — Mon–Sat 9:00–18:00
Haircut ₱350/45min · Hair Color ₱1,200/90min · Treatment ₱800/60min · Manicure ₱300/45min

Swap in any business by editing the assistant prompt + services config per branch.

## Status & cost

- ✅ Simulated end-to-end on Make and n8n (booking, logging, alerts all verified)
- 🧪 Zero cost during development — Vapi trial credit (~60+ min) covers live-call testing
- 💰 At scale: ~$0.05/min Vapi hosting + STT/TTS; BYO Gemini key keeps the brain free

Built by [@potaterrr](https://github.com/potaterrr) — Python automation dev who mashes the boring stuff so you don't have to. 🥔
