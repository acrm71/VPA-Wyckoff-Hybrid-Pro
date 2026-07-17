# BOS and CHOCH Specification

---

## Document Information

**Document ID:** SPEC-105

**Version:** 0.1 Draft

**Status:** Draft

**Module:** Break of Structure and Change of Character Engine

**Dependencies:**

- Framework
- Settings
- Market Structure Engine
- Trend Engine
- Relative Volume Engine
- Dashboard
- Alerts

**Dependent Modules:**

- Liquidity
- VPA
- Wyckoff
- Institutional Activity
- Signal Scoring

---

# 1. Purpose

The BOS and CHOCH Engine classifies price interactions with confirmed structural levels.

Its primary responsibilities are to identify:

- Bullish Break of Structure
- Bearish Break of Structure
- Bullish Change of Character
- Bearish Change of Character
- Wick breaches
- Close-confirmed breaks
- Failed structural breaks
- Rejections
- Acceptance beyond structure
- Structural continuation
- Structural reversal attempts

The engine shall convert confirmed Market Structure levels into structured market events.

It shall not independently generate trade entries.

---

# 2. Objectives

The engine shall:

- Use confirmed structural levels as authoritative references.
- Distinguish wick breaches from close-confirmed breaks.
- Distinguish continuation events from reversal events.
- Classify BOS and CHOCH consistently.
- Prevent duplicate event generation.
- Track whether a structural level remains active, broken, swept, or invalidated.
- Provide reusable events to Liquidity, Wyckoff, Institutional, Scoring, Dashboard, and Alert modules.
- Avoid intentional historical repainting.
- Support configurable confirmation strictness.
- Preserve event timing and reference-level metadata.

---

# 3. Design Principles

## 3.1 Unified Structural Event Pipeline

All structural interactions shall follow one common processing pipeline:

1. Reference level identified
2. Level eligibility checked
3. Price approach detected
4. Wick interaction evaluated
5. Close interaction evaluated
6. Confirmation rules applied
7. Structural context evaluated
8. Event classified
9. Confidence calculated
10. Event published

## 3.2 Confirmed Structure First

Only confirmed Market Structure levels shall produce confirmed BOS or CHOCH events.

Provisional pivots shall not be authoritative references.

## 3.3 Breach Is Not Confirmation

A wick beyond a structural level shall not automatically become BOS or CHOCH.

## 3.4 Context Determines Classification

The same directional break may represent BOS or CHOCH depending on the prior confirmed structural direction.

## 3.5 One Event Per Structural Interaction

The engine shall avoid repeatedly emitting the same event while price remains beyond a level.

## 3.6 Measurement Before Interpretation

The engine shall classify structural behaviour.

Liquidity, Wyckoff, and Institutional modules shall provide deeper contextual interpretation.

---

# 4. Core Definitions

## 4.1 Structural Reference Level

A Structural Reference Level is an eligible confirmed level supplied by the Market Structure Engine.

Possible references include:

- Current confirmed Swing High
- Current confirmed Swing Low
- Bullish protection level
- Bearish protection level
- Equal High
- Equal Low
- Previous unbroken structural level

## 4.2 Bullish Break

A Bullish Break occurs when price moves above an eligible structural high.

## 4.3 Bearish Break

A Bearish Break occurs when price moves below an eligible structural low.

## 4.4 Break of Structure

A Break of Structure is a confirmed break in the same direction as the prevailing confirmed structure.

Conceptually:

```text
Bullish structure + confirmed break above structural high = Bullish BOS
```

```text
Bearish structure + confirmed break below structural low = Bearish BOS
```

## 4.5 Change of Character

A Change of Character is a confirmed break against the prevailing confirmed structure.

Conceptually:

```text
Bearish structure + confirmed break above bearish protection level = Bullish CHOCH
```

```text
Bullish structure + confirmed break below bullish protection level = Bearish CHOCH
```

## 4.6 Wick Breach

A Wick Breach occurs when the high or low crosses a structural level, but the close does not satisfy the close-confirmation rule.

## 4.7 Close Break

A Close Break occurs when the bar closes beyond a structural reference level according to the configured confirmation threshold.

## 4.8 Acceptance

Acceptance occurs when price closes beyond a structural level and demonstrates continued trading beyond that level.

