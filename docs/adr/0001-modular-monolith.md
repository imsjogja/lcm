# ADR-0001: Modular monolith with Vue/Inertia and Laravel Reverb

**Status:** Accepted for blueprint (2026-11-22)

## Context
We need fast implementation and debugging, strong tenant isolation, reliable scheduling and analytics, without premature operational complexity. The backend must be Laravel + Reverb.

## Decision
Adopt one Laravel 13 modular monolith; Vue 3 + TypeScript + Inertia, Bootstrap, PostgreSQL, Redis/Horizon, and Laravel Reverb. Provider API integrations reside behind interfaces and normalized DTOs.

## Consequences
- One deployable service and one transactional database simplify debugging, correlation IDs, policy enforcement and data correction.
- Redis-backed jobs isolate provider rate limits and delayed historical analytics.
- Reverb powers internal operation updates only; it is not assumed to stream TikTok events.
- If cross-tenant scale or analytics volume grows significantly, extract workers/analytics components based on measurements, not prediction.
