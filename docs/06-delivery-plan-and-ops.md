# Delivery plan, CI/CD, testing and operations

## Execution stages and exit gates
**0 — Connector feasibility.** Authorize a real test store and establish approved TikTok Analytics scopes. Confirm historical live listing, `live_id`, minute-level `sales.items_sold` and `conversion.created_sku_orders`, 100-row pagination and available order data in the intended market. Record actual provider error/rate-limit contracts. Gate: fixture files with consent and redacted data, documented field semantics.

**1 — Foundation.** Bootstrap Laravel + Inertia/Vue/TS/Bootstrap, PostgreSQL, Redis, auth, org memberships, permission policies, Reverb+Echo, testing and CI. Gate: local dev from README, tenant isolation tests, Reverb auth checks.

**2 — Workforce.** Schedule/host CRUD, check-in/out, supervisor handover and audit, effective duration engine. Gate: actual human pilot succeeds without external API; out-of-range/duplicate shifts rejected.

**3 — Analytics.** OAuth and TikTok adapter, idempotent queued sync, pagination, per-minute normalization, reconciling states. Gate: actual 6-hour sample matches source within documented API semantics.

**4 — Business KPIs.** Product SKU/unit mappings, distinct KPI variants, target policies, fee policies, adjustments, exception queue and approval. Gate: expected historical spreadsheet cases pass golden-file tests.

**5 — Launch.** Dashboard, reports, logs/alerts, backup restore drill, security verification and staged tenant onboarding. Gate: no unassigned anomalies hidden, tested rollback.

Phase 2: CRM, cross-channel adapters, analytics warehouse when justified, subscription plans/billing.

## Suggested monorepo setup
```
.
├── app/
├── bootstrap/
├── config/
├── database/
├── resources/js/
├── resources/scss/
├── routes/
├── tests/
├── docs/
├── docker/
├── .github/workflows/
├── composer.json
├── package.json
└── README.md
```

## Local debugging workflow
- Prefer Docker Compose services for app, PHP worker, scheduler, Postgres, Redis and Reverb; use deterministic env example and seeded synthetic tenants.
- Local `php artisan serve`, `php artisan queue:work`, `php artisan reverb:start --debug`, `npm run dev`; Horizon for queued jobs.
- Enable Telescope locally, not in production public routes. Use query logging/EXPLAIN for slow queries.
- Provider request logs: correlation ID, status, latency, API path, pagination cursor, tenant/store IDs; **never** raw OAuth credentials or customer PII.
- Each import includes idempotency key, retry count, backoff, dead-letter/review state and replay commands.
- Build provider contract tests using sanitized fixtures plus a small opt-in integration suite; CI must not require live seller tokens.

## CI targets
1. Composer validation, `php -l`, Pint format, Larastan/PHPStan static analysis.
2. Pest unit/feature integration tests on PostgreSQL + Redis.
3. ESLint, TypeScript `vue-tsc --noEmit`, Vitest, Vite build.
4. Playwright smoke tests against bootstrapped seeded application (CI stage).
5. Dependency audit, secret scan and GitHub branch protection.

## Security and operations
- HTTPS everywhere, secure cookies, CSRF on session forms, role/policy checks on every write and read.
- No committed `.env`, seller token, database backup or unredacted order exports.
- Strict tenant filtering in API and Reverb channel authorization; verify membership for every subscription.
- Encrypt provider credentials at rest; rotate credentials and revocation workflow.
- Queue isolation (sync, broadcasts, default), per-provider rate-limit and circuit-breaker handling.
- Back up PostgreSQL daily, object storage as applicable, test restoration, configure RPO/RTO from service requirements.
- Monitor errors, API sync freshness, job latency, failed jobs, 95p HTTP latency, Reverb active connections and reconciliation backlog.
- Start on one modest VM with Docker Compose and reverse proxy; split app/websocket/workers only when load metrics justify.
- Data retention, access logging, data subject deletion and compliance for target market need policy review.

## Evidence-based claims
Don't claim exact real-time marketplace data unless provider delivers such events. Track sync latency explicitly. Full production capacity / performance is unknown until load testing and quota discovery.
