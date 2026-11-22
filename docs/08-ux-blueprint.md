# UX blueprint and screens

## Information architecture
- Workspace selector (tenant) and store selector; persisted per-user server-side preferences.
- **Overview**: today active hosts, scheduled/working/live duration, direct sales, pending sync, exceptions.
- **Live operations**: timeline by store and session, live room status, handover markers and staffing gaps.
- **Shifts**: calendar, roster, attendance, leave, corrections and approval.
- **Performance**: host comparisons, product/service unit selector, metric definitions, daily/monthly filters.
- **Sales & products**: authorized order reports, SKU-to-business-unit mapping, unknown SKU queue.
- **Reconciliation**: provider revision, ambiguous minutes, unassigned sessions, unresolved mappings, manual adjustments.
- **SOP & feedback**: checklist templates, content playbooks and incident notes.
- **Settings**: team roles, integration authorization/health, targets, fee policies and audit logs.

## Essential screens
### Host mobile
1. Upcoming shift card: store, scheduled local time, checklist.
2. **Start shift** button (server time); confirmed state and overlap warnings.
3. Current active shift: elapsed *working* time, actual LIVE match status, handover action.
4. **Finish shift** button; note/feedback optional.
5. My performance: provisional/approved tags and source freshness, do not imply realtime TikTok sales.

### Manager desktop
1. Operational timeline with each host shift overlaid on actual live session.
2. Reconciliation inbox before report finalization.
3. KPI chart by metric semantics: direct items, created SKU lines, business units (verified only), GMV and live duration.
4. Payroll/fee preview with rounding scope and effective-dated policy.
5. Monthly export and approval audit.

## UI behavior
- Responsive Bootstrap 5.3, semantic forms, keyboard navigation, accessible contrast, toast notifications and clear error recovery.
- Always include timezone and currency labels.
- Show an explicit `unknown` state rather than display zero when data is absent.
- Real-time Reverb updates change **LCM local workflow**, not TikTok metrics that aren't delivered in real-time.
- Use route-driven pages through Inertia and dedicated components/composables; avoid adding an extra SPA API layer in MVP.
- Design empty screens and failure/retry paths as first-class states.
