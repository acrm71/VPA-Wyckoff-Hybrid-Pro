# Liquidity Engine Specification

---

## Document Information

**Document ID:** SPEC-106

**Version:** 0.1 Draft

**Status:** Draft

**Module:** Liquidity Engine

**Dependencies:**

- Framework
- Settings
- Market Structure Engine
- BOS and CHOCH Engine
- Trend Engine
- Relative Volume Engine
- Dashboard
- Alerts

**Dependent Modules:**

- VPA
- Wyckoff
- Institutional Activity
- Signal Scoring

---

# 1. Purpose

The Liquidity Engine identifies price areas where resting orders and stop orders are likely to cluster and classifies price interactions with those areas.

Its primary responsibilities are to detect and interpret:

- Buy-side liquidity
- Sell-side liquidity
- Equal High liquidity
- Equal Low liquidity
- Swing High liquidity
- Swing Low liquidity
- Liquidity sweeps
- Stop runs
- Rejections
- Accepted liquidity breaks
- Failed auctions
- Liquidity traps
- Cleared and uncleared liquidity levels

The engine shall distinguish temporary liquidity-taking behaviour from genuine structural acceptance.

It shall not independently generate trade entries.

---

# 2. Objectives

The Liquidity Engine shall:

- Use confirmed structural levels as the primary liquidity source.
- Detect potential liquidity pools consistently.
- Distinguish wick sweeps from close-confirmed breaks.
- Track the lifecycle of each liquidity level.
- Prevent duplicate sweep events.
- Distinguish liquidity removal from structural continuation.
- Support Equal High and Equal Low analysis.
- Provide event metadata to Wyckoff, VPA, Institutional, Scoring, Dashboard, and Alerts.
- Avoid intentional historical repainting.
- Maintain bounded object and history usage.
- Operate safely when volume data is unavailable.

---

# 3. Design Principles

## 3.1 Liquidity Is a Location Hypothesis

A liquidity level represents a probable concentration of orders.

It shall not be treated as proof of actual order placement.

## 3.2 Interaction Determines Event Type

A liquidity level alone is not a signal.

Meaning is determined by:

- How price approaches
- Whether price wicks through
- Whether price closes beyond
- Whether price returns inside
- Whether volume expands
- Whether structure confirms acceptance

## 3.3 Sweep Is Not Breakout

A wick beyond a liquidity level with a close back inside shall remain distinct from a confirmed breakout.

## 3.4 Shared Structural Source

The Liquidity Engine shall consume confirmed levels from the Market Structure and BOS/CHOCH engines.

It shall not create an incompatible independent swing system.

## 3.5 Immutable Confirmed Events

Once a confirmed liquidity event is published, its historical record shall not be deleted or rewritten.

Later behaviour shall create additional lifecycle events.

## 3.6 Conservative Classification

Where the evidence is ambiguous, the engine shall use a neutral classification rather than forcing a sweep, breakout, or trap interpretation.

---

# 4. Core Definitions

## 4.1 Buy-Side Liquidity

Buy-side liquidity refers to likely stop-buy and breakout-buy orders located above visible price highs.

Typical sources include:

- Equal Highs
- Confirmed Swing Highs
- Previous session highs
- Range highs
- Repeated resistance levels

## 4.2 Sell-Side Liquidity

Sell-side liquidity refers to likely stop-sell and breakout-sell orders located below visible price lows.

Typical sources include:

- Equal Lows
- Confirmed Swing Lows
- Previous session lows
- Range lows
- Repeated support levels

## 4.3 Liquidity Pool

A Liquidity Pool is a price level or zone where clustered orders are considered likely.

## 4.4 Liquidity Sweep

A Liquidity Sweep occurs when price trades beyond a liquidity level but fails to establish acceptance beyond it.

Typical bullish reversal example:

```text
Price trades below Sell-Side Liquidity
Price closes back above the level
```

Typical bearish reversal example:

```text
Price trades above Buy-Side Liquidity
Price closes back below the level
```

## 4.5 Stop Run

A Stop Run is a liquidity sweep characterised by a decisive move through a level followed by rapid rejection.

The first implementation may treat Stop Run as a high-confidence sweep classification.

## 4.6 Accepted Liquidity Break

An Accepted Liquidity Break occurs when price closes beyond a liquidity level and demonstrates continued acceptance beyond it.

## 4.7 Failed Auction

A Failed Auction occurs when price probes beyond a liquidity level but cannot attract sustained trading beyond it.

## 4.8 Liquidity Trap

A Liquidity Trap occurs when price initially appears to confirm a breakout, attracts participation, and then reverses back through the level.

## 4.9 Cleared Liquidity

A liquidity level is Cleared when price has meaningfully traded through or accepted beyond the level according to the approved lifecycle policy.

