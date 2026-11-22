# ADR-0002: Separate platform metrics from business units

**Status:** Accepted for blueprint (2026-11-22)

## Context
Patra spreadsheet tracks printed invitation sheets, while TikTok provides SKU-line orders, sold items and per-live attribution. Other future tenants sell unrelated products/services.

## Decision
Store platform metrics separately from tenant-configured business-unit metrics. The metric catalog defines dimensions, source, time grain, payment state, definition version, provider support and confidence. Per-host minute attribution applies only to minute-resolution direct sales where ownership is unambiguous. Order-to-host linkage requires documented provider evidence.

## Consequences
- No arbitrary multiplication of orders by presumed packages.
- Unknown mapping is `unmapped`, not `0`.
- Reports distinguish live session attribution, direct minute sales, order lines, GMV, verified business units, adjustments and final approved KPI.
- Discrepancies do not mutate historical finalized results without a tracked revision.
