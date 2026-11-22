# Product blueprint — LCM

## Vision and tenancy
LCM is an independent SaaS for live-commerce teams. One **organization (tenant)** owns users, teams, sales channels, stores, hosts, plans, subscriptions and business policies. A tenant can manage many stores, multiple channels, multiple live sessions and rotating hosts. A host can be assigned across stores in one tenant. Never permit cross-tenant access.

## Personas
- **Platform admin:** SaaS operations, tenant onboarding, integration health, billing controls. No implicit access to seller PII.
- **Tenant owner:** organization configuration, store OAuth, plans and permissions.
- **Manager:** schedules, targets, policy versions and final approvals.
- **Supervisor:** handovers, corrections, incident resolution and shift verification.
- **Host:** own schedule, check-in/out, feedback and own verified KPI.
- **Analyst:** read-only reports and exports, appropriately scoped.

## MVP functional requirements
1. Organization creation, membership invitation, role-aware permissions.
2. Seller-store authorization and credential health, with one TikTok Shop adapter.
3. Host directory, scheduled shifts, attendance, absence/leave, and check-in/out with authoritative server time.
4. Actual LIVE session import and linking to each shift. A shift can remain pending linkage until session data arrives.
5. Minute-level sales ingestion; normalized KPI snapshots and reconciliation queues.
6. Unit mappings for products/SKUs, configurable daily target, target deficit/surplus, live fee rule and historical policy effective dates.
7. Supervisor corrections and approvals with immutable audit changes.
8. Feedback after each shift; SOP and preflight checklist.
9. Responsive host workspace, manager dashboard, export CSV/XLSX later.

## Phase 2
Multi-store cross-channel analytics, client CRM and lead tracking, product bundles, alerts, incentives, multiple languages/currencies, subscription billing, advanced reporting. Integration support depends on provider authorization and real API validation.

## Operational UX principles
- Host has one primary action: **Start shift** or **Finish shift**; optionally report incident or lead.
- No repeated entry of host name, duration, GMV or unit counts.
- Manager sets schedules and target policy once; platform derives daily/monthly results.
- Show `pending_sync`, `provisional`, `needs_review`, `approved`, `final` rather than concealing uncertainty.
- Every KPI displays source, freshness timestamp, and metric definition.
- Separate `scheduled_minutes`, `worked_minutes`, and `effective_live_minutes`.
- The organization may define business units such as `sheet`, `piece`, `package`, or `unit`; these must not be presumed equivalent.

## Quality attributes
- Per-tenant isolation and authorization tests for every query endpoint/event.
- Backend idempotency for OAuth callbacks, webhooks, imports and manual actions.
- Accessible, mobile-friendly UI, server-side validation and good empty/error states.
- Debuggable jobs with correlation IDs and deterministic replay; high availability is evolutionary, not MVP theater.
- Metrics can be recomputed under a **versioned attribution algorithm**.
- Privacy-by-default; least-privilege scopes and short retention for sensitive customer data.

## Out of scope for MVP
Actual livestream video broadcasting, streaming RTMP ingest, creator identity recognition, automatically scraping live-chat messages, claiming platform-level realtime sales, fully automated payroll disbursement and AI-based attribution. These are separate products/capabilities and should not be silently implied by an analytics API.
