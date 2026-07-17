# Market Structure Specification

---

## Document Information

**Document ID:** SPEC-104

**Version:** 0.1 Draft

**Status:** Draft

**Module:** Market Structure Engine

**Dependencies:**

- Framework
- Settings
- Trend Engine
- Dashboard
- Alerts

**Dependent Modules:**

- BOS and CHOCH
- Liquidity
- VPA
- Wyckoff
- Institutional Activity
- Signal Scoring

---

# 1. Purpose

The Market Structure Engine identifies and classifies meaningful price swings.

Its primary responsibilities are to detect:

- Swing highs
- Swing lows
- Higher highs
- Higher lows
- Lower highs
- Lower lows
- Structural direction
- Structural transitions
- Active structural reference levels

The engine shall provide confirmed structural information to all downstream modules.

It shall not independently generate trade entries.

---

# 2. Objectives

The Market Structure Engine shall:

- Detect price pivots consistently.
- Distinguish provisional and confirmed pivots.
- Classify confirmed swings as HH, HL, LH, or LL.
- Maintain an internal history of significant swings.
- Determine the current structural direction.
- Provide reference levels for BOS, CHOCH, and liquidity analysis.
- Minimise false structure changes caused by insignificant price movement.
- Operate without intentional historical repainting.
- Manage chart objects efficiently.
- Support configurable sensitivity.

---

# 3. Design Principles

## 3.1 Confirmed Structure

Confirmed structural events shall use confirmed pivots.

A pivot shall not be treated as confirmed until the required bars to its right have closed.

## 3.2 Provisional Awareness

The engine may internally track developing or provisional swings, but provisional states shall remain clearly separated from confirmed structure.

## 3.3 Structural Significance

Not every local price fluctuation shall become a structural swing.

The engine shall support filters that reduce insignificant pivots.

## 3.4 Stable Historical Output

Once a pivot is confirmed and recorded, it shall not move retrospectively under normal operation.

## 3.5 Separation of Responsibilities

The Market Structure Engine identifies and classifies swings.

The BOS and CHOCH module determines whether price has broken those structural levels.

The Liquidity module interprets sweeps and failed breaks around those levels.

## 3.6 Market Independence

The engine shall function across different asset classes and price scales.

---

# 4. Core Definitions

## 4.1 Swing High

A Swing High is a confirmed local maximum surrounded by lower highs according to the selected pivot method.

## 4.2 Swing Low

A Swing Low is a confirmed local minimum surrounded by higher lows according to the selected pivot method.

## 4.3 Higher High

A Higher High is a confirmed Swing High above the previous relevant confirmed Swing High.

## 4.4 Lower High

A Lower High is a confirmed Swing High below the previous relevant confirmed Swing High.

## 4.5 Higher Low

A Higher Low is a confirmed Swing Low above the previous relevant confirmed Swing Low.

## 4.6 Lower Low

A Lower Low is a confirmed Swing Low below the previous relevant confirmed Swing Low.

## 4.7 Equal High

An Equal High is a confirmed Swing High sufficiently close to the previous relevant Swing High according to a configurable tolerance.

## 4.8 Equal Low

An Equal Low is a confirmed Swing Low sufficiently close to the previous relevant Swing Low according to a configurable tolerance.

## 4.9 Structural Leg

A Structural Leg is the price movement between two relevant confirmed pivots.

## 4.10 Structural Direction

Structural Direction represents the current sequence of confirmed swings.

Required directions:

- Bullish
- Bearish
- Neutral
- Transition
- Unavailable

---

# 5. Inputs

## 5.1 Enable Market Structure Engine

Type:

Boolean

Default:

Enabled

When disabled:

- No new structure calculations shall be performed where avoidable.
- Structure visuals shall be hidden.
- Structure outputs shall be unavailable.
- Dependent scoring contributions shall be removed.
- BOS, CHOCH, and liquidity logic relying on this module shall be disabled or safely degraded.

---