## 4.9 Rejection

Rejection occurs when price trades through or touches a structural level but closes back on the original side.

## 4.10 Failed Break

A Failed Break occurs when an initially confirmed break loses acceptance and price returns through the broken level within a configured validation window.

---

# 5. Inputs

## 5.1 Enable BOS and CHOCH Engine

Type:

Boolean

Default:

Enabled

When disabled:

- No BOS or CHOCH events shall be generated.
- Related visuals shall be hidden.
- Related alerts shall be disabled.
- Dependent modules shall receive unavailable event states.

---

## 5.2 Break Confirmation Mode

Type:

Selection

Initial options:

- Wick
- Close
- Close Plus Buffer
- Close Plus Acceptance

Suggested production default:

Close

---

## 5.3 Break Buffer Method

Type:

Selection

Options:

- None
- Tick
- Percentage
- ATR

Suggested default:

ATR

---

## 5.4 Break Buffer Value

Type:

Float

Suggested provisional default:

```text
0.05 ATR
```

Purpose:

Prevents marginal closes beyond structure from being treated as meaningful breaks.

---

## 5.5 Acceptance Bars

Type:

Integer

Suggested default:

1

Minimum:

1

Purpose:

Defines the number of confirmed closes required beyond the structural level when acceptance confirmation is enabled.

---

## 5.6 Failure Validation Window

Type:

Integer

Suggested default:

3

Purpose:

Defines the number of bars after a confirmed structural break during which the event may be reclassified or supplemented as a Failed Break.

Confirmed BOS or CHOCH history shall not be deleted.

A later failure event shall be emitted separately.

---

## 5.7 Require Body Close Beyond Level

Type:

Boolean

Suggested default:

Enabled

Purpose:

When enabled, the closing price must be beyond the structural level.

A wick breach alone shall not confirm BOS or CHOCH.

---

## 5.8 Minimum Candle Body Strength

Type:

Float

Purpose:

Allows confirmation to require a minimum candle-body proportion relative to total range.

Suggested provisional default:

```text
0.40
```

This filter may be optional in the first alpha.

---

## 5.9 Require Directional Candle

Type:

Boolean

Suggested default:

Disabled

Purpose:

When enabled:

- Bullish breaks require a bullish confirmation candle.
- Bearish breaks require a bearish confirmation candle.

This shall remain optional because valid structural breaks can occur on non-directional or gap bars.

---

## 5.10 Require RVOL Confirmation

Type:

Boolean

Suggested default:

Disabled

Purpose:

When enabled, confirmed BOS or CHOCH shall require minimum relative volume.

This option shall be treated as a strict filter rather than the default behaviour.

---

## 5.11 Minimum Break RVOL

Type:

Float

Suggested default:

1.20

Purpose:

Defines the minimum RVOL required when RVOL confirmation is enabled.

---

## 5.12 Show BOS Labels

Type:

Boolean

Suggested default:

Enabled

---

## 5.13 Show CHOCH Labels

Type:

Boolean

Suggested default:

Enabled

---

## 5.14 Show Wick Breaches

Type:

Boolean

Suggested default:

Disabled

---

## 5.15 Show Failed Breaks

Type:

Boolean

Suggested default:

Enabled

---

## 5.16 Show Structural Break Lines

Type:

Boolean

Suggested default:

Enabled

---

## 5.17 Maximum Visible Events

Type:

Integer

Suggested default:

50

Purpose:

Limits retained BOS, CHOCH, sweep, rejection, and failure objects.

---

## 5.18 Confirm Events at Bar Close

Type:

Boolean

Suggested default:

Enabled

The production default shall be enabled.

---

# 6. Input Validation

The engine shall validate that:

- Acceptance Bars is at least 1.
- Failure Validation Window is non-negative.
- Break Buffer Value is non-negative.
- Minimum Candle Body Strength is between 0 and 1.
- Minimum Break RVOL is non-negative.
- Maximum Visible Events is positive.
- Buffer settings are compatible with the selected method.

Invalid configuration shall not cause runtime failure.

---

# 7. Reference-Level Eligibility

A structural level shall be eligible only when:

- It is based on a confirmed pivot.
- It has not been invalidated under the approved lifecycle policy.
- It has not already produced an equivalent terminal event unless retesting is permitted.
- It belongs to the active structure context.
- Its source price and bar index are known.

The engine shall record the source of every level.

Suggested sources:

```text
Swing High
Swing Low
Bullish Protection
Bearish Protection
Equal High
Equal Low
Previous Structure
```

---

# 8. Reference-Level Lifecycle

Each eligible structural level shall have one lifecycle state.

Suggested states:

1. Active
2. Approached
3. Wick Breached
4. Close Broken
5. Accepted
6. Rejected
7. Failed
8. Invalidated
9. Expired

The exact internal state model shall remain stable after implementation begins.

---

# 9. Approach Detection

A level may be classified as Approached when price enters a configurable proximity zone.

Approach detection may support:

- Dashboard context
- Debug output
- Future pre-alerts
- Liquidity analysis

Approach detection is optional for the first alpha implementation.

It shall not produce BOS or CHOCH by itself.

---

# 10. Wick Interaction

## 10.1 Bullish Wick Breach

A bullish Wick Breach occurs when:

```text
High > Structural High + Required Buffer
```

while:

```text
Close ≤ Structural High + Required Close Buffer
```

## 10.2 Bearish Wick Breach

A bearish Wick Breach occurs when:

```text
Low < Structural Low - Required Buffer
```

while:

```text
Close ≥ Structural Low - Required Close Buffer
```

Wick breaches shall be made available to the Liquidity Engine.

They shall not automatically become BOS or CHOCH.

---

# 11. Close Confirmation

## 11.1 Bullish Close Break

A bullish Close Break occurs when:

```text
Close > Structural High + Confirmation Buffer
```

## 11.2 Bearish Close Break

A bearish Close Break occurs when:

```text
Close < Structural Low - Confirmation Buffer
```

The event shall be evaluated only on the confirmation bar unless intrabar mode is explicitly enabled in a future release.

---

# 12. Buffer Calculation

The confirmation buffer may be calculated using:

## 12.1 No Buffer

```text
Buffer = 0
```

## 12.2 Tick Buffer

```text
Buffer = Minimum Tick × User Multiplier
```

## 12.3 Percentage Buffer

```text
Buffer = Structural Level × Percentage
```

## 12.4 ATR Buffer

```text
Buffer = ATR × User Multiplier
```

ATR-based buffering is the suggested production default because it adapts to volatility and instrument scale.

---

# 13. BOS Classification

## 13.1 Bullish BOS

A Bullish BOS shall require:

- Confirmed structural direction is Bullish.
- An eligible structural high exists.
- Price satisfies the bullish break-confirmation rule.
- The level has not already produced the same active BOS event.
- Required confirmation filters pass.

## 13.2 Bearish BOS

A Bearish BOS shall require:

- Confirmed structural direction is Bearish.
- An eligible structural low exists.
- Price satisfies the bearish break-confirmation rule.
- The level has not already produced the same active BOS event.
- Required confirmation filters pass.

BOS shall represent structural continuation.

---

# 14. CHOCH Classification

## 14.1 Bullish CHOCH

A Bullish CHOCH shall require:

- Confirmed structural direction is Bearish.
- An eligible bearish protection level or relevant structural high exists.
- Price confirms a break above that level.
- Required confirmation filters pass.

## 14.2 Bearish CHOCH

A Bearish CHOCH shall require:

- Confirmed structural direction is Bullish.
- An eligible bullish protection level or relevant structural low exists.
- Price confirms a break below that level.
- Required confirmation filters pass.

CHOCH shall represent a structural reversal attempt or material structural warning.

CHOCH shall not automatically establish a fully reversed trend.

---

# 15. Neutral-Structure Breaks

When confirmed structural direction is Neutral or Unavailable, a structural break shall not automatically be labelled BOS or CHOCH.

Possible classification:

- Bullish Structural Break
- Bearish Structural Break
- Range Break
- Unclassified Break

The final naming policy remains an open design decision.

This avoids falsely assigning continuation or reversal context where no confirmed directional structure exists.

---

# 16. Transition-Structure Breaks

When Market Structure is Transition:

- A break aligned with the emerging direction may be classified as a provisional continuation or reversal confirmation.
- A break against the emerging direction may indicate transition failure.