## 4.10 Untouched Liquidity

A liquidity level is Untouched when price has not yet interacted with it after creation.

---

# 5. Inputs

## 5.1 Enable Liquidity Engine

Type:

Boolean

Default:

Enabled

When disabled:

- No new liquidity levels shall be registered where avoidable.
- Liquidity events shall not be generated.
- Liquidity visuals shall be hidden.
- Alerts shall be disabled.
- Scoring contribution shall be removed.

---

## 5.2 Liquidity Sources

Type:

Multi-selection or grouped booleans

Initial sources:

- Confirmed Swing Highs
- Confirmed Swing Lows
- Equal Highs
- Equal Lows
- Structural Protection Levels
- Previous Session High
- Previous Session Low
- Range High
- Range Low

Suggested first alpha sources:

- Confirmed Swing Highs
- Confirmed Swing Lows
- Equal Highs
- Equal Lows

Session and range sources may be deferred.

---

## 5.3 Equal-Level Minimum Touches

Type:

Integer

Suggested default:

2

Minimum:

2

Purpose:

Defines the minimum number of sufficiently similar highs or lows required to form an Equal High or Equal Low liquidity pool.

---

## 5.4 Equal-Level Tolerance Method

Type:

Selection

Options:

- Tick
- Percentage
- ATR

Suggested default:

ATR

---

## 5.5 Equal-Level Tolerance

Type:

Float

Suggested provisional default:

```text
0.10 ATR
```

Purpose:

Defines the maximum distance between levels considered part of the same liquidity pool.

---

## 5.6 Liquidity Zone Width

Type:

Float

Purpose:

Allows a liquidity level to be treated as a narrow zone rather than an exact price.

Possible methods:

- Tick width
- Percentage width
- ATR width

Suggested default:

```text
0.05 ATR
```

---

## 5.7 Sweep Confirmation Mode

Type:

Selection

Options:

- Wick Beyond and Close Inside
- Wick Beyond and Body Inside
- Wick Beyond and Next-Bar Confirmation
- Wick Beyond with Volume Confirmation

Suggested production default:

Wick Beyond and Close Inside

---

## 5.8 Minimum Sweep Distance

Type:

Float

Purpose:

Defines the minimum penetration beyond a liquidity level required for a valid sweep.

Suggested normalisation:

ATR fraction

Suggested provisional default:

```text
0.02 ATR
```

---

## 5.9 Maximum Sweep Distance

Type:

Float

Purpose:

Prevents very large accepted moves from being misclassified as sweeps.

A large penetration with sustained close beyond the level is more likely to represent acceptance or breakout.

Suggested provisional default:

```text
0.50 ATR
```

---

## 5.10 Sweep Confirmation Window

Type:

Integer

Suggested default:

1

Purpose:

Defines the number of bars allowed for price to return inside the liquidity level after the initial breach.

---

## 5.11 Acceptance Bars

Type:

Integer

Suggested default:

2

Purpose:

Defines the number of confirmed closes beyond the level required to classify acceptance.

---

## 5.12 Trap Validation Window

Type:

Integer

Suggested default:

3

Purpose:

Defines the number of bars after apparent acceptance during which a rapid return through the level may produce a Liquidity Trap event.

---

## 5.13 Require RVOL for High-Confidence Sweep

Type:

Boolean

Suggested default:

Disabled

Purpose:

When enabled, elevated RVOL is required for the highest sweep-confidence classification.

Basic sweep detection shall remain operational without volume.

---

## 5.14 Minimum Sweep RVOL

Type:

Float

Suggested default:

1.20

---

## 5.15 Maximum Active Liquidity Levels

Type:

Integer

Suggested default:

30

Purpose:

Limits the number of concurrently tracked liquidity levels.

---

## 5.16 Liquidity Level Expiry Bars

Type:

Integer

Suggested default:

500

Purpose:

Allows old, untouched levels to expire after a configurable number of bars.

A value of zero may represent no time-based expiry.

---

## 5.17 Show Liquidity Levels

Type:

Boolean

Suggested default:

Enabled

---

## 5.18 Show Liquidity Zones

Type:

Boolean

Suggested default:

Disabled

---

## 5.19 Show Sweep Markers

Type:

Boolean

Suggested default:

Enabled

---

## 5.20 Show Accepted Breaks

Type:

Boolean

Suggested default:

Enabled

---

## 5.21 Show Cleared Levels

Type:

Boolean

Suggested default:

Disabled

---

## 5.22 Enable Liquidity Alerts

Type:

Boolean

Suggested default:

Enabled

---

# 6. Input Validation

The engine shall validate that:

- Minimum Touches is at least 2.
- Tolerances are non-negative.
- Minimum Sweep Distance is non-negative.
- Maximum Sweep Distance is greater than or equal to Minimum Sweep Distance.
- Sweep Confirmation Window is non-negative.
- Acceptance Bars is at least 1.
- Trap Validation Window is non-negative.
- Maximum Active Levels is positive.
- Expiry Bars is non-negative.
- Minimum RVOL is non-negative.

Invalid input shall not cause runtime failure.

---

# 7. Liquidity-Level Sources

## 7.1 Swing High Liquidity

Each eligible confirmed Swing High may create a Buy-Side Liquidity level.

## 7.2 Swing Low Liquidity

Each eligible confirmed Swing Low may create a Sell-Side Liquidity level.

## 7.3 Equal High Liquidity

Confirmed Equal High clusters shall create higher-priority Buy-Side Liquidity pools.

## 7.4 Equal Low Liquidity

Confirmed Equal Low clusters shall create higher-priority Sell-Side Liquidity pools.

## 7.5 Protection-Level Liquidity

Bullish and bearish protection levels may also represent structurally important liquidity.

This source may be implemented after core swing liquidity is validated.

## 7.6 Session Liquidity

Previous session highs and lows may be supported in a future session-aware release.

## 7.7 Range Liquidity

Trading-range highs and lows may be supplied by the Wyckoff or future range-detection module.

---

# 8. Liquidity Level Registration

When a valid source level is received, the engine shall store:

- Liquidity level price
- Zone upper boundary
- Zone lower boundary
- Liquidity side
- Source type
- Source bar index
- Creation bar index
- Number of contributing touches
- Priority
- Lifecycle state
- Last interaction bar
- Sweep count
- Break status
- Acceptance status
- Expiry status

Duplicate levels within tolerance shall be merged where appropriate.

---

# 9. Liquidity Side

Suggested internal values:

```text
+1 = Buy-Side Liquidity
-1 = Sell-Side Liquidity
 0 = Unavailable
```

Buy-side liquidity lies above price at creation.

Sell-side liquidity lies below price at creation.

---

# 10. Liquidity Priority

The engine may assign a priority score to each level.

Suggested range:

```text
0 to 100
```

Possible factors:

| Factor | Suggested Weight |
|---|---:|
| Equal-level cluster | 25 |
| Number of touches | 20 |
| Structural significance | 20 |
| Level recency | 10 |
| Distance from current price | 10 |
| Higher-timeframe relevance | 15 |

Priority scoring is optional for the first alpha.

At minimum, Equal High and Equal Low levels should rank above isolated minor pivots.

---

# 11. Level Merging

New liquidity levels may be merged when:

- They are on the same liquidity side.
- Their price difference is within tolerance.
- They are structurally compatible.
- Neither level has been permanently invalidated.

When merging:

- Touch count shall increase.
- The representative level may use average, median, or most recent price.
- Zone boundaries shall expand only within approved limits.
- The earliest source bar may be retained for provenance.
- Priority may increase.

The final representative-price method remains an open decision.

---

# 12. Level Lifecycle

Each liquidity level shall have one active lifecycle state.

Suggested states:

1. Untouched
2. Approaching
3. Touched
4. Swept
5. Rejected
6. Broken
7. Accepted
8. Trapped
9. Cleared
10. Expired
11. Invalidated

---

# 13. Untouched State

A level is Untouched when no qualifying price interaction has occurred after registration.

Untouched levels may remain active until:

- Swept
- Broken
- Accepted
- Expired
- Replaced
- Invalidated

---

# 14. Approach State

A level may enter Approaching when price moves within a configurable distance.

Approach state may support:

- Dashboard context
- Pre-alerts
- Visual emphasis
- Institutional analysis

Approach detection is optional for the first alpha.

---

# 15. Touch State

A level is Touched when price enters the defined liquidity zone without materially breaching it.

Touch alone shall not create a sweep or breakout event.

---

# 16. Buy-Side Liquidity Sweep

A Buy-Side Liquidity Sweep shall generally require:

- An active Buy-Side Liquidity level exists.
- Price trades above the level or zone.
- Penetration satisfies the minimum sweep distance.
- Price closes back below the level or inside the original range within the confirmation window.
- Acceptance beyond the level is not established.

Interpretation:

```text
Liquidity above highs was taken, followed by rejection.
```

This event is directionally bearish in immediate reaction terms but shall remain context-dependent.

---

# 17. Sell-Side Liquidity Sweep

A Sell-Side Liquidity Sweep shall generally require:

- An active Sell-Side Liquidity level exists.
- Price trades below the level or zone.
- Penetration satisfies the minimum sweep distance.
- Price closes back above the level or inside the original range within the confirmation window.
- Acceptance below the level is not established.

Interpretation:

```text
Liquidity below lows was taken, followed by rejection.
```

This event is directionally bullish in immediate reaction terms but shall remain context-dependent.