## 5.2 Pivot Detection Method

Type:

Selection

Initial options:

- Fixed Left/Right Pivot
- ATR-Filtered Pivot
- Percentage-Filtered Pivot
- Hybrid Pivot

Suggested first implementation:

Fixed Left/Right Pivot

Other methods may be deferred until the fixed method is validated.

---

## 5.3 Pivot Left Bars

Type:

Integer

Suggested default:

3

Minimum:

1

Purpose:

Defines the number of bars to the left required for pivot confirmation.

---

## 5.4 Pivot Right Bars

Type:

Integer

Suggested default:

3

Minimum:

1

Purpose:

Defines the number of bars to the right required for pivot confirmation.

Increasing this value shall:

- Reduce pivot frequency.
- Increase confirmation delay.
- Increase structural significance.

---

## 5.5 Minimum Swing Distance

Type:

Float

Purpose:

Prevents very small price movements from being classified as significant swings.

Possible normalisation options:

- Absolute price
- Percentage
- ATR
- Tick distance

Suggested production approach:

ATR-normalised distance

The final method remains an open design decision.

---

## 5.6 ATR Length

Type:

Integer

Suggested default:

14

Purpose:

Used when ATR-based swing filtering is enabled.

---

## 5.7 Minimum ATR Swing Multiple

Type:

Float

Suggested default:

0.50

Purpose:

Defines the minimum distance required between relevant pivots.

This value remains provisional.

---

## 5.8 Equal-Level Tolerance

Type:

Float

Purpose:

Defines how close two swing prices must be to be classified as equal.

Possible normalisation options:

- Percentage
- ATR fraction
- Tick distance

Suggested default approach:

ATR fraction

---

## 5.9 Structure Sensitivity Preset

Type:

Selection

Options:

- Fast
- Balanced
- Conservative
- Custom

Suggested default:

Balanced

Presets may adjust:

- Pivot left bars
- Pivot right bars
- Minimum swing distance
- Equal-level tolerance

---

## 5.10 Show Swing Labels

Type:

Boolean

Suggested default:

Enabled

---

## 5.11 Show Pivot Markers

Type:

Boolean

Suggested default:

Disabled

Purpose:

Allows raw pivot markers to be shown separately from HH, HL, LH, and LL labels.

---

## 5.12 Show Structure Lines

Type:

Boolean

Suggested default:

Enabled

---

## 5.13 Show Equal Highs and Equal Lows

Type:

Boolean

Suggested default:

Enabled

---

## 5.14 Maximum Visible Swing Labels

Type:

Integer

Suggested default:

50

Purpose:

Limits the number of retained labels.

---

## 5.15 Maximum Visible Structure Lines

Type:

Integer

Suggested default:

50

Purpose:

Limits the number of retained structural lines.

---

## 5.16 Use Confirmed Swings Only

Type:

Boolean

Suggested default:

Enabled

Purpose:

Controls whether chart visuals and downstream outputs use only confirmed pivots.

The production default shall be enabled.

---

# 6. Input Validation

The engine shall validate that:

- Pivot Left Bars is at least 1.
- Pivot Right Bars is at least 1.
- ATR Length is positive.
- Minimum swing distance is non-negative.
- Equal-level tolerance is non-negative.
- Object limits are positive.
- User settings do not exceed safe TradingView resource limits.

The script shall not fail due to invalid structure settings.

---

# 7. Pivot Detection

The initial implementation shall use confirmed left/right pivots.

A confirmed Swing High shall occur when a high is greater than or equal to the highs of the required surrounding bars according to the selected tie-handling policy.

A confirmed Swing Low shall occur when a low is less than or equal to the lows of the required surrounding bars according to the selected tie-handling policy.

Because right-side bars are required, pivot confirmation shall be delayed by the selected Pivot Right Bars value.

This delay is expected and shall be documented clearly.

---

# 8. Pivot Confirmation Timing

For a pivot candidate at bar index `N`:

```text
Confirmation occurs after Pivot Right Bars have closed.
```

Example:

```text
Pivot Right Bars = 3
```

A candidate at bar `N` becomes confirmed only when bar `N + 3` has closed.

The marker may be plotted back on bar `N`, but the event did not become known until `N + 3`.

Alerts shall use the actual confirmation bar.

---

# 9. Provisional Pivots

The engine may internally track possible developing pivots.

Provisional pivots may:

- Move as new highs or lows form.
- Fail before confirmation.
- Be replaced by a more extreme candidate.

Provisional pivots shall not:

- Trigger confirmed BOS or CHOCH events.
- Contribute to final historical structure.
- Trigger confirmed alerts.
- Be presented as confirmed HH, HL, LH, or LL labels.

Any provisional visualisation shall be optional and clearly differentiated.

---

# 10. Pivot Filtering

Raw pivot detection may produce excessive minor swings.

The engine shall support a significance filter.

A new pivot may be rejected or treated as minor when:

- Its distance from the previous opposite pivot is below the minimum threshold.
- It occurs too soon after the previous relevant pivot.
- It does not exceed the selected volatility-adjusted distance.
- It duplicates an existing pivot at effectively the same level.

The exact filtering order shall be recorded before implementation.

---

# 11. Alternating Pivot Sequence

The final relevant pivot sequence should normally alternate:

```text
High → Low → High → Low
```

or:

```text
Low → High → Low → High
```

When consecutive pivots of the same type occur:

- Two Swing Highs without an intervening Swing Low
- Two Swing Lows without an intervening Swing High

the engine shall retain the more structurally significant extreme.

For consecutive highs:

```text
Retain the higher high.
```

For consecutive lows:

```text
Retain the lower low.
```

The replaced pivot shall not remain in the final confirmed structural sequence.

---

# 12. Swing Classification

Each accepted confirmed pivot shall be classified relative to the previous accepted pivot of the same type.

## 12.1 Swing High Classification

A confirmed Swing High may be:

- Higher High
- Lower High
- Equal High
- Initial High

## 12.2 Swing Low Classification

A confirmed Swing Low may be:

- Higher Low
- Lower Low
- Equal Low
- Initial Low

---

# 13. Higher High Classification

A Swing High shall be classified as Higher High when:

```text
Current Swing High > Previous Relevant Swing High + Tolerance
```

---

# 14. Lower High Classification

A Swing High shall be classified as Lower High when:

```text
Current Swing High < Previous Relevant Swing High - Tolerance
```

---

# 15. Equal High Classification

A Swing High shall be classified as Equal High when:

```text
Absolute Difference ≤ Equal-Level Tolerance
```

Equal Highs shall be exposed to the Liquidity Engine as potential buy-side liquidity.

---

# 16. Higher Low Classification

A Swing Low shall be classified as Higher Low when:

```text
Current Swing Low > Previous Relevant Swing Low + Tolerance
```

---

# 17. Lower Low Classification

A Swing Low shall be classified as Lower Low when:

```text
Current Swing Low < Previous Relevant Swing Low - Tolerance
```

---

# 18. Equal Low Classification

A Swing Low shall be classified as Equal Low when:

```text
Absolute Difference ≤ Equal-Level Tolerance
```

Equal Lows shall be exposed to the Liquidity Engine as potential sell-side liquidity.

---

# 19. Initial Pivots

The first confirmed Swing High cannot be classified as HH, LH, or EH because no prior relevant Swing High exists.

It shall be classified internally as:

```text
Initial High
```

The first confirmed Swing Low shall be classified internally as:

```text
Initial Low
```

Initial pivots shall establish the structural reference sequence.

---

# 20. Structural Sequence

The engine shall maintain the most recent accepted structural pivots.

At minimum, it shall retain:

- Current Swing High
- Previous Swing High
- Current Swing Low
- Previous Swing Low
- Most recent pivot type
- Most recent pivot price
- Most recent pivot bar index
- Previous pivot price
- Previous pivot bar index

