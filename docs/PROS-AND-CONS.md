# Platform Pros & Cons — AI Receptionist ×3

Same pipeline (webhook → parse → calendar booking → log → owner alert) built three ways.
Tested with identical simulated calls: bookable / pricing question / past-slot request.

## Make.com

**Pros**
- Fastest visual debugging: scenario history shows every module's in/out
- Native Vapi + Google Calendar + Telegram apps = fewest custom mappings
- Data Store gives zero-setup persistence (no external DB)
- Instant webhooks; scenario auto-pause on repeated errors protects quota

**Cons**
- IML function gaps bite hard: no `and()`, no `or()`, no `empty()` — logic gets nested-`if` spaghetti
- Filters silently drop non-matching routes (1-op "success" executions) — easy to misdiagnose
- Free plan: 1,000 ops/mo and ONE data store; scenario count caps out fast
- Error in one run auto-deactivates the whole scenario (surprise downtime)
- Router filters re-evaluate full expressions per route — duplicate logic

## n8n (self-hosted)

**Pros**
- **Free ops forever** — runs on your own hardware; only external APIs cost
- Real Code nodes (full JS) — parsing/validation in readable code instead of expression soup
- Built-in Data Tables = free database; Switch/IF nodes far more expressive than Make filters
- Version-controllable, exportable JSON; no vendor op-quota anxiety
- Best sellability for technical clients who want self-hosting/data ownership

**Cons**
- You own uptime, updates, security (this box needed a tunnel for public webhooks)
- Google Calendar node requires OAuth2 credential per instance (Service Accounts unsupported)
- No auto-retry/replay UI like Make's; failed runs need manual rerun
- Steeper learning curve for non-devs — hardest to hand to a non-technical client

## Zapier

**Pros**
- Easiest handoff: clients can read/edit Zaps without training
- 7,000+ app library; Sheets/Gmail/Calendar connections are bulletproof
- Config-as-Spreadsheet pattern is genuinely client-friendly (non-tech owners edit a tab)
- Best for conservative SMB buyers who already pay for Zapier

**Cons**
- Cannot be built/edited via API — manual click-ops only (worst for agencies scaling)
- Webhooks by Zapier is premium-gated on lower plans; task limits burn fast on webhooks
- Weakest logic layer: Formatter/Paths are clunky vs n8n Code or even Make IML
- No native Telegram; alerts forced through Gmail/SMS workarounds
- Per-task pricing makes chatty call volume expensive at scale

## Scorecard (for this use case)

| Criteria | Make | n8n | Zapier |
|---|---|---|---|
| Setup speed | ★★★★★ | ★★★☆ | ★★★★ |
| Logic flexibility | ★★★ | ★★★★★ | ★★ |
| Debugging UX | ★★★★ | ★★★ | ★★★ |
| Cost @ 100 calls/mo | $9+ ops | ~$0 | ~$20+ |
| Sellable to SMBs | ★★★★ | ★★★ | ★★★★★ |
| Sellable to tech cos | ★★★ | ★★★★★ | ★★ |

**Pitch line:** same receptionist, three engines — pick by who maintains it:
client-managed → Zapier · agency-managed → Make · self-hosted/privacy → n8n.