---

# 18. Wick-Only Sweep

A wick-only sweep occurs when:

- The wick exceeds the liquidity level.
- The candle body remains on the original side.
- The close returns inside.

This is the standard sweep pattern.

Wick-only sweeps shall remain distinct from confirmed structural breaks.

---

# 19. Delayed Sweep Confirmation

A delayed sweep may occur when:

1. Price breaches the liquidity level.
2. The initial bar closes marginally beyond or near the level.
3. A following bar closes back inside within the Sweep Confirmation Window.

Delayed confirmation shall be optional and conservative.

The original breach bar and confirmation bar shall both be retained in event metadata.

---

# 20. Accepted Buy-Side Break

An Accepted Buy-Side Break shall require:

- Price closes above Buy-Side Liquidity.
- Required acceptance conditions are satisfied.
- Price does not return below the level within the immediate confirmation period.
- BOS/CHOCH context supports or permits breakout classification.

This event indicates that buy-side liquidity was removed and price accepted above the level.

---

# 21. Accepted Sell-Side Break

An Accepted Sell-Side Break shall require:

- Price closes below Sell-Side Liquidity.
- Required acceptance conditions are satisfied.
- Price does not return above the level within the immediate confirmation period.

---

# 22. Liquidity Rejection

A Liquidity Rejection occurs when price:

- Touches or marginally breaches the level.
- Fails to establish acceptance.
- Closes decisively back on the original side.

A Rejection may be weaker than a Sweep when the minimum penetration distance is not satisfied.

---

# 23. Liquidity Trap

## 23.1 Bull Trap

A Bull Trap may occur when:

1. Price closes above Buy-Side Liquidity.
2. The move initially appears accepted or breakout-valid.
3. Price closes back below the level within the Trap Validation Window.
4. Follow-through fails.

## 23.2 Bear Trap

A Bear Trap may occur when:

1. Price closes below Sell-Side Liquidity.
2. The move initially appears accepted or breakout-valid.
3. Price closes back above the level within the Trap Validation Window.
4. Follow-through fails.

Trap events shall not erase the original breakout event.

They shall be published separately.

---

# 24. Failed Auction

A Failed Auction may be classified when:

- Price explores beyond a liquidity boundary.
- Trading cannot be sustained beyond that boundary.
- Price returns rapidly toward prior value.
- Volume and spread behaviour support rejection where available.

The first implementation may map Failed Auction to a high-confidence Sweep or Trap event.

A separate event class may be introduced later.

---

# 25. Sweep Versus Structural Break

The engine shall use the following conceptual distinction:

## Sweep

```text
Price breaches level
No sustained close beyond
Price returns inside
```

## Structural Break

```text
Price closes beyond level
Confirmation rules pass
Acceptance may develop
```

## Trap

```text
Price initially confirms beyond level
Price rapidly returns through level
```

The BOS/CHOCH Engine remains authoritative for structural-break classification.

The Liquidity Engine adds liquidity interpretation.

---

# 26. Duplicate Sweep Suppression

A liquidity level shall not generate repeated identical sweep events without a meaningful reset.

After a confirmed sweep:

- The level may remain active if not fully cleared.
- Additional interactions may be recorded as retests.
- A second sweep may require price to move away by a minimum reset distance.
- Repeated intrabar breaches on the same bar shall count as one event.

The final reset rule remains an open decision.

---

# 27. Level Consumption

A level may be considered consumed when:

- Acceptance beyond the level is confirmed.
- The level has been fully cleared according to the approved policy.
- A replacement liquidity pool supersedes it.
- It expires.
- Its maximum sweep count is exceeded.

A wick-only sweep shall not automatically consume the level.

This allows repeated liquidity tests to remain visible where appropriate.

---

# 28. Cleared-Level Policy

A level may be marked Cleared when:

- Price accepts beyond it.
- Structure confirms a valid breakout.
- The liquidity pool is no longer considered relevant.

Cleared levels shall not generate new primary sweep alerts unless explicitly reactivated.

Historical visual display may be optional.

---

# 29. Expiry Policy

A liquidity level may expire when:

- It remains untouched beyond the configured number of bars.
- It is replaced by a more recent nearby level.
- It becomes structurally irrelevant.
- It exceeds the maximum active-level count.

Expiry shall not be treated as a liquidity event.

---

# 30. Equal High Handling

Equal High liquidity shall retain:

- Representative price
- Zone width
- Touch count
- First touch bar
- Most recent touch bar
- Individual source pivots where feasible

A sweep above Equal Highs shall generally be considered more significant than a sweep above an isolated minor Swing High.

A confirmed close and acceptance above Equal Highs may indicate:

- Breakout
- Buy-side liquidity removal
- Bullish BOS
- Sign of Strength
- Range expansion

