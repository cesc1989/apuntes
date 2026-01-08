# Delay en la creación de POCs o Fallan y luego hay retry

Etiquetas: #luna_help_desk 

Document principal: [[026 - Missing POCs 🟡]]

## Contexto Inicial

**Critical Finding**: 2-Day Delay Confirmed!

**Timeline**:
- **Chart signed**: 2026-01-06 18:29:11
- **POC created**: 2026-01-08 19:34:07
- **Delay**: ~2 days (48 hours)

This proves the **original problem is real**: POCs are NOT being generated automatically when charts are signed.

CRITICAL FINDING: 2-Day Delay Confirmed!

  Timeline:
  - Chart signed: 2026-01-06 18:29:11
  - POC created: 2026-01-08 19:34:07
  - Delay: ~2 days (48 hours)

  This proves the original problem is real: POCs are NOT being generated automatically when charts are signed.

  What Happened

  1. Jan 6: Chart 1921806 was signed
  2. Jan 6: POC generation failed/skipped (no POC created)
  3. Jan 8: POC finally generated (backfill, manual trigger, or retry)

  This Explains the 825+ Missing POCs

  The issue isn't that POCs were never generated - they're being generated late through some backfill/retry mechanism, which is why:
  - Spreadsheet tracks them as "missing" initially
  - Later queries show they exist (after backfill)
  - 85% "skip rate" might include cases being processed multiple times

1. Jan 6: Chart 1921806 was signed
2. Jan 6: POC generation failed/skipped (no POC created)
3. Jan 8: POC finally generated (backfill, manual trigger, or retry)

This Explains the 825+ Missing POCs

The issue isn't that POCs were never generated - they're being generated late through some backfill/retry mechanism, which is why:
- Spreadsheet tracks them as "missing" initially
- Later queries show they exist (after backfill)
- 85% "skip rate" might include cases being processed multiple times