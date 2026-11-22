# Sales attribution and reconciliation policy

## Terminology: do not conflate metrics
1. `created_sku_orders`: created SKU-order lines; **not** quantities of product and may include unpaid orders depending on platform semantics.
2. `direct_items_sold` (`sales.items_sold`): minute-bucket direct sold items, when available.
3. `live_attributed_items_sold`: provider-attributed items at the whole live-session level, possibly different attribution window.
4. `direct_gmv`: direct GMV as provided; not necessarily the same as paid/refunded net revenue.
5. `business_units`: e.g. printed invitation sheets, derived from verified order SKU quantities and `units_per_sku`, **not from created_sku_orders**.
6. `leads`/DM chats: own CRM metric, unrelated to purchase counts.
7. `worked_duration` versus `effective_live_duration`: duration at work versus overlap with actual on-air intervals.

## Effective live time
For each checked-in shift [a,b) and verified platform live interval [c,d), effective duration in seconds is:
```
max(0, min(b,d) - max(a,c))
```
Sum disjoint verified overlaps only. A host checked in early has non-live working minutes; a host with missing check-out remains provisional, not automatically credited indefinitely.

## Minute ownership
Normalize provider minute bucket as half-open [m,m+60s); ensure bucket timezone and endpoint interpretation are validated.
- Exactly one host shift **fully covers** the bucket, intersected with the corresponding LIVE session: `assigned`.
- No host covers/intersects it: `unassigned`.
- Some coverage, partial handover, or multiple overlapping hosts: `ambiguous`.
- Do **not** prorate quantities over the minute, because orders could have occurred at any instant.
- A source minute may have one attribution assignment per algorithm version; persist provenance and keep revisions for audits.

```text
for each live minute metric:
    candidates = active linked shifts that intersect [minute_start, minute_end)
    if count(candidates) == 1 and candidate fully covers interval:
        assign direct_items_sold, created_sku_orders, direct_gmv to candidate
    else if count(candidates) == 0:
        mark unassigned
    else:
        mark ambiguous
```

If LIVE starts/ends partway through a minute, do not assert full-bucket coverage or exact event time. Treat uncovered boundary intervals as `ambiguous` unless the provider defines timestamp semantics sufficient to resolve.

## Conservation and data quality
For each live session:
```
total_direct_items = assigned_items + ambiguous_items + unassigned_items
```
Check source `overall.items_sold` against sum of minute metrics, accounting for pagination, time-bucket definitions, refunds, data delay and rounding. Record discrepancies, never auto-hide them. A single 6-hour live requires at least 4 pages if provider limit is 100 minutes/page.

## Marketplace orders and business units
- Normalize order lines only when OAuth grants relevant order scopes.
- Associate order lines to live/host **only with verified evidence** (e.g. provider live ID or documented join key), not from host name or near-time guesses.
- Multiply known fulfilled/paid SKU quantities by effective-dated unit mappings. Example: 2 sold SKUs * 50 invitation sheets/SKU = 100 sheets.
- `unknown` mapping must appear as unmapped quantity, not zero; manual adjustments require evidence and approval.
- Consider return, refund and cancellation separate adjustment records, rather than retrospectively rewriting history.
- For markets where per-order live-room identifiers are unavailable, do not claim exact per-host business units. Provide direct items sold per shift and separate business-unit provisional/manual reconciliation instead.

## Formula/version policy
`performance_per_hour = assigned_direct_items / (effective_live_seconds / 3600)` when denominator > 0.
`daily_target_variance = verified_business_units - configured_target` only on eligible shifts; carry-forward deficit is a distinct ledger policy.
Fee computation is per effective-dated policy with explicit rounding scope: e.g. **monthly total hours rounded up**, not per-shift rounding unless configured. Never use payroll from provisional shifts.
Store `rule_version`, input snapshot checksum, reviewer, timestamp and data-source freshness for finalization.

## Acceptance tests
1. One 6-hour session and three 2-hour hosts produce no double counting.
2. Handover at 10:00:30 marks 10:00 bucket ambiguous.
3. Overlapping check-ins cannot silently duplicate units.
4. Unassigned minute remains visible.
5. Early check-in does not count as live duration.
6. Order lines with 50/100/250 sheets per SKU convert correctly only when verified.
7. Created SKU order count never converted into sheets.
8. Provider revision after approval triggers a discrepancy/new adjustment workflow, not silent rewriting.
9. Mixed timezone and DST converted to UTC consistently.
10. Tenant B cannot read tenant A's store, shift, metric or Reverb channel.