---

# 31. Equal Low Handling

Equal Low liquidity shall use equivalent logic.

A sweep below Equal Lows may indicate:

- Sell-side liquidity removal
- Spring candidate
- Stop run
- Failed bearish auction

Acceptance below Equal Lows may indicate:

- Bearish breakout
- Bearish BOS
- Sign of Weakness
- Range breakdown

---

# 32. Trend Integration

Trend shall provide context but shall not determine whether a sweep occurred.

Examples:

| Liquidity Event | Trend Context | Possible Interpretation |
|---|---|---|
| Sell-Side Sweep | Bullish Trend | Pullback completion |
| Sell-Side Sweep | Bearish Trend | Countertrend reaction |
| Buy-Side Sweep | Bearish Trend | Continuation setup |
| Buy-Side Sweep | Bullish Trend | Distribution or temporary rejection |
| Accepted Buy-Side Break | Bullish Trend | Continuation confirmation |
| Accepted Sell-Side Break | Bearish Trend | Continuation confirmation |

Trend alignment may affect confidence and scoring.

---

# 33. Market Structure Integration

The Market Structure Engine supplies:

- Swing Highs
- Swing Lows
- Equal Highs
- Equal Lows
- Protection levels
- Structural direction
- Pivot metadata

The Liquidity Engine shall not alter confirmed swing classifications.

---

# 34. BOS and CHOCH Integration

The BOS/CHOCH Engine supplies:

- Wick breaches
- Close breaks
- BOS events
- CHOCH events
- Rejections
- Failed breaks
- Acceptance status
- Reference-level metadata

The Liquidity Engine shall combine these with liquidity-source context.

Examples:

```text
Bullish Wick Breach
+
Equal High Source
+
Close Back Below
=
Buy-Side Liquidity Sweep
```

```text
Bullish BOS
+
Equal High Source
+
Acceptance
=
Accepted Buy-Side Break
```

---

# 35. Relative Volume Integration

Relative Volume may influence liquidity-event confidence.

Possible interpretation:

- High RVOL sweep may indicate aggressive stop-taking or absorption.
- Extreme RVOL sweep may indicate climax or institutional transfer.
- Low RVOL sweep may be less reliable.
- High RVOL accepted break may support genuine breakout.
- Extreme RVOL failed breakout may support trap interpretation.
- Missing RVOL shall not prevent basic liquidity classification.

RVOL shall remain optional unless strict filtering is enabled.

---

# 36. VPA Integration

The VPA Engine may combine liquidity events with:

- Candle spread
- Close location
- Wick size
- Relative volume
- Effort versus result
- No-demand or no-supply conditions
- Stopping volume
- Absorption

Examples:

```text
Sell-Side Sweep
+
High RVOL
+
Narrow Spread
+
Close Near High
=
Potential Bullish Absorption
```

```text
Buy-Side Sweep
+
Extreme RVOL
+
Close Near Low
=
Potential Distribution or Upthrust
```

---

# 37. Wyckoff Integration

Liquidity events shall support Wyckoff classification.

Possible relationships:

| Liquidity Event | Wyckoff Candidate |
|---|---|
| Sell-Side Sweep of Range Low | Spring |
| Buy-Side Sweep of Range High | Upthrust |
| Accepted Buy-Side Break | Sign of Strength |
| Accepted Sell-Side Break | Sign of Weakness |
| Low-volume retest after Spring | Secondary Test |
| Low-volume retest after SOS | Last Point of Support |
| Low-volume retest after SOW | Last Point of Supply |

The Wyckoff Engine shall apply additional phase and range conditions.

---

# 38. Institutional Activity Integration

Institutional Activity may be inferred more strongly when liquidity events occur with:

- Extreme RVOL
- Narrow spread despite high volume
- Large rejection wick
- Strong close back inside
- Failed BOS
- CHOCH
- Major structural level
- Repeated liquidity tests

The Institutional Engine shall perform the final interpretation.

---

# 39. Scoring Integration

Possible scoring effects:

- Sell-Side Sweep may support bullish reversal evidence.
- Buy-Side Sweep may support bearish reversal evidence.
- Accepted Buy-Side Break may support bullish continuation.
- Accepted Sell-Side Break may support bearish continuation.
- Bull Trap may support bearish evidence.
- Bear Trap may support bullish evidence.
- High-priority Equal High or Equal Low events may receive greater weight.
- Events aligned with trend and structure may receive higher confidence.
- Low-quality isolated-pivot sweeps may receive reduced weight.

The final score weights belong to the Signal Scoring specification.

---

# 40. Liquidity Event Confidence

The engine may expose a confidence score from:

```text
0 to 100
```

Possible components:

| Component | Suggested Weight |
|---|---:|
| Level priority | 20 |
| Penetration quality | 15 |
| Close rejection quality | 20 |
| RVOL context | 15 |
| Structural context | 15 |
| Follow-through | 15 |

These values remain provisional.

Confidence shall represent event quality, not trade probability.

---

# 41. Outputs

The Liquidity Engine shall expose:

- Engine enabled status
- Nearest Buy-Side Liquidity level
- Nearest Sell-Side Liquidity level
- Distance to nearest Buy-Side Liquidity
- Distance to nearest Sell-Side Liquidity
- Active liquidity-level count
- Last liquidity event type
- Last liquidity event direction
- Last event level
- Last event source
- Last event bar index
- Last event confirmation bar
- Buy-Side Sweep event
- Sell-Side Sweep event
- Accepted Buy-Side Break event
- Accepted Sell-Side Break event
- Bull Trap event
- Bear Trap event
- Liquidity Rejection event
- Level-cleared event
- Event confidence
- Level priority
- Volume context
- Sufficient-history status

---

# 42. Suggested Enumerations

## Liquidity Source

```text
0 = Unavailable
1 = Swing High
2 = Swing Low
3 = Equal High
4 = Equal Low
5 = Bullish Protection
6 = Bearish Protection
7 = Previous Session High
8 = Previous Session Low
9 = Range High
10 = Range Low
```

## Liquidity Event Type

```text
0 = None
1 = Buy-Side Touch
2 = Sell-Side Touch
3 = Buy-Side Sweep
4 = Sell-Side Sweep
5 = Buy-Side Rejection
6 = Sell-Side Rejection
7 = Accepted Buy-Side Break
8 = Accepted Sell-Side Break
9 = Bull Trap
10 = Bear Trap
11 = Buy-Side Liquidity Cleared
12 = Sell-Side Liquidity Cleared
13 = Failed Auction High
14 = Failed Auction Low
```

## Lifecycle State

```text
0 = Untouched
1 = Approaching
2 = Touched
3 = Swept
4 = Rejected
5 = Broken
6 = Accepted
7 = Trapped
8 = Cleared
9 = Expired
10 = Invalidated
```

---

# 43. Dashboard Integration

The dashboard shall be able to display:

- Nearest Buy-Side Liquidity
- Nearest Sell-Side Liquidity
- Distance to each level
- Last liquidity event
- Event confidence
- Level source
- Acceptance or rejection status
- RVOL context

Suggested output:

```text
Liquidity: Sell-Side Sweep
Level: 1.24820
Source: Equal Low
Confidence: 81
```

Nearest-level output:

```text
BSL: 1.26250
SSL: 1.24820
```

When no valid level exists:

```text
Liquidity: N/A
```

---

# 44. Chart Visualisation

Optional visual elements may include:

- Buy-Side Liquidity lines
- Sell-Side Liquidity lines
- Equal High zones
- Equal Low zones
- Sweep markers
- Accepted-break markers
- Trap markers
- Cleared-level styling
- Nearest-liquidity emphasis

Default visuals shall avoid excessive clutter.

Minor isolated swing levels may be hidden by default while Equal High and Equal Low pools remain visible.

---

# 45. Visual Conventions

Suggested abbreviations:

```text
BSL = Buy-Side Liquidity
SSL = Sell-Side Liquidity
BS Sweep = Buy-Side Sweep
SS Sweep = Sell-Side Sweep
BT = Bull Trap
BeT = Bear Trap
```

Exact labels shall be finalised in the Dashboard and Settings specifications.

---

# 46. Alert Integration

The engine shall support alerts for:

- Buy-Side Liquidity approached
- Sell-Side Liquidity approached
- Buy-Side Sweep confirmed
- Sell-Side Sweep confirmed
- Accepted Buy-Side Break
- Accepted Sell-Side Break
- Bull Trap
- Bear Trap
- High-priority liquidity level cleared
- Equal High liquidity taken
- Equal Low liquidity taken

Approach alerts shall be disabled by default.

Confirmed alerts shall use bar-close behaviour unless explicitly configured otherwise.

Alert messages shall include:

- Symbol
- Chart timeframe
- Event type
- Liquidity side
- Level price
- Source type
- Event confidence
- RVOL where available
- Structural context
- Timestamp where supported

Example:

```text
GBPUSD 15m — Sell-Side Liquidity Sweep below Equal Low 1.27340 — Confidence 84
```

---

# 47. Repainting Policy

The engine shall not intentionally repaint confirmed liquidity events.

Requirements:

- Liquidity sources shall use confirmed structural levels.
- No lookahead shall be used.
- Sweep confirmation shall follow closed-bar rules by default.
- Confirmed event history shall remain immutable.
- Later acceptance, trap, or failure events shall be added separately.
- Developing intrabar sweeps shall remain provisional.
- Level merging shall not rewrite previously published event metadata.