The first production release should use conservative classification.

Where context is ambiguous, the event should remain:

```text
Structural Break
```

rather than forcing BOS or CHOCH.

---

# 17. Acceptance Confirmation

Acceptance may be confirmed using one or more of the following:

- One close beyond the level
- Multiple consecutive closes beyond the level
- Retest that holds beyond the level
- Minimum body distance beyond the level
- Minimum time spent beyond the level

Suggested first implementation:

```text
One confirmed close beyond level
```

Optional strict mode:

```text
Two consecutive confirmed closes beyond level
```

---

# 18. Rejection Classification

A Rejection may occur when:

- Price touches or breaches the level.
- Price closes back on the original side.
- No valid close-confirmed break occurs.

Examples:

```text
High exceeds Swing High
Close returns below Swing High
```

```text
Low exceeds below Swing Low
Close returns above Swing Low
```

Rejection events shall be passed to the Liquidity and VPA modules.

---

# 19. Failed Break Classification

A Failed Bullish Break may occur when:

1. Price confirms a break above structure.
2. The event is initially classified as Bullish BOS, Bullish CHOCH, or Bullish Structural Break.
3. Within the Failure Validation Window, price closes back below the broken level.
4. Acceptance has not persisted according to the approved rule.

A Failed Bearish Break shall use the inverse logic.

The original break event shall remain recorded.

A separate failure event shall be generated.

---

# 20. Retest Behaviour

After a confirmed break, price may retest the structural level.

Possible retest states:

- Retest Pending
- Retest Holding
- Retest Failed
- No Retest
- Retest Expired

Retest tracking is optional for the first alpha implementation.

If implemented, a successful bullish retest may require:

```text
Low tests the broken level
Close remains above the level
```

A successful bearish retest may require:

```text
High tests the broken level
Close remains below the level
```

---

# 21. Duplicate-Event Suppression

The engine shall prevent repeated events from the same structural level.

After a confirmed event:

- The source level shall be marked as processed for that event type.
- Repeated closes beyond the same level shall not generate repeated BOS or CHOCH events.
- A new event may occur only after a new eligible structural level is established or a distinct lifecycle event occurs.

Distinct later events may include:

- Failed Break
- Retest Hold
- Retest Failure
- Re-entry
- Opposite Structural Break

---

# 22. Level Consumption Policy

A structural level may be considered consumed when:

- A confirmed break occurs.
- Acceptance is established.
- A replacement structural level becomes active.
- The level expires under history limits.

Wick-only breaches shall not necessarily consume the level.

The final level-consumption policy shall be approved before implementation.

---

# 23. Event Direction

Every event shall have a directional value.

Suggested values:

```text
+1 = Bullish
 0 = Neutral or unavailable
-1 = Bearish
```

---

# 24. Event Classification

Required event classes:

```text
None
Bullish BOS
Bearish BOS
Bullish CHOCH
Bearish CHOCH
Bullish Structural Break
Bearish Structural Break
Bullish Wick Breach
Bearish Wick Breach
Bullish Rejection
Bearish Rejection
Failed Bullish Break
Failed Bearish Break
Bullish Acceptance
Bearish Acceptance
```

Some event classes may be deferred from the first alpha but shall remain architecturally supported.

---

# 25. Event Metadata

Each structural event shall retain:

- Event type
- Event direction
- Event bar index
- Event timestamp
- Reference-level price
- Reference-level type
- Reference pivot bar index
- Structural direction before event
- Trend direction before event
- RVOL value where available
- Confirmation mode
- Buffer used
- Event confidence
- Failure status
- Acceptance status

Event history shall be bounded.

---

# 26. Event Confidence

The engine may produce an event-confidence score.

Suggested range:

```text
0 to 100
```

Possible components:

| Component | Suggested Weight |
|---|---:|
| Close distance beyond level | 20 |
| Candle body quality | 15 |
| RVOL confirmation | 15 |
| Trend alignment | 15 |
| Structural clarity | 20 |
| Acceptance or follow-through | 15 |

These weights are provisional.

Event confidence shall not replace the project-wide Signal Score.

It represents the quality of the structural event only.

---

# 27. Structural Clarity

Structural clarity may be higher when:

- The reference pivot is well separated.
- The level has not been repeatedly crossed.
- The prior structure is clearly directional.
- The break is not occurring inside heavy structural compression.
- The level is recent and relevant.

Structural clarity may be lower when:

- Multiple nearby pivots exist.
- Equal levels create ambiguity.
- Structure is Neutral or Transition.
- The level is old.
- Price has repeatedly crossed the level.

---

# 28. Trend Engine Integration

The Trend Engine shall provide context but shall not redefine BOS or CHOCH.

Possible interactions:

| Structure Event | Trend Context | Interpretation |
|---|---|---|
| Bullish BOS | Bullish Trend | Continuation confirmation |
| Bullish BOS | Bearish Trend | Structural conflict |
| Bullish CHOCH | Bearish Trend | Early reversal warning |
| Bullish CHOCH | Bullish Trend | Trend recovery |
| Bearish BOS | Bearish Trend | Continuation confirmation |
| Bearish CHOCH | Bullish Trend | Early reversal warning |

Trend alignment may affect event confidence and scoring.

---

# 29. RVOL Integration

Relative Volume may strengthen or weaken structural-event confidence.

Possible interpretation:

- High RVOL with BOS may support conviction.
- Extreme RVOL with BOS may support breakout or signal climax risk.
- Low RVOL with BOS may indicate weak participation.
- High RVOL with failed break may support absorption or trap interpretation.
- Missing RVOL shall not invalidate the event unless strict RVOL confirmation is enabled.

The BOS and CHOCH Engine shall remain operational when volume is unavailable.

---

# 30. Liquidity Integration

The Liquidity Engine shall consume:

- Wick breach events
- Rejection events
- Failed Break events
- Equal High and Equal Low interactions
- Reference-level metadata
- Close-confirmed break events
- Acceptance status

The Liquidity Engine shall determine whether the event represents:

- Buy-side liquidity sweep
- Sell-side liquidity sweep
- Stop run
- Failed auction
- Accepted breakout
- Liquidity trap

The BOS and CHOCH Engine shall not duplicate that interpretation.

---

# 31. VPA Integration

The VPA Engine may combine structural events with:

- Spread
- Close location
- RVOL
- Candle direction
- Effort versus result
- Absorption characteristics

Examples:

- High-volume bullish BOS with wide spread
- Narrow-spread failed bullish break on extreme RVOL
- Low-volume retest after bullish BOS
- Bearish CHOCH with climactic selling volume

---

# 32. Wyckoff Integration

The Wyckoff Engine may use structural-event outputs when classifying:

- Spring
- Upthrust
- Sign of Strength
- Sign of Weakness
- Last Point of Support
- Last Point of Supply
- Secondary Test

Examples:

```text
Sell-side wick breach
+
Close back above range low
=
Spring candidate
```

```text
Bullish BOS
+
Range acceptance
+
High RVOL
=
Sign of Strength candidate
```

---

# 33. Institutional Activity Integration

Institutional interpretation may be strengthened by combinations such as:

- High RVOL failed BOS
- Extreme-volume CHOCH
- Structural break with absorption
- Breakout followed by rapid re-entry
- Strong acceptance beyond a major swing level

The Institutional module shall perform the interpretation.

---

# 34. Scoring Integration

Suggested scoring effects:

- BOS aligned with trend increases continuation confidence.
- CHOCH against trend increases reversal-watch confidence.
- Failed BOS applies a continuation penalty.
- Failed CHOCH reduces reversal confidence.
- Acceptance increases event confidence.
- Wick-only breach shall not receive full structural-break weight.
- High RVOL may increase structural-event weight.
- Low RVOL may reduce breakout conviction.
- Neutral-structure breaks receive lower confidence.

The final scoring rules belong to the Signal Scoring specification.

---

# 35. Outputs

The engine shall expose:

- Engine enabled status
- Active reference high
- Active reference low
- Reference-level source
- Bullish wick-breach event
- Bearish wick-breach event
- Bullish close-break event
- Bearish close-break event
- Bullish BOS event
- Bearish BOS event
- Bullish CHOCH event
- Bearish CHOCH event
- Bullish rejection event
- Bearish rejection event
- Failed bullish break event
- Failed bearish break event
- Bullish acceptance event
- Bearish acceptance event
- Last structural event type
- Last structural event direction
- Last event level
- Last event bar index
- Last event confidence
- Last event failure status
- Last event acceptance status