Additional history may be retained using bounded arrays.

---

# 21. Structural Direction

The engine shall produce one current structural direction.

Required states:

1. Bullish
2. Bearish
3. Neutral
4. Transition
5. Unavailable

---

# 22. Bullish Structure

Bullish structure may be confirmed when the accepted sequence contains:

```text
Higher High + Higher Low
```

A more conservative rule may require:

```text
At least two consecutive bullish structural confirmations.
```

The final confirmation rule remains an open design decision.

---

# 23. Bearish Structure

Bearish structure may be confirmed when the accepted sequence contains:

```text
Lower Low + Lower High
```

A more conservative rule may require repeated bearish structural confirmation.

---

# 24. Neutral Structure

Neutral structure may exist when:

- There is insufficient pivot history.
- Swing classifications are mixed.
- Equal highs and equal lows dominate.
- Price is range-bound.
- Neither bullish nor bearish sequence is confirmed.

---

# 25. Transition Structure

Transition may exist when:

- A bullish sequence produces a Lower High.
- A bullish sequence produces a Lower Low.
- A bearish sequence produces a Higher Low.
- A bearish sequence produces a Higher High.
- Structure becomes mixed before confirmed reversal.
- Trend and structure conflict materially.

Transition shall represent structural uncertainty.

The BOS and CHOCH specification shall define when transition becomes a confirmed structural reversal.

---

# 26. Major and Minor Structure

The architecture may support two structure layers:

- Minor Structure
- Major Structure

## 26.1 Minor Structure

Minor Structure uses more sensitive pivot settings.

It is intended to show short-term internal movement.

## 26.2 Major Structure

Major Structure uses more conservative pivot settings.

It is intended to represent broader swing direction.

The first production implementation may support only one structure layer.

Dual-layer structure remains a possible enhancement unless approved for Version 1.0.

---

# 27. Structural Reference Levels

The engine shall expose active reference levels, including:

- Most recent confirmed Swing High
- Most recent confirmed Swing Low
- Previous confirmed Swing High
- Previous confirmed Swing Low
- Active bullish protection level
- Active bearish protection level
- Most recent Equal High
- Most recent Equal Low

These levels shall be consumed by BOS, CHOCH, and Liquidity modules.

---

# 28. Bullish Protection Level

In bullish structure, the active bullish protection level may be:

```text
Most recent confirmed Higher Low
```

A confirmed break below this level may indicate:

- Bullish structure failure
- CHOCH
- Bearish BOS
- Transition

The BOS and CHOCH specification shall define the event classification.

---

# 29. Bearish Protection Level

In bearish structure, the active bearish protection level may be:

```text
Most recent confirmed Lower High
```

A confirmed break above this level may indicate:

- Bearish structure failure
- CHOCH
- Bullish BOS
- Transition

---

# 30. Trend Engine Integration

The Market Structure Engine shall remain analytically independent from the Trend Engine.

However, combined context shall be available.

Possible combinations:

| Trend | Structure | Interpretation |
|---|---|---|
| Bullish | Bullish | Strong directional agreement |
| Bullish | Bearish | Conflict or pullback |
| Bearish | Bearish | Strong directional agreement |
| Bearish | Bullish | Conflict or reversal attempt |
| Neutral | Bullish | Emerging bullish structure |
| Neutral | Bearish | Emerging bearish structure |
| Transition | Mixed | High uncertainty |

Trend shall not redefine structural pivots.

Structure shall not alter moving-average calculations.

---

# 31. BOS and CHOCH Integration

The Market Structure Engine shall provide:

- Confirmed swing levels
- Structural direction
- Protection levels
- Pivot classifications
- Pivot confirmation bars

The BOS and CHOCH module shall determine whether price action constitutes:

- Bullish Break of Structure
- Bearish Break of Structure
- Bullish Change of Character
- Bearish Change of Character
- Failed break
- Intrabar breach
- Confirmed close beyond structure