---

# 48. Real-Time Behaviour

During a developing bar:

- A liquidity breach may appear and disappear.
- Penetration distance may change.
- Sweep qualification may remain provisional.
- Acceptance cannot be final before required closes complete.
- Intrabar alerts may produce more false positives and shall not be the production default.

The dashboard shall distinguish developing from confirmed events where provisional states are displayed.

---

# 49. Same-Bar Opposing Events

A highly volatile bar may breach both Buy-Side and Sell-Side Liquidity.

The engine shall apply deterministic handling.

Possible policy:

- Evaluate both interactions.
- Retain both raw breach events.
- Classify only the side supported by the final close and context as the primary event.
- Mark the bar as Two-Sided Liquidity Expansion where neither side clearly dominates.

The final policy remains an open design decision.

---

# 50. Gap Handling

A gap beyond liquidity shall be treated separately from a wick sweep.

A gap may produce:

- Immediate accepted break
- Gap rejection
- Gap trap
- Unclassified gap interaction

The engine shall use:

- Open
- High
- Low
- Close
- BOS/CHOCH event state
- Acceptance behaviour

A price gap shall not be falsely classified as a wick-based stop run.

---

# 51. Level Distance

The engine may calculate distance from current price to active liquidity.

Possible forms:

- Absolute price
- Percentage
- ATR-normalised distance
- Tick distance

Suggested internal form:

```text
ATR-normalised distance
```

Dashboard display may use price and percentage.

---

# 52. Object Management

Liquidity lines, zones, and labels shall use bounded collections.

When limits are reached:

- Expired and cleared objects shall be deleted first.
- Old inactive objects shall be deleted before active high-priority levels.
- Internal references shall be removed safely.
- Active nearest-liquidity levels shall be preserved where practical.

---

# 53. Performance Requirements

The engine shall:

- Evaluate only active liquidity levels.
- Avoid full-history rescanning.
- Reuse structural and BOS/CHOCH outputs.
- Reuse ATR and RVOL calculations.
- Use bounded arrays.
- Merge nearby levels efficiently.
- Avoid drawing objects when visuals are disabled.
- Update nearest-level calculations efficiently.
- Avoid repeated event string construction on historical bars.

---

# 54. Error Handling

The engine shall safely handle:

- No active liquidity levels
- Duplicate levels
- Overlapping zones
- Consecutive Equal Highs
- Consecutive Equal Lows
- Missing RVOL
- Zero ATR
- Gap bars
- Illiquid markets
- Large volatility spikes
- Same-bar two-sided breaches
- Insufficient structural history
- Symbol changes
- Timeframe changes
- Object-limit pressure

No edge case shall cause a runtime error.

---

# 55. Testing Requirements

The Liquidity Engine shall be tested across:

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

- Isolated Swing High
- Isolated Swing Low
- Equal High cluster
- Equal Low cluster
- Wick-only sweep
- Delayed sweep confirmation
- Accepted breakout
- Bull Trap
- Bear Trap
- Repeated sweep
- Level expiry
- Gap through liquidity
- Two-sided volatile bar
- Missing volume
- Low-volume sweep
- High-volume sweep
- Extreme-volume failed breakout

---

# 56. Functional Test Cases

## Test LIQ-001 — Buy-Side Sweep

Given:

- An active Buy-Side Liquidity level exists.
- Price trades above the level.
- Minimum penetration is satisfied.
- Price closes back below the level.

Expected:

- Buy-Side Sweep event is confirmed.
- No accepted break is generated.
- Event metadata records the source level.

---

## Test LIQ-002 — Sell-Side Sweep

Given:

- An active Sell-Side Liquidity level exists.
- Price trades below the level.
- Price closes back above it.

Expected:

- Sell-Side Sweep event is confirmed.

---

## Test LIQ-003 — Accepted Buy-Side Break

Given:

- Price closes above Buy-Side Liquidity.
- Acceptance Bars requirement is satisfied.
- Price remains above the level.

Expected:

- Accepted Buy-Side Break is generated.
- Level is marked Cleared or Accepted according to lifecycle policy.

---

## Test LIQ-004 — Accepted Sell-Side Break

Given:

- Price closes below Sell-Side Liquidity.
- Acceptance conditions are satisfied.

Expected:

- Accepted Sell-Side Break is generated.

---

## Test LIQ-005 — Equal High Pool

Given:

- Two confirmed Swing Highs occur within tolerance.

Expected:

- One Equal High liquidity pool is registered.
- Touch count equals at least two.
- Priority exceeds an isolated minor Swing High.

---

## Test LIQ-006 — Equal Low Pool

Given:

- Two confirmed Swing Lows occur within tolerance.

Expected:

- One Equal Low liquidity pool is registered.