---

# 36. Suggested Enumerations

## Event Type

```text
 0 = None
 1 = Bullish Wick Breach
 2 = Bearish Wick Breach
 3 = Bullish Structural Break
 4 = Bearish Structural Break
 5 = Bullish BOS
 6 = Bearish BOS
 7 = Bullish CHOCH
 8 = Bearish CHOCH
 9 = Bullish Rejection
10 = Bearish Rejection
11 = Failed Bullish Break
12 = Failed Bearish Break
13 = Bullish Acceptance
14 = Bearish Acceptance
```

## Event Status

```text
0 = Inactive
1 = Developing
2 = Confirmed
3 = Failed
4 = Expired
```

Display strings shall remain separate from internal state values.

---

# 37. Dashboard Integration

The dashboard shall be able to display:

- Last structural event
- Event direction
- Reference level
- Event confidence
- Acceptance status
- Failure warning
- Trend alignment
- RVOL context

Suggested output:

```text
Structure Event: Bullish BOS
Level: 1.25420
Confidence: 78
Status: Accepted
```

For wick-only interaction:

```text
Structure Event: High Sweep
Status: Rejected
```

The exact liquidity terminology shall be controlled by the Liquidity module.

---

# 38. Chart Visualisation

Optional chart elements may include:

- BOS label
- CHOCH label
- Wick-breach marker
- Rejection marker
- Failed-break marker
- Horizontal line at broken level
- Retest marker
- Acceptance marker

Suggested label abbreviations:

```text
BOS
CHOCH
WB
FB
ACC
REJ
```

Visual settings shall remain configurable.

---

# 39. Event Placement

A structural event label shall normally be placed on the bar where the event becomes confirmed.

The reference line may extend from the source pivot to the confirmation bar.

The source pivot bar and confirmation bar shall remain distinct.

This avoids implying that the event was known at the original pivot.

---

# 40. Alert Integration

The engine shall support alert conditions for:

- Bullish BOS confirmed
- Bearish BOS confirmed
- Bullish CHOCH confirmed
- Bearish CHOCH confirmed
- Bullish wick breach
- Bearish wick breach
- Bullish rejection
- Bearish rejection
- Failed bullish break
- Failed bearish break
- Bullish acceptance
- Bearish acceptance

Default alerts shall use confirmed bar-close events.

Wick-breach alerts should be optional because they may be frequent.

Alert messages shall include:

- Symbol
- Chart timeframe
- Event type
- Event direction
- Structural level
- Trend context
- RVOL where available
- Event confidence
- Timestamp where supported

Example:

```text
EURUSD 1H — Bullish CHOCH confirmed above 1.08450 — Confidence 74
```

---

# 41. Repainting Policy

The engine shall not intentionally repaint confirmed BOS or CHOCH events.

Requirements:

- Only confirmed structural levels shall be used.
- No lookahead shall be used.
- Confirmed events shall use closed-bar data by default.
- Historical event classifications shall not be removed.
- Failed Break shall be emitted as a later event rather than rewriting the original event.
- Provisional intrabar states shall remain separate from confirmed events.

---

# 42. Real-Time Behaviour

During a developing bar:

- A wick breach may appear and disappear.
- A close break is not confirmed until bar close.
- Event confidence may change.
- Acceptance cannot be final until its required confirmation bars close.

The production dashboard shall distinguish:

```text
Developing
```

from:

```text
Confirmed
```

where developing information is exposed.

---

# 43. Event Ordering

When multiple structural conditions occur on the same bar, the engine shall apply a deterministic priority.

Suggested evaluation order:

1. Existing event failure
2. Active reference-level wick interaction
3. Close-confirmed break
4. BOS or CHOCH contextual classification
5. Acceptance
6. New reference-level registration

This order remains provisional.

The final order shall prevent a newly created pivot from incorrectly affecting an event already occurring on the same bar.

---

# 44. Gap Handling

A market gap may open beyond a structural level without trading through it intrabar.

The engine shall classify the event based on:

- Open location
- High and low
- Close location
- Confirmation mode
- Acceptance behaviour

A gap close beyond structure may qualify as a Close Break.

It shall not be classified as a Wick Breach unless price actually trades through and rejects the level.

Gap-specific metadata may be retained for later interpretation.

---

# 45. Equal-Level Handling

Breaking an Equal High or Equal Low may represent both:

- Structural interaction
- Liquidity interaction

The event pipeline shall retain the level source.

For example:

```text
Reference Source: Equal High
Event: Bullish Close Break
```

The Liquidity Engine can then classify the event as:

- Buy-side liquidity taken
- Accepted breakout
- Sweep and rejection
- Failed breakout

---

# 46. Event History

The engine may retain a bounded event history.

Each stored record may include:

- Event type
- Direction
- Level
- Event bar index
- Source pivot bar index
- Confidence
- Accepted status
- Failed status

Maximum history depth shall be configurable internally and remain within Pine Script limits.

---

# 47. Object Management

Labels and lines shall use bounded collections.

When the visual limit is exceeded:

- The oldest non-active event object shall be deleted.
- Active reference lines shall be preserved where practical.
- Internal event history may remain deeper than visible history.
- Object references shall be removed safely.

---

# 48. Performance Requirements

The engine shall:

- Evaluate only active structural levels.
- Avoid rescanning full historical structure on every bar.
- Reuse Market Structure outputs.
- Reuse ATR and RVOL values from their owning modules.
- Use bounded arrays.
- Avoid duplicate event calculations.
- Avoid object creation when visuals are disabled.
- Perform text formatting only when required.

---

# 49. Error Handling

The engine shall safely handle:

- No active structural reference
- Insufficient structure history
- Neutral structure
- Transition structure
- Duplicate levels
- Equal levels
- Gap bars
- Missing RVOL
- Zero-range candles
- Large volatility spikes
- Same-bar opposing breaches
- Symbol changes
- Timeframe changes
- Object-limit pressure

No edge case shall produce a runtime error.

---

# 50. Testing Requirements

The engine shall be tested across:

- Stocks
- Indices
- Forex
- Cryptocurrency
- Futures
- Commodities

Required timeframes:

- 1 minute
- 5 minute
- 15 minute
- 1 hour
- 4 hour
- Daily
- Weekly

Required conditions:

- Clean bullish continuation
- Clean bearish continuation
- Bullish reversal
- Bearish reversal
- Wick-only sweep
- Close-confirmed breakout
- Failed breakout
- Gap through structure
- Low-volume break
- High-volume break
- Equal High break
- Equal Low break
- Neutral structure
- Transition structure
- Repeated closes beyond the same level

---

# 51. Functional Test Cases

## Test BOS-001 — Bullish BOS

Given:

- Confirmed structure is Bullish.
- An eligible confirmed Swing High exists.
- Price closes above the level and required buffer.

Expected:

- Bullish BOS event is generated once.
- Event level and source pivot are recorded.
- Repeated closes above the same level do not retrigger the event.

---

## Test BOS-002 — Bearish BOS

Given:

- Confirmed structure is Bearish.
- An eligible confirmed Swing Low exists.
- Price closes below the level and required buffer.

Expected:

- Bearish BOS event is generated once.

---

## Test BOS-003 — Bullish CHOCH

Given:

- Confirmed structure is Bearish.
- Price closes above the active bearish protection level.

Expected:

- Bullish CHOCH event is generated.
- Event is classified as reversal context rather than continuation.

---

## Test BOS-004 — Bearish CHOCH

Given:

- Confirmed structure is Bullish.
- Price closes below the active bullish protection level.

Expected:

- Bearish CHOCH event is generated.

---

## Test BOS-005 — Wick Breach Without Close

Given:

- Price trades above a structural high.
- Price closes below the required confirmation level.

Expected:

- Bullish Wick Breach or Rejection event is generated.
- No Bullish BOS or CHOCH event is generated.

---

## Test BOS-006 — Failed Bullish Break

Given:

- Bullish structural break is confirmed.
- Price closes back below the broken level within the failure window.

Expected:

- Original event remains stored.
- Failed Bullish Break event is generated separately.

---

## Test BOS-007 — Failed Bearish Break

Given:

- Bearish structural break is confirmed.
- Price closes back above the broken level within the failure window.

Expected:

- Failed Bearish Break event is generated separately.

---

## Test BOS-008 — Duplicate Suppression

Given:

- Price remains above a previously broken structural high for five bars.

Expected:

- Only one initial break event is generated.
- Classification remains available without repeated alerts.

---

## Test BOS-009 — Neutral Structure Break

Given:

- Market Structure direction is Neutral.
- Price closes above a confirmed structural high.

Expected:

- Event is classified as Bullish Structural Break or approved neutral-context equivalent.
- Event is not falsely classified as BOS or CHOCH.

---

## Test BOS-010 — ATR Buffer

Given:

- Structural high equals 100.
- ATR equals 2.
- Buffer multiplier equals 0.05.
- Confirmation buffer equals 0.10.

Expected:

- Close at 100.05 does not confirm.
- Close above 100.10 confirms according to comparison policy.

---

## Test BOS-011 — Acceptance Confirmation

Given:

- Acceptance Bars equals 2.
- Price closes above structure for one bar only.
- The following bar closes back below.

Expected:

- No accepted breakout event is generated.
- Rejection or failure logic may activate.

---

## Test BOS-012 — Gap Break

Given:

- Price opens above a structural high.
- The bar closes above the level.

Expected:

- Close Break classification follows the approved gap policy.
- No false wick-rejection event is created.

---

## Test BOS-013 — Missing RVOL

Given:

- Structural break conditions are valid.
- RVOL is unavailable.
- Strict RVOL confirmation is disabled.

Expected:

- Structural event still confirms.
- RVOL metadata is unavailable.
- No runtime error occurs.

---

## Test BOS-014 — Strict RVOL Filter

Given:

- Structural break conditions are valid.
- RVOL is below the configured minimum.
- Strict RVOL confirmation is enabled.

Expected:

- Confirmed structural event is withheld or downgraded according to the approved policy.
- Debug state identifies the failed RVOL condition.

---

## Test BOS-015 — Historical Stability

Given:

- A BOS or CHOCH event is confirmed.
- Additional future bars are loaded.

Expected:

- Original event type, bar index, level, and direction remain unchanged.

---

# 52. Acceptance Criteria

The BOS and CHOCH Engine shall be complete when:

- Confirmed structural levels are consumed correctly.
- Wick breaches and close breaks are distinct.
- Bullish and bearish BOS classifications work.
- Bullish and bearish CHOCH classifications work.
- Neutral-context breaks are not misclassified.
- Duplicate events are suppressed.
- Failed Break events are generated separately.
- Event metadata is available.
- Alerts trigger only under intended conditions.
- Object counts remain bounded.
- Missing RVOL is handled safely.
- Confirmed historical events remain stable.
- TradingView compilation succeeds.
- Required test cases pass.

---

# 53. Open Design Decisions

The following items remain unresolved:

1. Final production confirmation mode.
2. Final break-buffer method.
3. Final ATR buffer value.
4. Whether wick breaches are published by this module or only passed internally.
5. Final acceptance rule.
6. Final failure-validation window.
7. Final level-consumption policy.
8. Final neutral-structure event terminology.
9. Final transition-structure classification policy.
10. Whether candle-body strength is included in Version 1.0.
11. Whether strict RVOL confirmation is included in Version 1.0.
12. Final structural-event confidence weights.
13. Whether retest tracking is included in Version 1.0.
14. Final same-bar event-priority order.
15. Gap-break classification policy.
16. Whether Equal High and Equal Low breaks can produce BOS directly.
17. Whether failed events reduce or replace acceptance status.
18. Maximum stored event-history depth.
19. Whether developing intrabar events are visible outside debug mode.
20. Whether BOS and CHOCH labels display confidence values.

---

# 54. Future Enhancements

Possible future enhancements include:

- Retest-quality scoring
- Multi-timeframe BOS and CHOCH
- Major and minor structural-event layers
- Breakout persistence measurement
- Distance-to-structure pre-alerts
- Session-aware structural events
- Volatility-regime-specific confirmation
- Event clustering
- Statistical failure-rate tracking
- Structural-event heatmaps
- User-defined confidence weights
- Separate internal and external structure breaks