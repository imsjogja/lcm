# HTTP API, authorization and Laravel Reverb events

## API strategy
Inertia controllers serve first-party UI. Use `/api/v1` for mobile/device or external clients, with Sanctum abilities and tenant-aware policies. Explicit versioning and DTO validation. Success responses carry data and correlation IDs, failure responses use consistent problem codes and actionable messages.

## Candidate endpoints (design, not implemented)
| Method | Path | Permission / effect |
|---|---|---|
| GET | `/api/v1/me/assignments` | host own upcoming schedules |
| POST | `/api/v1/shifts/check-in` | host check-in; server timestamp, idempotency key |
| POST | `/api/v1/shifts/{id}/check-out` | host own shift; audit |
| POST | `/api/v1/shifts/{id}/handover` | supervisor / current host transition policy |
| GET | `/api/v1/live-sessions` | manager/analyst, tenant-filtered |
| GET | `/api/v1/live-sessions/{id}/minutes` | manager authorized |
| GET | `/api/v1/performance/hosts` | date range, metric definition, provenance |
| GET | `/api/v1/reconciliation/exceptions` | supervisor |
| POST | `/api/v1/reconciliation/{id}/resolve` | supervisor, reason and version check |
| POST | `/api/v1/reviews/{id}/finalize` | manager, immutable ledger entry |
| GET | `/api/v1/integrations/{provider}/authorize` | owner, redirects via OAuth with CSRF state |
| GET | `/integrations/{provider}/callback` | validated OAuth callback |
| POST | `/webhooks/{provider}` | verify signature + timestamp + dedupe; never trust blindly |

Example check-in request:
```json
{"store_id":"01J...","schedule_id":"01J...","idempotency_key":"shift-uuid"}
```
Example response:
```json
{"data":{"shift_id":"01J...","checkin_at":"2026-11-22T06:00:00Z","status":"active"},"request_id":"req_..."}
```

## State transitions
- Shift: `scheduled -> active -> completed -> reviewed`; corrections become separate audited revisions.
- Sync: `queued -> syncing -> completed | failed | partial`.
- Attribution: `pending -> assigned | ambiguous | unassigned -> resolved -> approved`.
- Review: `draft -> in_review -> final`; reopen is privileged and audited.

## Reverb channels and event contracts
| Private channel | Event | Payload (minimal) |
|---|---|---|
| `organization.{tenantId}.operations` | `shift.started` | shift_id, host_id, timestamp |
| `organization.{tenantId}.operations` | `shift.finished` | shift_id, timestamp |
| `organization.{tenantId}.live.{liveSessionId}` | `analytics.synced` | live_session_id, sync_run_id, freshness |
| `organization.{tenantId}.operations` | `reconciliation.updated` | exception_id, state |
| `organization.{tenantId}.host.{hostId}` | `review.finalized` | review_id, period |

Authorize each channel using tenant membership and resource policy; never broadcast seller tokens, PII, entire order payloads or financial details if recipient not permitted. Use `ShouldBroadcast` after DB commit, ensure safe queue retries. Browser automatically reconnects but must refetch current state from HTTP after reconnect.

## Provider boundary
```php
interface LiveAnalyticsProvider {
    public function listFinishedSessions(StoreConnection $connection, DateRange $range): iterable;
    public function getMinuteMetrics(StoreConnection $connection, string $externalLiveId): iterable;
    public function capabilities(StoreConnection $connection): array;
}
```
A provider adapter returns strongly validated normalized DTOs and reports unsupported capabilities explicitly. Do not fabricate metrics by substituting unrelated fields.

## Security
- Rate-limit auth and writes, validate idempotency keys.
- Server-generated timestamps with UTC storage.
- OAuth state nonce and redirect allowlist, tokens encrypted at rest.
- Ensure replay-protected webhooks where providers offer signatures; provider-specific code must be validated against official docs.
- Maintain audit log for edits, approval and fee calculation version.
