# Agent instructions — nodus

## Repository-specific deterministic quality contract (2026-10-26)

**Project:** Minimal documentation-only repository.

**Engineering requirements:** Preserve README semantics and do not claim unit coverage where no source exists.

**Verification commands:** test -s README.md; git diff --check.

Run relevant tests and give actual outputs, including failures. Add regression coverage for behavior, edges and error paths. Never weaken/delete/skip tests, suppress errors or reduce thresholds solely to get CI green. Changes to this file, `QUALITY_GATES.md`, workflows and test policy require human review. Make the `quality` check required on protected branches. Disclose missing system dependencies. Preserve existing instructions.
