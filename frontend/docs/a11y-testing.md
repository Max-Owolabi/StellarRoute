# Accessibility testing baseline

Reference for the automated axe scans in `frontend/e2e/a11y-swap-flow.spec.ts` (issue #312).
This document defines the axe configuration, the severity threshold, and the baseline
exclusions so that every axe rule that is disabled in CI is explicitly justified and reviewed.

## How the scans work

- Framework: Playwright + `@axe-core/playwright` (see `frontend/playwright.config.ts`, project `a11y`).
- Each test scopes the scan to a DOM subtree with `.include(selector)` and disables the
  baseline exclusions with `.disableRules(BASELINE_EXCLUSIONS)`.
- Only violations with impact `critical` or `serious` fail a test. `moderate` and `minor`
  violations are included in the failure report for visibility but do not block CI.

## Axe rules in use

The scans run the full default axe-core rule set shipped with `@axe-core/playwright`
(tags: `wcag2a`, `wcag2aa`, `wcag21a`, `wcag21aa`, `best-practice`). No rule subset is
configured; the only rules removed from the run are the baseline exclusions below.

Surfaces covered by the suite:

| Surface                   | Selector scope                          | Scenarios                                                               |
| ------------------------- | --------------------------------------- | ----------------------------------------------------------------------- |
| Swap form                 | `[data-testid="swap-card"]`             | default, amount entered, insufficient balance, stale quote, quote error |
| Token dialog              | `[role="dialog"]`                       | scan, focus on open, Escape, focus trap                                 |
| Route list                | `[data-testid="route-display"]`         | scan, accessible names, keyboard selection                              |
| High-impact confirm modal | `[role="dialog"]`                       | scan, focus on open, Escape, focus trap                                 |
| Settings panel            | `[data-testid="settings-panel"]`        | scan, slippage input label                                              |
| Cross-chain deck          | `[data-testid="cross-chain-swap-deck"]` | scan, corridor alert, keyboard selection                                |

## Baseline exclusions

Axe rule IDs that are explicitly deferred. Every entry must stay in sync with the
`BASELINE_EXCLUSIONS` array in `frontend/e2e/a11y-swap-flow.spec.ts`.

| Rule ID          | Rationale                                                                                                                                                                                                                                                           | Tracking                                                                           |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `color-contrast` | The indigo primary palette (`#6366f1`) fails WCAG 2.1 AA (1.4.3) against the dark-mode surfaces `#0a0e1a` (page background) and `#141c2b` (card background). Fixing requires a design-level colour-token change tracked separately and must not block unrelated CI. | Design token remediation — see `../../docs/design/accessibility-contrast-audit.md` |

## Adding or removing an exclusion

1. Edit `BASELINE_EXCLUSIONS` in `frontend/e2e/a11y-swap-flow.spec.ts`.
2. Keep the comment in the spec and the table above in sync — each exclusion needs a rationale.
3. When a rule is fixed, remove it from both places so axe starts enforcing it again.

## Running the suite

```bash
npm --prefix frontend run test:e2e -- a11y-swap-flow --project=a11y
```
