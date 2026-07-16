# Budget and Hosting: Three Implementation Options

All figures below are **rough estimates**, in USD, for one café with a single location. Actual cost depends on the contractor's region, the complexity of the client's website, and how many additional order channels are planned down the line.

## Comparing the options

| | Google Apps Script (native) | No-code (n8n / Make) | Custom service (Node/Python + Sheets API) |
|---|---|---|---|
| **Where it runs** | Inside Google (time-driven trigger), no hosting needed | Cloud subscription platform, or self-hosted n8n on your own server | Separate server/PaaS (Render, Railway, Fly.io, etc.) |
| **Infrastructure / mo** | $0 | n8n Cloud ≈ $20–50, or self-host ≈ $5–10 (VPS) | ≈ $5–25 (small instance) |
| **Development (one-time)** | $300–600 | $400–800 | $800–1500 |
| **Support / mo** | $0–50 | $50–100 | $100–200 |
| **Time to launch** | 3–5 days | 5–7 days | 10–14 days |
| **Pros** | Free, everything lives inside Google, minimal moving parts | Visual builder, dozens of ready-made integrations (WhatsApp, CRM, Telegram) | Full control over logic, easier to test, not bound by Google's/the platform's limits |
| **Cons** | Logic changes require basic Apps Script (JavaScript) skills | Monthly subscription; self-hosting needs server administration | Needs a developer for any change; separate hosting and monitoring |
| **Best fit** | One café, a simple scenario — like this case | A café/chain planning to add more channels (messengers, CRM) | A café chain, custom analytics/dashboards needed, higher load |

## Recommendation for this case

For a single café with three order channels and a simple 30-minute sync — **Google Apps Script**:

- zero infrastructure cost;
- everything lives right where the data already does (Google Sheets), so staff don't need to learn a new interface;
- as the business grows (a second location, delivery integration, a WhatsApp bot), the logic migrates cleanly to a no-code platform or a custom service, because the business rules (the agent prompts) are already documented separately from the implementation.

## What development involves (for the Apps Script option)

| Stage | What's done | Estimate |
|---|---|---|
| Order-intake integrations | A form for walk-ins/phone (Google Form or a simple web form) + a website widget that writes to the same sheet via a Web App endpoint | 1–2 days |
| Agent 1 script | Order intake, `new` status, daily/monthly tab and spreadsheet rotation | 1 day |
| Agent 2 script | Time-driven trigger every 30 minutes, sum/status logic, sorting | 1–2 days |
| Testing on real café data | Running a typical day's orders through it, checking edge cases (first order of the day, month rollover, "stuck" done items) | 0.5–1 day |
| Staff training | A 30-minute call/walkthrough: how to set the status in the kitchen | — |

## What's not included in the base cost (optional, separately scoped)

- Payment gateway/online-payment integration on the website.
- Push/SMS notifications to the customer when the order is ready.
- A full analytics dashboard (sales trends by dish, hour, day of week) on top of the same data — technically straightforward, since the data is already structured, but it's a separate module.