The Market Structure Engine itself shall not label breaks.

---

# 32. Liquidity Integration

The engine shall expose:

- Equal High levels
- Equal Low levels
- Recent Swing Highs
- Recent Swing Lows
- Unbroken structural levels
- Previously swept levels where tracked

The Liquidity Engine shall interpret:

- Sweeps
- Stop runs
- Rejections
- Acceptances
- Failed breaks

---

# 33. VPA Integration

The VPA Engine may interpret volume events differently when they occur at:

- Confirmed Swing Highs
- Confirmed Swing Lows
- Higher Lows
- Lower Highs
- Equal Highs
- Equal Lows
- Structure boundaries

Examples:

- Stopping volume near a Swing Low
- No demand near a Lower High
- No supply near a Higher Low
- Climactic volume at a structural extreme

---

# 34. Wyckoff Integration

The Wyckoff Engine may use structural pivots to help define:

- Trading ranges
- Springs
- Upthrusts
- Secondary tests
- Signs of strength
- Signs of weakness
- Last points of support
- Last points of supply

The Wyckoff Engine shall not create a separate incompatible pivot system unless explicitly approved.

---

# 35. Institutional Activity Integration

Institutional events may gain significance when they occur near:

- Major Swing Highs
- Major Swing Lows
- Equal Highs
- Equal Lows
- Structural protection levels
- Recently broken structure

The Market Structure Engine provides location context.

The Institutional Engine provides interpretation.

---

# 36. Outputs

The Market Structure Engine shall expose:

- Engine enabled status
- Pivot confirmed event
- Pivot type
- Pivot price
- Pivot bar index
- Swing classification
- Previous Swing High price
- Current Swing High price
- Previous Swing Low price
- Current Swing Low price
- Structural direction
- Structural transition status
- Bullish protection level
- Bearish protection level
- Equal High event
- Equal Low event
- Equal High level
- Equal Low level
- Sufficient-history status
- Structure confidence where implemented

---

# 37. Suggested Enumerations

## Pivot Type

```text
+1 = Swing High
-1 = Swing Low
 0 = None
```

## Swing Classification

```text
 0 = Unavailable
 1 = Initial High
 2 = Initial Low
 3 = Higher High
 4 = Higher Low
 5 = Lower High
 6 = Lower Low
 7 = Equal High
 8 = Equal Low
```

## Structural Direction

```text
+1 = Bullish
 0 = Neutral
-1 = Bearish
 2 = Transition
```

Internal values shall remain stable after implementation begins.

Display text shall be generated separately.

---

# 38. Dashboard Integration

The dashboard shall be able to display:

- Structure direction
- Last confirmed swing
- Last swing classification
- Active protection level
- Equal High or Equal Low status
- Structure and trend agreement

Suggested display:

```text
Structure: Bullish
Last Swing: HL
Protection: 1.24850
Trend Alignment: Confirmed
```

When insufficient data exists:

```text
Structure: Building
```

or:

```text
Structure: N/A
```

---

# 39. Chart Visualisation

Optional visual elements may include:

- HH labels
- HL labels
- LH labels
- LL labels
- EH labels
- EL labels
- Pivot markers
- Swing lines
- Structural reference lines
- Active protection lines
- Structure-coloured labels

Visuals shall remain configurable.

Default labels shall be compact.

---

# 40. Label Placement

Swing High labels should normally appear above the pivot bar.

Swing Low labels should normally appear below the pivot bar.

Labels shall use the original pivot bar index, even though confirmation occurs later.

The tooltip or documentation should clarify that pivots are confirmed after the required right-side bars.

---

# 41. Object Management

The engine shall manage labels and lines using bounded collections.

When maximum visible objects are exceeded:

- The oldest object shall be deleted.
- The corresponding reference shall be removed from its storage structure.
- Active structural reference objects shall be preserved where necessary.

The engine shall not create unlimited labels or lines.

---

