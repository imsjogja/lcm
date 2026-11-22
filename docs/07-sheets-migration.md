# Migration from Patra livestream spreadsheet (pilot tenant)

Source (user-provided): https://docs.google.com/spreadsheets/d/1gfFFFambDty2rlrRWwIm51XMDJGYLoSQ9Zks0ZlN6hM/edit

## Observed tabs and mapping
| Existing tab | Main content | Target module | Input source |
|---|---|---|---|
| Absen Tim Live | host/shift attendance, duration, monthly fee, total printed sheets | Workforce + Fee | check-in/out + verified sessions + policy |
| Laporan Live | session time, host, viewers, comments, chats, sales, perolehan, 350 target, GMV, impressions, followers, notes | Live analytics + KPI + Feedback | provider metrics + shift attribution + exception-only manual input |
| Perolehan | per-host daily sheets, deficit/over-target carryforward | KPI targets + business unit ledger | derived once from verified order-line quantities |
| feedback | challenges, ideas, follow-up guidance, keyword research | SOP + feedback / CRM leads | optional host feedback, manager notes |
| Sheet5 | program, themes and live content strategy | SOP / content playbooks | configurable templates |

## Critical issues established by audit
- Repeated durations and perolehan between reports/attendance.
- Absensi linked to specific 'Laporan Live' cells; inserting/moving rows creates fragile references.
- Example `Laporan Live!C708` reads `16.30-21.35` (305 min), but `D708 =245/60` (245 min): 60-minute discrepancy; import must flag mismatches, not trust duration blindly.
- January `Absen Tim Live!AI4 = CEILING(AH4)` then `AK4 = 5000 * AI4`: sample suggests monthly total duration rounded up and Rp5,000/hour, **subject to business confirmation**.
- Same host may be written with different capitalization/trailing spaces. Map to canonical host IDs.
- Target 350 printed sheets is not equal to created SKU orders or platform items sold.
- Blank values, explicit zero, `-`, holiday and absent entries must remain distinguishable.

## Migration approach
1. Export a read-only historical snapshot and preserve original sheet, tab, row/column and import date.
2. Detect section/month headings and row-level data. Resolve merged date cells by explicit fill-down within section; never extrapolate outside section.
3. Normalize host names with manual approval of aliases.
4. Parse time strings carefully (dot separator, midnight rollover, local timezone). Compare independent duration calculation with formula-derived durations and flag disagreements > 1 minute.
5. Import 'Laporan Live' as historical `legacy_shift_records` with `source=spreadsheet`, **not TikTok-verified**.
6. Use 'Absen Tim Live' and 'Perolehan' only for cross-checking, not additional duplicated source rows.
7. Keep historical 'lembar' as legacy/manual measured quantities with provenance; distinguish from verified SKU/order-based units.
8. Reconcile on selected months before enabling fee calculations. Supervisor signs migration report.
9. Run in parallel for one payroll period; freeze old workbook only after sign-off.

## Acceptance criteria
- No silent loss of notes, leave days or missing values.
- Each migrated record includes source citation/row origin and confidence.
- Two independent calculations expose discrepancies rather than automatically overwriting historical records.
- Shift count, total duration, gross GMV, manual lembar and fee all have separate reconciliation summaries.
- Never present spreadsheet totals as API-verified marketplace data.
