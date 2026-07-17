---

## ADR-003 — Centralised Non-Directional Relative Volume Engine

**Date:** 2026-07-17

**Status:** Proposed

### Context

Several planned modules require relative-volume information, including VPA, Wyckoff, Liquidity, Institutional Activity, Dashboard, Alerts, and Signal Scoring.

Allowing each module to calculate its own volume baseline would create duplicated calculations, inconsistent thresholds, and conflicting interpretations.

Relative volume also measures participation intensity rather than directional intent.

### Decision

The project shall use one central Relative Volume Engine.

The engine shall:

- Calculate one shared baseline.
- Produce one shared RVOL value.
- Classify participation intensity.
- Distinguish unavailable volume from genuinely low volume.
- Remain non-directional.
- Expose reusable outputs to all dependent modules.

Directional interpretation shall be performed by contextual modules such as VPA, Wyckoff, Liquidity, and Institutional Activity.

### Consequences

**Positive:**

- Consistent volume measurements across all modules
- Reduced duplicate calculations
- Lower Pine Script execution cost
- Clear separation between measurement and interpretation
- Easier testing and threshold validation

**Negative:**

- All dependent modules rely on one shared baseline policy
- Baseline changes may affect several modules simultaneously
- Session-aware behaviour requires careful future design

### Approval

Pending specification review.