# 42. Historical Data Requirements

Before full classification is available, the engine requires:

- At least two confirmed Swing Highs for HH/LH/EH classification.
- At least two confirmed Swing Lows for HL/LL/EL classification.
- A sufficient alternating pivot sequence for structural direction.

Until sufficient history exists, outputs shall be:

- Building
- Neutral
- Unavailable

depending on the approved display policy.

---

# 43. Repainting Policy

Confirmed pivots shall not move after confirmation.

A pivot marker may be placed on a historical bar when it becomes confirmed later.

This is delayed confirmation, not future leakage, provided:

- The pivot is not used before confirmation.
- Alerts trigger only on the confirmation bar.
- Downstream events use the confirmation timestamp.
- Lookahead is not used.

Provisional pivots may change before confirmation and shall remain clearly separated.

---

# 44. Real-Time Behaviour

On the current developing bar:

- Price may approach or exceed a structural level.
- No pivot requiring future right-side bars can yet be confirmed.
- Intrabar structural breaches may be observed by BOS/CHOCH logic.
- Confirmed structural state shall update only according to bar-close rules.

The dashboard may optionally show developing conditions, but confirmed status shall remain distinct.

---

# 45. Performance Requirements

The engine shall:

- Avoid scanning unlimited historical bars.
- Use built-in pivot functions where appropriate.
- Maintain bounded pivot arrays.
- Reuse stored pivot values.
- Avoid creating objects unless their display is enabled.
- Delete old objects promptly.
- Avoid repeated classification string construction.
- Keep structure calculations independent of visual rendering.

---

# 46. Error Handling

The engine shall safely handle:

- Insufficient chart history
- Duplicate pivot prices
- Consecutive same-type pivots
- Missing values
- Extremely volatile bars
- Illiquid symbols
- Large gaps
- Invalid tolerance settings
- Object-limit pressure
- Symbol changes
- Timeframe changes

No invalid state shall cause a runtime error.

---

# 47. Testing Requirements

The engine shall be tested across:

- Stocks
- Indices
- Forex
- Cryptocurrency
- Futures
- Commodities

Required timeframe tests:

- 1 minute
- 5 minute
- 15 minute
- 1 hour
- 4 hour
- Daily
- Weekly

Required condition tests:

- Clean uptrend
- Clean downtrend
- Sideways range
- Volatile range
- Equal highs
- Equal lows
- Gap movement
- Illiquid price action
- Rapid reversal
- Extended trend
- Limited history
- Repeated same-type pivot candidates

---

# 48. Functional Test Cases

## Test STRUCT-001 — Confirmed Swing High

Given:

- A pivot-high candidate exists.
- Required right-side bars close below the candidate high.

Expected:

- Swing High is confirmed after the configured delay.
- Pivot price and bar index are stored.
- No confirmation occurs early.

---

## Test STRUCT-002 — Confirmed Swing Low

Given:

- A pivot-low candidate exists.
- Required right-side bars close above the candidate low.

Expected:

- Swing Low is confirmed after the configured delay.
- Pivot price and bar index are stored.

---

## Test STRUCT-003 — Higher High

Given:

- A previous confirmed Swing High exists.
- A new confirmed Swing High exceeds it beyond tolerance.

Expected:

- New Swing High classification is HH.

---

## Test STRUCT-004 — Lower High

Given:

- A previous confirmed Swing High exists.
- A new confirmed Swing High is below it beyond tolerance.

Expected:

- New Swing High classification is LH.

---

## Test STRUCT-005 — Higher Low

Given:

- A previous confirmed Swing Low exists.
- A new confirmed Swing Low is above it beyond tolerance.

Expected:

- New Swing Low classification is HL.

---

## Test STRUCT-006 — Lower Low

Given:

- A previous confirmed Swing Low exists.
- A new confirmed Swing Low is below it beyond tolerance.

Expected:

- New Swing Low classification is LL.

---

## Test STRUCT-007 — Equal High

Given:

- A new Swing High is within tolerance of the previous Swing High.

Expected:

- Classification is EH.
- Equal High level is exposed to the Liquidity Engine.

---

## Test STRUCT-008 — Equal Low

Given:

- A new Swing Low is within tolerance of the previous Swing Low.

Expected:

- Classification is EL.
- Equal Low level is exposed to the Liquidity Engine.

---

## Test STRUCT-009 — Bullish Structure

Given:

- Confirmed sequence contains HH and HL according to the approved confirmation rule.

Expected:

- Structural Direction becomes Bullish.

---

## Test STRUCT-010 — Bearish Structure

Given:

- Confirmed sequence contains LL and LH according to the approved confirmation rule.

Expected:

- Structural Direction becomes Bearish.

---

## Test STRUCT-011 — Same-Type Pivot Replacement

Given:

- Two Swing High candidates occur without an accepted Swing Low between them.
- The second high exceeds the first.

Expected:

- The more extreme Swing High is retained.
- Final structure does not contain duplicate consecutive highs.

---

## Test STRUCT-012 — Pivot Confirmation Delay

Given:

- Pivot Right Bars equals 3.

Expected:

- A pivot at bar `N` is not confirmed before bar `N + 3` closes.
- Alert occurs on the confirmation bar, not the pivot bar.

---

## Test STRUCT-013 — Insufficient History

Given:

- Only one confirmed Swing High and one confirmed Swing Low exist.

Expected:

- Structure remains Building, Neutral, or Unavailable.
- No false HH, HL, LH, or LL classification occurs.

---

## Test STRUCT-014 — Object Limit

Given:

- The number of structure labels exceeds the configured maximum.

Expected:

- The oldest label is deleted.
- Current structure remains visible.
- No runtime object-limit failure occurs.

---

## Test STRUCT-015 — Historical Stability

Given:

- A pivot has been confirmed.
- Additional future bars are loaded.

Expected:

- The confirmed pivot price and bar index do not change unexpectedly.

---

# 49. Acceptance Criteria

The Market Structure Engine shall be complete when:

- Confirmed Swing Highs and Swing Lows are detected correctly.
- Pivot confirmation delay is respected.
- HH, HL, LH, LL, EH, and EL classifications work correctly.
- Consecutive same-type pivots are resolved safely.
- Structural direction is produced.
- Active protection levels are available.
- Equal levels are exposed to the Liquidity Engine.
- Confirmed pivots remain historically stable.
- Provisional and confirmed states remain separate.
- Dashboard outputs are available.
- Object counts remain bounded.
- Insufficient history is handled safely.
- TradingView compilation succeeds.
- Required tests pass.

---

# 50. Open Design Decisions

The following items remain unresolved:

1. Final pivot detection method.
2. Final default left/right pivot values.
3. Whether left and right values must be equal.
4. Final minimum swing-distance method.
5. Final ATR multiple.
6. Final equal-level tolerance method.
7. Final bullish structure confirmation rule.
8. Final bearish structure confirmation rule.
9. Whether Transition is a separate structure state.
10. Whether Major and Minor Structure are both included in Version 1.0.
11. Whether raw pivots are shown in the first alpha.
12. Final same-type pivot replacement rules.
13. Maximum pivot-history depth.
14. Whether active protection levels are drawn by default.
15. Whether equal levels expire after a sweep or confirmed break.
16. Whether structure confidence is numerical.
17. Whether structure labels are placed on the pivot bar or confirmation bar.
18. Whether developing pivots are visible in debug mode only.

---

# 51. Future Enhancements

Possible future enhancements include:

- Dual-layer internal and swing structure
- Adaptive pivot sensitivity
- Volatility-regime-specific pivot settings
- Fractal hierarchy
- Multi-timeframe structure
- Session structure
- Structural range detection
- Automatic support and resistance zones
- Structural leg momentum
- Structural leg volume analysis
- Pivot-quality scoring
- Swing-duration analysis