---

## Test LIQ-007 — Bull Trap

Given:

- Price initially confirms above Buy-Side Liquidity.
- Price closes back below the level within the Trap Validation Window.

Expected:

- Original break event remains recorded.
- Bull Trap event is generated separately.

---

## Test LIQ-008 — Bear Trap

Given:

- Price initially confirms below Sell-Side Liquidity.
- Price closes back above the level within the validation window.

Expected:

- Bear Trap event is generated.

---

## Test LIQ-009 — Duplicate Sweep Suppression

Given:

- The same liquidity level is breached repeatedly without a reset.

Expected:

- Only one primary sweep event is generated.
- Repeated intrabar interactions do not create duplicate alerts.

---

## Test LIQ-010 — Wick Without Minimum Penetration

Given:

- Price exceeds the level by less than the minimum sweep distance.
- Price closes back inside.

Expected:

- Event may be classified as Touch or Rejection.
- Full Sweep classification is withheld.

---

## Test LIQ-011 — Large Accepted Move

Given:

- Price exceeds a liquidity level by more than the maximum sweep distance.
- Price closes and remains beyond the level.

Expected:

- Event is not classified as a sweep.
- Accepted Break logic is evaluated.

---

## Test LIQ-012 — Missing RVOL

Given:

- Sweep price conditions are valid.
- RVOL is unavailable.

Expected:

- Basic Sweep event still confirms.
- Volume metadata is unavailable.
- No runtime error occurs.

---

## Test LIQ-013 — Level Expiry

Given:

- A liquidity level remains untouched beyond the configured expiry period.

Expected:

- Level state becomes Expired.
- No sweep or break event is generated.
- Visual object may be removed.

---

## Test LIQ-014 — Level Merging

Given:

- A new liquidity level is created within tolerance of an existing compatible level.

Expected:

- Levels merge.
- Touch count increases.
- No duplicate nearby pool remains active.

---

## Test LIQ-015 — Gap Above Buy-Side Liquidity

Given:

- Price opens above Buy-Side Liquidity without trading through the level.
- Price closes above it.

Expected:

- Gap policy evaluates Accepted Break.
- Event is not falsely classified as Wick Sweep.

---

## Test LIQ-016 — Historical Stability

Given:

- A liquidity sweep has been confirmed.
- Additional bars are loaded.

Expected:

- Event type, level, source, and confirmation bar remain unchanged.

---

# 57. Acceptance Criteria

The Liquidity Engine shall be complete when:

- Swing and equal-level liquidity sources register correctly.
- Buy-Side and Sell-Side Liquidity remain distinct.
- Wick sweeps and accepted breaks are classified separately.
- Equal High and Equal Low pools merge correctly.
- Duplicate sweep events are suppressed.
- Trap events are added without rewriting original breaks.
- Level lifecycle states operate correctly.
- Expiry and clearing policies work safely.
- Missing volume does not break classification.
- Dashboard outputs are available.
- Alerts trigger only under intended conditions.
- Object counts remain bounded.
- Confirmed historical events remain stable.
- TradingView compilation succeeds.
- Required tests pass.

---

# 58. Open Design Decisions

The following items remain unresolved:

1. Final liquidity sources included in Version 1.0.
2. Final Equal High and Equal Low tolerance method.
3. Final representative price for merged pools.
4. Final zone-width method.
5. Final minimum and maximum sweep distances.
6. Final sweep-confirmation window.
7. Final acceptance-bar requirement.
8. Final trap-validation window.
9. Final duplicate-sweep reset rule.
10. Final level-consumption policy.
11. Whether swept levels remain active.
12. Maximum allowed sweep count per level.
13. Final expiry policy.
14. Whether structural protection levels create liquidity pools.
15. Whether session highs and lows are included in Version 1.0.
16. Whether range highs and lows are supplied by Wyckoff.
17. Final liquidity-priority weights.
18. Final liquidity-event confidence weights.
19. Final same-bar opposing-event policy.
20. Final gap-handling policy.
21. Whether Approach alerts are included in Version 1.0.
22. Whether Liquidity Trap is a dedicated event or a Failed Break subtype.
23. Whether Failed Auction is separate from Sweep.
24. Whether cleared levels remain visible by default.
25. Maximum stored liquidity-history depth.

---

# 59. Future Enhancements

Possible future enhancements include:

- Previous day high and low liquidity
- Session high and low liquidity
- Weekly and monthly liquidity
- Multi-timeframe liquidity maps
- Internal and external liquidity layers
- Liquidity void detection
- Fair value gap integration
- Volume-profile liquidity zones
- Time-at-level analysis
- Order-block proximity
- Liquidity heatmaps
- Statistical sweep follow-through
- Session-specific liquidity presets
- Dynamic liquidity priority