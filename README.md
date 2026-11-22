# LCM — Live Commerce Management

**Blueprint-first, multi-tenant Live Commerce Operations & Analytics SaaS.**

LCM is designed for merchants and agencies managing rotating livestream hosts across one or more connected commerce channels. The first connector target is TikTok Shop; the architecture remains platform-agnostic so future marketplace connectors do not require rewriting workforce management or KPI logic.

> **Status (2026-11-22):** Architecture and product blueprint only. No Laravel application or TikTok connector has been implemented, deployed, or integration-tested yet.

## Product goals

- Manage hosts, shifts, SOPs, live sessions, and performance reviews with minimal manual entry.
- Measure **effective livestream time per host** against platform sessions.
- Attribute **direct live sales per minute** to the host on duty, with explicit ambiguous/unassigned buckets.
- Keep **created SKU orders, direct items sold, live-attributed items, GMV, and order-item quantities** as distinct measures.
- Support configurable product units (e.g. ticket, piece, box, invitation sheet) without inventing package sizes.
- Provide isolated tenant dashboards, audit trails, fee calculations, and exports.
- Onboard sellers through authorized OAuth connections and platform-specific capabilities.

## Recommended stack

| Area | Decision |
| --- | --- |
| Backend | **Laravel 13**, PHP 8.4+, modular monolith |
| Frontend | Vue 3 + TypeScript + Inertia.js + Vite |
| UI | Bootstrap 5.3 and design tokens; mobile-first shift workflows |
| Realtime | **Laravel Reverb** + Laravel Echo (application events, not a TikTok live feed) |
| Database | PostgreSQL 17+ (prefer a production-supported stable major) |
| Cache and jobs | Redis + Laravel Horizon + Scheduler |
| Auth | Laravel authentication, Sanctum for scoped API tokens; tenant-aware policies |
| Observability | Structured JSON logs + request/job IDs, Telescope (local only), Pulse/Horizon, optional Sentry |
| Testing | Pest / PHPUnit, Laravel HTTP fakes, Vue Testing Library + Vitest, Playwright |
| Deployment | Docker Compose for first production stage; CI via GitHub Actions; S3-compatible object storage |

Architecture principle: **one deployable modular monolith, tenant-scoped relational data, provider adapters at the integration boundary, and asynchronous external sync**. Do not split into microservices before operational evidence justifies it.

## Documentation

1. [Product scope and roles](docs/01-product-blueprint.md)
2. [Architecture and technology decisions](docs/02-architecture-and-stack.md)
3. [Data model and invariants](docs/03-data-model.md)
4. [Application API and Reverb events](docs/04-api-and-events.md)
5. [Sales attribution and reconciliation](docs/05-attribution-and-reconciliation.md)
6. [Delivery roadmap, testing and operations](docs/06-delivery-plan-and-ops.md)
7. [Migration from the existing live-team spreadsheet](docs/07-sheets-migration.md)
8. [UX flows and screen inventory](docs/08-ux-blueprint.md)
9. [Architecture decisions](docs/adr/0001-modular-monolith.md)
10. [Metric semantics](docs/adr/0002-sales-metric-semantics.md)
11. [Contribution and debugging guide](CONTRIBUTING.md)

## Non-negotiable product truths

- **A marketplace account is not a person.** A live session may have multiple host shifts.
- **Reverb is realtime for LCM's own state**, such as a host checking in or a sync completing; historic TikTok analytics are not automatically realtime.
- A minute's aggregated sales is attributed only to one *unambiguously assigned* host interval. Boundary collisions are flagged for review; no double counting.
- A marketplace's `items_sold` is **not automatically** an offline/local business unit count such as invitation sheets. Package conversion needs verified SKU/order-item evidence.
- TikTok API fields, approved scopes, supported markets and attribution rules are subject to validation with a real authorized seller account.
- Tenant boundaries are enforced in every query, API and websocket channel. No production secrets, PII exports or seller tokens in the repository.

## Development entry criteria

Start implementation only after confirming the TikTok analytics scope for the partner app and testing one finished session with minute-level metrics. Workforce scheduling/check-in can proceed independently of external API approval.

## References

- [Laravel 13 release notes](https://laravel.com/docs/13.x/releases)
- [Laravel Reverb](https://laravel.com/docs/13.x/reverb)
- [Laravel Broadcasting](https://laravel.com/docs/13.x/broadcasting)
- [TikTok Shop Partner developer docs](https://partner.tiktokshop.com/docv2/page/tts-developer-guide)
- [TikTok LIVE Analytics changelog](https://partner.tiktokshop.com/docv2/page/6az7iwe9)

© 2026 LCM project. Architecture proposals are subject to implementation and integration validation.
