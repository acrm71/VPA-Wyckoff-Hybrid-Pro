# Phase 1 Foundation Regression Record

## Test Information

**Implementation Phase:** Phase 1 — Shared Foundation

**Script Version:** 0.2.0-dev

**Pine Version:** 6

**Test Status:** Passed

---

## Compilation Test

- [x] Script compiles without errors.
- [x] Script adds to chart.
- [x] No unexpected visible plots appear.
- [x] No labels, lines, boxes or tables are created.
- [x] Input groups appear in the expected order.
- [ ] Confirmed Bar Only is the default.
- [ ] Balanced is the default profile.
- [ ] Debug Mode is disabled by default.

---

## Framework Tests

- [ ] Shared ATR is available after sufficient history.
- [ ] Minimum-history status changes at the configured bar.
- [ ] Symbol metadata is available.
- [ ] Timeframe metadata is available.
- [ ] Tick size is available.
- [ ] Volume availability does not cause a runtime error.
- [ ] Disabled indicator state is safe.
- [ ] BOS dependency fails safely when Structure is disabled.
- [ ] Liquidity dependency fails safely when Structure or BOS is disabled.

---

## Historical Stability

- [x] No confirmed historical event is created by the foundation.
- [x] Reloading the script produces the same framework values.
- [ ] Additional future bars do not alter prior framework context.

---

## Test Notes

Record TradingView compiler messages and any deviations here.
