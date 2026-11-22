# Data model blueprint

PostgreSQL, UTC `timestamptz`, monetary `numeric(20,4)` plus explicit currency, quantities `numeric(20,4)` where units may be fractional, UUID/ULID application identifiers. Never store third-party IDs as floating point. Every tenant-owned table includes `organization_id` or an unequivocally tenant-scoped parent reference.

## Foundational tables
| Table | Key fields and purpose |
|---|---|
| `organizations` | id, slug, display_name, timezone, status |
| `users` | id, email, password, etc |
| `organization_memberships` | organization_id, user_id, role, status |
| `hosts` | organization_id, user_id nullable, display_name, active |
| `channels` | code (tiktok_shop / future providers), capabilities |
| `stores` | organization_id, channel_id, external_store_id, name, timezone, status |
| `store_connections` | store_id, scopes, encrypted access/refresh tokens, token_expiry, sync_status |
| `shift_templates` | organization_id, name, local_start_time, duration, timezone |
| `shift_schedules` | organization_id, store_id, host_id, starts_at, ends_at, status |
| `host_shifts` | organization_id, store_id, host_id, schedule_id nullable, checkin_at, checkout_at nullable, status, source |
| `live_sessions` | organization_id, store_id, provider_live_id, start_at, end_at, status, account_id, raw_version |
| `host_shift_live_links` | organization_id, host_shift_id, live_session_id, overlap_start/end, linkage_status |
| `live_minute_metrics` | organization_id, live_session_id, minute_start/end, direct_items_sold, created_sku_orders, direct_gmv, currency, import_revision |
| `live_product_metrics` | organization_id, live_session_id, provider_product_id, provider_sku_id, metric_name, metric_value |
| `orders` | organization_id, store_id, provider_order_id, placed_at, status, currency (only if authorized) |
| `order_items` | organization_id, order_id, provider_line_id, sku_id, ordered_qty, paid_qty, status, provider_live_id nullable |
| `product_unit_mappings` | organization_id, store_id, provider_sku_id, business_unit, units_per_sku, valid_from, valid_to, verified_by |
| `minute_attributions` | organization_id, live_minute_metric_id, host_shift_id nullable, state, rule_version, attributed_items_sold, attributed_gmv |
| `order_attributions` | organization_id, order_item_id, host_shift_id nullable, evidence_source, state, rule_version |
| `target_policies` | organization_id, scope, metric, value, unit, effective_from/to |
| `fee_policies` | organization_id, hourly_rate, currency, rounding_mode, rounding_scope, effective_from/to |
| `shift_reviews` | organization_id, host_shift_id, reviewer_id, status, findings, finalized_at |
| `feedback_entries` | organization_id, host_shift_id, category, note, created_by |
| `sync_runs` | organization_id, store_id, provider, type, state, request_id, cursor, attempted_at, completed_at |
| `audit_logs` | organization_id, actor_id, subject_type/id, action, before/after sanitized, occurred_at |

## Required invariants
- Unique `stores(organization_id,channel_id,external_store_id)`.
- Unique `live_sessions(store_id,provider_live_id)`.
- Unique `live_minute_metrics(live_session_id,minute_start)` for a given normalized metric revision.
- Unique active shift per host/store where business policy forbids overlapping check-ins. Enforce transactionally and, where practical, with PostgreSQL exclusion/partial constraints; a tenant may intentionally schedule but not activate overlapping assignments.
- Unique `minute_attributions(live_minute_metric_id,rule_version)`, never two assigned hosts for the same source minute.
- Every `host_shift_live_link` must be within both its shift and session interval.
- `completed_at >= started_at` and no negative time windows.
- No cascade deletion for audited financial/attribution facts; archive or tombstone entities.

## Tenant security
Use tenant-aware repository scopes and authorization policies, verify foreign-key ownership across all records, deny cross-tenant websocket subscriptions, and test IDOR attacks. Do not rely on a global scope alone. Use connection-level secrets encryption with key rotation and no raw tokens in logs.

## Data lineage and reporting
`raw_provider_metric` (external source) -> `normalized_metric` -> `minute_attribution` -> `kpi_snapshot` (optional materialization). Every report includes `metric_definition_version`, `rule_version`, `source_synced_at` and `approval_state`. Monthly totals should be derived, not manually duplicated. Denormalized snapshots are optional caches and never the source of truth.

## Partitioning policy
Do not partition prematurely. Add indexes on `(organization_id,store_id,start_at)`, `(live_session_id,minute_start)`, `(organization_id,host_id,checkin_at)`, and `(organization_id,state)` first. Consider range partitioning for minute metrics only after measurement confirms retention/volume pressure.
