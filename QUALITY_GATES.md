# Quality Contract

An AI agent must add appropriate tests for new functionality and report the actual results of every check. Never weaken tests, thresholds or CI to force a passing result. Human review is required for security and policy changes.

**Automated today:** `git diff --check` and nonempty README and conflict-marker detection.

This lightweight repository does not yet have a numerical coverage or performance gate; do not assert such guarantees. Require the `quality` status in the branch ruleset to block violating PRs.
