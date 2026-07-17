# Wyckoff Engine Specification

---

## Document Information

**Document ID:** SPEC-108

**Version:** 0.1 Draft

**Status:** Draft

**Module:** Wyckoff Engine

**Dependencies:**

- Framework
- Settings
- Trend Engine
- Relative Volume Engine
- Market Structure Engine
- BOS and CHOCH Engine
- Liquidity Engine
- Volume Price Analysis Engine
- Dashboard
- Alerts

**Dependent Modules:**

- Institutional Activity
- Signal Scoring

---

# 1. Purpose

The Wyckoff Engine evaluates market behaviour using Wyckoff-style trading-range, event-sequence, phase, effort-versus-result, and supply-demand analysis.

Its primary responsibilities are to identify and track:

- Trading ranges
- Accumulation context
- Distribution context
- Reaccumulation context
- Redistribution context
- Phase A
- Phase B
- Phase C
- Phase D
- Phase E
- Preliminary Support
- Preliminary Supply
- Selling Climax
- Buying Climax
- Automatic Rally
- Automatic Reaction
- Secondary Test
- Spring
- Upthrust
- Test after Spring
- Test after Upthrust
- Sign of Strength
- Sign of Weakness
- Last Point of Support
- Last Point of Supply
- Range breakout
- Range failure
- Range invalidation
- Phase confidence
- Schematic confidence

The engine shall interpret ordered market evidence rather than identify isolated candle patterns.

It shall not independently generate trade entries.

---

# 2. Objectives

The Wyckoff Engine shall:

- Detect credible trading-range candidates.
- Track range boundaries and internal behaviour.
- Evaluate Wyckoff events as ordered sequences.
- Avoid assigning phases from one isolated bar.
- Distinguish accumulation from distribution conservatively.
- Support reaccumulation and redistribution context.
- Consume shared RVOL, structure, liquidity, and VPA outputs.
- Distinguish range tests from accepted breakouts.
- Track spring, upthrust, SOS, SOW, LPS, and LPSY candidates.
- Produce phase and schematic confidence values.
- Handle ambiguous and incomplete schematics safely.
- Avoid intentional historical repainting.
- Preserve confirmed event history.
- Limit active range and event storage.
- Provide structured outputs to Institutional Activity, Scoring, Dashboard, and Alerts.

---

# 3. Design Principles

## 3.1 Sequence Before Label

Wyckoff events derive meaning from order and context.

The engine shall prefer:

```text
Range Context
+
Prior Event
+
Current Event
+
Follow-Through
```

over isolated single-bar labelling.

## 3.2 Probabilistic Classification

Wyckoff schematics are interpretive models.

The engine shall publish:

- Candidate events
- Confirmed events
- Confidence values
- Ambiguous states

It shall not present uncertain interpretations as facts.

## 3.3 Shared Analytical Sources

The engine shall consume:

- Trend from the Trend Engine
- Swings and structure from Market Structure
- Structural breaks from BOS/CHOCH
- Liquidity events from the Liquidity Engine
- Volume behaviour from RVOL
- Candle interpretation from VPA

It shall not duplicate those engines.

## 3.4 Trading Range as the Primary Context

Most Wyckoff phase and event classifications require an active or recently completed trading range.

Events outside a valid range shall not be force-labelled as range events.

## 3.5 Phase State Is Persistent

A phase shall be treated as a state that evolves over multiple bars.

It shall not oscillate freely on minor price changes.

## 3.6 Immutable Confirmed Events

Confirmed historical events shall remain recorded.

Later information may:

- Confirm an interpretation
- Lower current schematic confidence
- Invalidate the active range
- Produce a failure event

It shall not erase the original confirmed observation.

## 3.7 Conservative Ambiguity Handling

Where both accumulation and distribution remain plausible, the engine shall publish:

```text
Undetermined Trading Range
```

rather than forcing a directional schematic.

---

# 4. Core Definitions

## 4.1 Trading Range

A Trading Range is a bounded price region characterised by repeated interaction with an upper and lower boundary and insufficient directional acceptance outside those boundaries.

## 4.2 Range High

The active upper boundary of a trading range.

Possible sources include:

- Buying Climax
- Automatic Rally high
- Confirmed structural high
- Equal High cluster
- Repeated rejection zone

## 4.3 Range Low

The active lower boundary of a trading range.

Possible sources include:

- Selling Climax
- Automatic Reaction low
- Confirmed structural low
- Equal Low cluster
- Repeated rejection zone

## 4.4 Accumulation

Accumulation is a trading-range context in which supply appears to be absorbed before potential markup.

This is an interpretive classification, not proof of institutional accumulation.

## 4.5 Distribution

Distribution is a trading-range context in which demand appears to be absorbed or supply appears to emerge before potential markdown.

This is an interpretive classification, not proof of institutional distribution.

## 4.6 Reaccumulation

Reaccumulation is a trading range occurring within a broader bullish context before potential continuation.

## 4.7 Redistribution

Redistribution is a trading range occurring within a broader bearish context before potential continuation.

## 4.8 Preliminary Support

Preliminary Support is an early indication that meaningful demand is appearing during a decline.

## 4.9 Preliminary Supply

Preliminary Supply is an early indication that meaningful supply is appearing during an advance.

## 4.10 Selling Climax

A Selling Climax is a potential exhaustion event characterised by unusually high selling effort, wide spread or volatility, and evidence of rejection or reduced downside result.

## 4.11 Buying Climax

A Buying Climax is a potential exhaustion event characterised by unusually high buying effort, wide spread or volatility, and evidence of rejection or reduced upside result.

## 4.12 Automatic Rally

An Automatic Rally is the reaction upward following a Selling Climax or major stopping action.

## 4.13 Automatic Reaction

An Automatic Reaction is the reaction downward following a Buying Climax or major stopping action.

## 4.14 Secondary Test

A Secondary Test is a revisit toward a prior climax or range boundary to assess remaining supply or demand.

## 4.15 Spring

A Spring is a temporary move below range support or Sell-Side Liquidity followed by rejection and return into the range.

## 4.16 Upthrust

An Upthrust is a temporary move above range resistance or Buy-Side Liquidity followed by rejection and return into the range.

## 4.17 Test after Spring

A Test after Spring is a later low-volume or reduced-effort revisit that holds above or near the Spring area.

## 4.18 Test after Upthrust

A Test after Upthrust is a later reduced-demand revisit that fails to regain acceptance above the Upthrust area.

## 4.19 Sign of Strength

A Sign of Strength is a meaningful bullish expansion away from the range, normally supported by spread, volume, structure, and acceptance.

## 4.20 Sign of Weakness

A Sign of Weakness is a meaningful bearish expansion away from the range, normally supported by spread, volume, structure, and acceptance.

## 4.21 Last Point of Support

A Last Point of Support is a successful pullback or test after strength in an accumulation or reaccumulation context.

## 4.22 Last Point of Supply

A Last Point of Supply is a failed rally or test after weakness in a distribution or redistribution context.

---

# 5. Wyckoff Phase Definitions

## 5.1 Phase A

Phase A represents the potential stopping of the prior trend.

Possible evidence includes:

- Preliminary Support or Preliminary Supply
- Selling Climax or Buying Climax
- Automatic Rally or Automatic Reaction
- Initial Secondary Test
- Trend weakening
- High-volume rejection
- Structural transition

## 5.2 Phase B

Phase B represents cause-building and range development.

Possible evidence includes:

- Repeated tests of range boundaries
- Internal swings
- Churn
- Absorption
- Equal Highs or Equal Lows
- Reduced directional progress
- Multiple liquidity interactions
- No sustained acceptance outside the range

## 5.3 Phase C

Phase C represents a terminal test or deceptive move near a range boundary.

Possible accumulation evidence:

- Spring
- Failed bearish break
- Sell-Side Liquidity Sweep
- Successful low-volume test

Possible distribution evidence:

- Upthrust
- Failed bullish break
- Buy-Side Liquidity Sweep
- Successful bearish test

Phase C is optional in real markets and shall not be mandatory for all schematics.

## 5.4 Phase D

Phase D represents emerging directional dominance.

Accumulation evidence may include:

- Sign of Strength
- Bullish BOS
- Acceptance above range resistance
- Higher Lows
- Last Point of Support
- Strong bullish VPA

Distribution evidence may include:

- Sign of Weakness
- Bearish BOS
- Acceptance below range support
- Lower Highs
- Last Point of Supply
- Strong bearish VPA

## 5.5 Phase E

Phase E represents departure from the range and the emergence of markup or markdown.

Evidence may include:

- Sustained acceptance beyond the range
- Trend alignment
- Repeated continuation BOS
- Limited return into the range
- Range boundary acting as support or resistance
- Reduced opposing supply or demand

---

# 6. Inputs

## 6.1 Enable Wyckoff Engine

Type:

Boolean

Default:

Enabled

When disabled:

- No new Wyckoff ranges or events shall be generated.
- Wyckoff visuals shall be hidden.
- Wyckoff alerts shall be disabled.
- Wyckoff scoring contribution shall be removed.
- Dependent modules shall receive unavailable state.

---

## 6.2 Range Detection Mode

Type:

Selection

Options:

- Structure Based
- ATR Compression
- Hybrid

Suggested production default:

Hybrid

---

## 6.3 Minimum Range Bars

Type:

Integer

Suggested default:

20

Minimum:

5

Purpose:

Defines the minimum duration required before a trading range may become confirmed.

---

## 6.4 Maximum Range Bars

Type:

Integer

Suggested default:

300

Purpose:

Limits range duration and retained state.

A value of zero may represent no explicit maximum within safe history bounds.

---

## 6.5 Minimum Range Touches

Type:

Integer

Suggested default:

4

Purpose:

Defines the minimum combined upper-boundary and lower-boundary interactions required to confirm a range.

---

## 6.6 Minimum Boundary Touches Per Side

Type:

Integer

Suggested default:

2

Minimum:

1

---

## 6.7 Range Width Method

Type:

Selection

Options:

- ATR
- Percentage
- Absolute Price
- Structural Swing Width

Suggested default:

ATR

---

## 6.8 Minimum Range Width

Type:

Float

Suggested provisional default:

```text
1.0 ATR
```

---

## 6.9 Maximum Range Width

Type:

Float

Suggested provisional default:

```text
8.0 ATR
```

Purpose:

Prevents extremely broad market structures from being classified as one coherent trading range.

---

## 6.10 Boundary Tolerance

Type:

Float

Suggested provisional default:

```text
0.15 ATR
```

Purpose:

Defines the tolerance zone around range boundaries.

---

## 6.11 Range Break Confirmation Bars

Type:

Integer

Suggested default:

2

Purpose:

Defines the number of closes beyond a range boundary required for Phase E or range-break confirmation.

---

## 6.12 Range Re-entry Window

Type:

Integer

Suggested default:

3

Purpose:

Defines the period during which a breakout returning into the range may be classified as failed or trapped.

---

## 6.13 Spring Penetration Minimum

Type:

Float

Suggested provisional default:

```text
0.02 ATR
```

---

## 6.14 Spring Penetration Maximum

Type:

Float

Suggested provisional default:

```text
0.75 ATR
```

Purpose:

Prevents large accepted breakdowns from being classified as Springs.

---

## 6.15 Upthrust Penetration Minimum

Type:

Float

Suggested provisional default:

```text
0.02 ATR
```

---

## 6.16 Upthrust Penetration Maximum

Type:

Float

Suggested provisional default:

```text
0.75 ATR
```

---

## 6.17 Test Window

Type:

Integer

Suggested default:

10

Purpose:

Defines the maximum number of bars after a Spring or Upthrust in which a Test may be associated with that event.

---

## 6.18 Phase Confirmation Threshold

Type:

Integer

Suggested default:

65

Range:

```text
0 to 100
```

Purpose:

Defines the minimum phase-confidence score required to publish a confirmed phase state.

---

## 6.19 Schematic Confirmation Threshold

Type:

Integer

Suggested default:

70

Purpose:

Defines the minimum confidence required to classify accumulation, distribution, reaccumulation, or redistribution.

---

## 6.20 Require Volume for Full Wyckoff Classification

Type:

Boolean

Suggested default:

Enabled

Purpose:

When volume is unavailable, the engine may still detect ranges and price events, but full Wyckoff confidence shall be reduced.

---

## 6.21 Require Liquidity Context for Spring and Upthrust

Type:

Boolean

Suggested default:

Enabled

---

## 6.22 Show Trading Range

Type:

Boolean

Suggested default:

Enabled

---

## 6.23 Show Range Midline

Type:

Boolean

Suggested default:

Disabled

---

## 6.24 Show Wyckoff Events

Type:

Boolean

Suggested default:

Enabled

---

## 6.25 Show Phase Labels

Type:

Boolean

Suggested default:

Enabled

---

## 6.26 Show Schematic Classification

Type:

Boolean

Suggested default:

Enabled

---

## 6.27 Maximum Visible Wyckoff Events

Type:

Integer

Suggested default:

50

---

## 6.28 Maximum Active Ranges

Type:

Integer

Suggested default:

3

Purpose:

Allows bounded tracking of the current range and limited historical ranges.

---

## 6.29 Enable Wyckoff Alerts

Type:

Boolean

Suggested default:

Enabled

---

## 6.30 Minimum Alert Confidence

Type:

Integer

Suggested default:

70

Range:

```text
0 to 100
```

---

# 7. Input Validation

The engine shall validate that:

- Minimum Range Bars is at least 5.
- Maximum Range Bars is zero or greater than Minimum Range Bars.
- Minimum Range Touches is at least 2.
- Boundary touches per side is at least 1.
- Width thresholds are non-negative.
- Maximum Range Width is greater than Minimum Range Width.
- Boundary Tolerance is non-negative.
- Break Confirmation Bars is at least 1.
- Re-entry Window is non-negative.
- Spring and Upthrust maximum penetration are not below minimum penetration.
- Test Window is non-negative.
- Confidence thresholds are between 0 and 100.
- Maximum Visible Events is positive.
- Maximum Active Ranges is positive.

Invalid input shall not cause runtime failure.

---

# 8. Range Candidate Detection

A trading-range candidate may begin when:

- Prior trend momentum weakens.
- Structural direction becomes Neutral or Transition.
- Price begins oscillating between recurring upper and lower levels.
- Directional efficiency declines.
- ATR-normalised net progress contracts.
- Opposing BOS events fail.
- Liquidity sweeps occur on both sides.
- VPA shows churn or absorption.

A candidate shall remain provisional until minimum range requirements are met.

---

# 9. Structure-Based Range Detection

Structure-based detection may require:

- At least two confirmed highs within an upper tolerance zone.
- At least two confirmed lows within a lower tolerance zone.
- Alternating swings between the zones.
- No sustained close acceptance beyond either boundary.
- Range width within configured limits.
- Minimum duration satisfied.

Equal High and Equal Low clusters shall strengthen range confidence.

---

# 10. Compression-Based Range Detection

Compression-based detection may evaluate:

- Declining ATR
- Reduced MA separation
- Repeated mean crossings
- Low directional efficiency
- Narrowing net progress
- Balanced bullish and bearish bar distribution
- Frequent structural transitions

Compression alone shall not confirm a Wyckoff range.

It shall support the range candidate.

---

# 11. Hybrid Range Detection

Hybrid mode shall combine:

- Confirmed structural boundaries
- Repeated boundary interactions
- Duration
- Width constraints
- Compression or reduced directional progress
- Failure to establish acceptance outside the range

Hybrid mode is the suggested production default.

---

# 12. Range Boundary Establishment

The range shall maintain:

- Upper boundary
- Lower boundary
- Upper zone
- Lower zone
- Midline
- Range width
- Range height in ATR
- Start bar
- Last update bar
- Upper touch count
- Lower touch count
- Break status
- Re-entry status
- Active phase
- Schematic bias
- Range confidence

The engine shall record the structural sources used to establish each boundary.

---

# 13. Range Boundary Updating

Before range confirmation, boundaries may update as new candidate extremes form.

After range confirmation:

- Boundary changes shall be conservative.
- Minor breaches shall not automatically expand the range.
- Springs and Upthrusts shall not redefine boundaries unless the approved policy allows it.
- Accepted structural expansion may invalidate or replace the range.
- Historical confirmed event levels shall remain unchanged.

---

# 14. Range Confirmation

A range may become confirmed when:

- Minimum duration is satisfied.
- Width lies within configured bounds.
- Minimum total touches are present.
- Minimum touches per side are present.
- At least one internal rotation exists.
- No accepted breakout has occurred.
- Confidence exceeds the range-confirmation threshold.

Confirmed range history shall remain immutable.

---

# 15. Range Confidence

The engine may calculate Range Confidence from:

```text
0 to 100
```

Possible components:

| Component | Suggested Weight |
|---|---:|
| Boundary touch count | 20 |
| Alternating rotations | 15 |
| Duration | 10 |
| Structural clarity | 15 |
| Compression | 10 |
| Boundary rejection quality | 15 |
| Lack of external acceptance | 15 |

Weights remain provisional.

---

# 16. Prior Trend Context

The prior trend shall influence schematic interpretation.

Possible relationships:

| Prior Trend | Range Bias Candidate |
|---|---|
| Bearish | Accumulation or bearish continuation |
| Bullish | Distribution or bullish continuation |
| Neutral | Undetermined |
| Transition | Undetermined |

Prior trend shall not determine the schematic by itself.

---

# 17. Schematic Types

Required schematic states:

1. Unavailable
2. Undetermined Trading Range
3. Accumulation Candidate
4. Accumulation
5. Distribution Candidate
6. Distribution
7. Reaccumulation Candidate
8. Reaccumulation
9. Redistribution Candidate
10. Redistribution
11. Failed Accumulation
12. Failed Distribution
13. Range Breakout
14. Range Breakdown

---

# 18. Accumulation Candidate

An Accumulation Candidate may be supported by:

- Prior bearish trend
- Selling Climax
- Stopping Volume
- Sell-Side Liquidity Sweep
- Bullish Absorption
- Repeated support holding
- Reduced downside result on high effort
- Bullish CHOCH
- Spring
- Successful bullish Test
- Sign of Strength

The engine shall require multiple compatible observations.

---

# 19. Distribution Candidate

A Distribution Candidate may be supported by:

- Prior bullish trend
- Buying Climax
- Buy-Side Liquidity Sweep
- Bearish Absorption
- Repeated resistance holding
- Reduced upside result on high effort
- Bearish CHOCH
- Upthrust
- Successful bearish Test
- Sign of Weakness

---

# 20. Reaccumulation Candidate

A Reaccumulation Candidate may be supported by:

- Broader bullish trend
- Temporary trading range
- No major bearish reversal structure
- Sell-Side Sweeps within the range
- No Supply
- Bullish absorption
- Bullish SOS
- LPS behaviour
- Acceptance above the range

---

# 21. Redistribution Candidate

A Redistribution Candidate may be supported by:

- Broader bearish trend
- Temporary trading range
- No major bullish reversal structure
- Buy-Side Sweeps within the range
- No Demand
- Bearish absorption
- Bearish SOW
- LPSY behaviour
- Acceptance below the range

---

# 22. Phase State Model

Suggested phase states:

```text
0 = Unavailable
1 = Pre-Range
2 = Phase A Candidate
3 = Phase A
4 = Phase B
5 = Phase C Candidate
6 = Phase C
7 = Phase D Candidate
8 = Phase D
9 = Phase E
10 = Invalidated
11 = Completed
```

The engine shall distinguish candidate and confirmed states where practical.

---

# 23. Phase Transition Rules

Valid transitions should generally follow:

```text
Pre-Range
→ Phase A Candidate
→ Phase A
→ Phase B
→ Phase C Candidate or Phase D Candidate
→ Phase C or Phase D
→ Phase D
→ Phase E
→ Completed
```

Permitted exceptions:

- Phase B may transition directly to Phase D.
- Phase C may be absent.
- A range may invalidate before Phase D.
- A confirmed breakout may skip some candidate display states.
- Re-entry may return the active state to Phase B or produce a failed-break state according to policy.

The engine shall not require a perfect textbook sequence.

---

# 24. Phase A Detection

Phase A may require a sequence involving:

1. Prior directional trend
2. Preliminary stopping evidence
3. Climax or major stopping event
4. Automatic reaction
5. Secondary Test or stabilisation

## 24.1 Bullish Phase A Candidate

Possible evidence:

- Prior bearish trend
- Preliminary Support
- Selling Climax
- Stopping Volume
- Automatic Rally
- Secondary Test
- Bullish absorption
- Sell-Side Sweep

## 24.2 Bearish Phase A Candidate

Possible evidence:

- Prior bullish trend
- Preliminary Supply
- Buying Climax
- Automatic Reaction
- Secondary Test
- Bearish absorption
- Buy-Side Sweep

---

# 25. Preliminary Support

A Preliminary Support candidate may require:

- Existing bearish trend
- Elevated or expanding RVOL
- Increased spread or volatility
- Reduced downside efficiency
- Close away from the low
- Lower wick or bullish absorption
- No confirmed range yet

Preliminary Support shall precede or support a later Selling Climax interpretation.

---

# 26. Preliminary Supply

A Preliminary Supply candidate may require:

- Existing bullish trend
- Elevated or expanding RVOL
- Increased spread or volatility
- Reduced upside efficiency
- Close away from the high
- Upper wick or bearish absorption
- No confirmed range yet

---

# 27. Selling Climax

The Wyckoff Engine shall consume the VPA Selling Climax classification and evaluate additional context.

A Wyckoff Selling Climax candidate may require:

- Prior bearish trend
- Extreme or high RVOL
- Wide spread or volatility expansion
- Sell-Side Liquidity interaction
- Rejection or recovery from the low
- Followed by an Automatic Rally candidate

A VPA Selling Climax without sequence confirmation shall remain a candidate event.

---

# 28. Buying Climax

A Wyckoff Buying Climax candidate may require:

- Prior bullish trend
- Extreme or high RVOL
- Wide spread or volatility expansion
- Buy-Side Liquidity interaction
- Rejection or weakness from the high
- Followed by an Automatic Reaction candidate

---

# 29. Automatic Rally

An Automatic Rally may be detected after:

- Selling Climax
- Stopping Volume
- Major Sell-Side Sweep
- Bullish absorption

It shall represent the first meaningful bullish reaction.

Possible confirmation:

- Bullish swing forms
- Price moves a minimum ATR distance from the climax low
- Bullish structure develops
- RVOL normalises or remains constructive
- Range upper boundary candidate is established

---

# 30. Automatic Reaction

An Automatic Reaction may be detected after:

- Buying Climax
- Major Buy-Side Sweep
- Bearish absorption

It shall represent the first meaningful bearish reaction.

The resulting low may establish the initial range lower boundary.

---

# 31. Secondary Test

A Secondary Test may require:

- A prior Climax and Automatic Reaction/Rally exist.
- Price revisits the climax area or relevant boundary.
- Effort is reduced relative to the Climax.
- Spread is narrower or rejection is evident.
- The range remains intact.
- No accepted break beyond the climax extreme occurs.

A Secondary Test shall be directionally interpreted according to the schematic context.

---

# 32. Phase B Detection

Phase B may be confirmed when:

- A trading range is established.
- Both boundaries have been tested.
- Multiple internal rotations occur.
- Neither side achieves sustained acceptance.
- Supply-demand evidence remains mixed or evolving.
- Cause-building duration becomes meaningful.

Possible supporting events:

- Churn
- Absorption
- No Demand
- No Supply
- Equal Highs
- Equal Lows
- Minor Springs
- Minor Upthrusts
- Failed structural breaks

---

# 33. Phase B Internal Events

The engine may track:

- Internal BOS
- Internal CHOCH
- Mid-range tests
- Secondary Tests
- Minor liquidity sweeps
- Absorption zones
- Repeated boundary touches

These events may modify confidence but shall not automatically change the phase.

---

# 34. Spring Detection

A Spring candidate shall generally require:

- An active confirmed range.
- Price trades below the lower boundary or Sell-Side Liquidity.
- Penetration is within configured limits.
- Price returns into the range within the confirmation window.
- Acceptance below the range is not established.
- Liquidity Engine confirms Sell-Side Sweep, Bear Trap, or failed bearish break.
- Structural and VPA context do not indicate a valid accepted breakdown.

Supporting evidence may include:

- Stopping Volume
- Bullish Absorption
- Selling Climax
- Close near high
- Long lower wick
- Bullish CHOCH
- Reduced follow-through below support

A Spring shall represent a Phase C candidate in accumulation or reaccumulation context.

---

# 35. Upthrust Detection

An Upthrust candidate shall generally require:

- An active confirmed range.
- Price trades above the upper boundary or Buy-Side Liquidity.
- Penetration is within configured limits.
- Price returns into the range.
- Acceptance above the range is not established.
- Liquidity Engine confirms Buy-Side Sweep, Bull Trap, or failed bullish break.

Supporting evidence may include:

- Buying Climax
- Bearish Absorption
- Close near low
- Long upper wick
- Bearish CHOCH
- Reduced follow-through above resistance

An Upthrust shall represent a Phase C candidate in distribution or redistribution context.

---

# 36. Spring and Upthrust Distinction from Boundary Sweeps

Not every range-boundary sweep is a Spring or Upthrust.

A Wyckoff classification shall require:

- Appropriate range phase
- Schematic compatibility
- Penetration and return behaviour
- Contextual VPA evidence
- Lack of sustained acceptance
- Suitable event sequence

The Liquidity Engine remains authoritative for the sweep itself.

The Wyckoff Engine determines whether the sweep fits a schematic role.

---

# 37. Test after Spring

A bullish Test after Spring may require:

- A confirmed or high-confidence Spring exists.
- Price revisits the lower part of the range or Spring area.
- Spread is Narrow or Normal.
- RVOL is Low or lower than on the Spring.
- Price closes constructively.
- No accepted break below the Spring low occurs.
- Bullish follow-through appears within the Test window.

A successful Test shall increase Phase C and accumulation confidence.

---

# 38. Test after Upthrust

A bearish Test after Upthrust may require:

- A confirmed or high-confidence Upthrust exists.
- Price revisits the upper part of the range or Upthrust area.
- Spread is Narrow or Normal.
- RVOL is Low or lower than on the Upthrust.
- Price closes weakly.
- No accepted break above the Upthrust high occurs.
- Bearish follow-through appears.

---

# 39. Phase C Detection

Phase C may be confirmed when:

- A valid Spring or Upthrust occurs.
- The event fits the active schematic.
- The range remains valid.
- Acceptance outside the range is rejected.
- Test behaviour supports the interpretation, where required.
- Phase confidence exceeds the threshold.

Phase C may be absent.

The engine shall not reduce confidence solely because no Spring or Upthrust occurs.

---

# 40. Sign of Strength

A Sign of Strength candidate may require:

- Bullish expansion from the range.
- Wide or strong bullish result.
- High or constructive RVOL.
- Close near high.
- Bullish BOS.
- Accepted Buy-Side Liquidity Break or range breakout.
- Follow-through beyond range resistance.
- Accumulation or reaccumulation context.

The event shall be distinguished from a one-bar false breakout.

---

# 41. Sign of Weakness

A Sign of Weakness candidate may require:

- Bearish expansion from the range.
- Wide or strong bearish result.
- High or constructive RVOL.
- Close near low.
- Bearish BOS.
- Accepted Sell-Side Liquidity Break or range breakdown.
- Follow-through below range support.
- Distribution or redistribution context.

---

# 42. Last Point of Support

A Last Point of Support candidate may require:

- Prior Sign of Strength.
- Pullback toward the former range high, mid-range, or support zone.
- Lower RVOL than the SOS.
- Narrower spread or reduced selling result.
- No Supply or bullish Test.
- Support holds.
- Bullish follow-through resumes.

Multiple LPS events may occur.

---

# 43. Last Point of Supply

A Last Point of Supply candidate may require:

- Prior Sign of Weakness.
- Rally toward former range support, mid-range, or resistance.
- Lower RVOL than the SOW.
- Narrower spread or reduced buying result.
- No Demand or bearish Test.
- Resistance holds.
- Bearish follow-through resumes.

Multiple LPSY events may occur.

---

# 44. Phase D Detection

Phase D may be confirmed when directional evidence becomes dominant.

Accumulation or reaccumulation evidence:

- SOS
- Bullish BOS
- Higher Low
- LPS
- Accepted range breakout
- Strong bullish VPA
- Reduced supply on pullbacks

Distribution or redistribution evidence:

- SOW
- Bearish BOS
- Lower High
- LPSY
- Accepted range breakdown
- Strong bearish VPA
- Reduced demand on rallies

---

# 45. Phase E Detection

Phase E may be confirmed when:

- Price sustains acceptance outside the range.
- Trend aligns with the breakout direction.
- The range boundary holds on retest or price departs without meaningful return.
- Continuation structure develops.
- Opposing range interpretation weakens materially.
- Break Confirmation Bars requirement is satisfied.

Phase E shall represent range departure rather than a single breakout bar.

---

# 46. Range Breakout

A bullish range breakout may require:

- Close above the upper range boundary.
- BOS/CHOCH confirmation where applicable.
- Accepted Buy-Side Break.
- Required confirmation bars.
- No immediate re-entry.
- Bullish result quality.

A bearish range breakdown shall use inverse logic.

---

# 47. Failed Range Breakout

A failed bullish range breakout may occur when:

1. Price appears to accept above the range.
2. Price returns inside within the Re-entry Window.
3. Bullish follow-through fails.
4. A Bull Trap or failed bullish break is confirmed.

This event may increase distribution confidence.

The original breakout event shall remain recorded.

---

# 48. Failed Range Breakdown

A failed bearish range breakdown may occur when:

1. Price appears to accept below the range.
2. Price returns inside within the Re-entry Window.
3. Bearish follow-through fails.
4. A Bear Trap or failed bearish break is confirmed.

This event may increase accumulation confidence.

---

# 49. Range Invalidation

An active range may become invalidated when:

- Sustained acceptance occurs beyond a boundary.
- Range width expands beyond approved limits.
- A major structural redefinition supersedes the range.
- Maximum duration is exceeded.
- Both boundaries shift materially.
- Data becomes insufficient or inconsistent.
- A new higher-priority range replaces it.

Invalidation shall not delete historical range events.

---

# 50. Range Completion

A range may become Completed when:

- Phase E is established.
- A breakout or breakdown remains accepted.
- The range no longer serves as the active market context.
- Historical metadata is retained.

Completed ranges may remain available for later retest interpretation.

---

# 51. Event Sequence Model

Each active range may retain ordered event history such as:

```text
Preliminary Support
Selling Climax
Automatic Rally
Secondary Test
Spring
Test
Sign of Strength
Last Point of Support
Phase E
```

or:

```text
Preliminary Supply
Buying Climax
Automatic Reaction
Secondary Test
Upthrust
Test
Sign of Weakness
Last Point of Supply
Phase E
```

The engine shall not require every textbook event.

Missing events shall lower confidence where appropriate but shall not automatically invalidate the schematic.

---

# 52. Event Eligibility

A Wyckoff event shall be eligible only when:

- Required upstream event data is available.
- Required range or trend context exists.
- The event occurs in a plausible sequence position.
- It has not already been emitted for the same source interaction.
- Confidence meets the event publication threshold where configured.

---

# 53. Duplicate-Event Suppression

The engine shall prevent duplicate Wyckoff events from one source event.

Examples:

- One Sell-Side Sweep shall not repeatedly emit Spring.
- Repeated closes above the range shall not repeatedly emit SOS.
- One Climax bar shall not repeatedly register as Preliminary Support and Selling Climax unless the event model explicitly allows both roles.

Separate roles may be stored as metadata where appropriate.

---

# 54. Event Priority

Suggested event-priority order for one bar:

1. Range invalidation
2. Failed range breakout or breakdown
3. Spring or Upthrust
4. Sign of Strength or Sign of Weakness
5. Selling Climax or Buying Climax
6. Automatic Rally or Automatic Reaction
7. Secondary Test
8. Last Point of Support or Last Point of Supply
9. Preliminary Support or Preliminary Supply
10. Phase transition
11. Internal range event

This order is provisional.

One primary Wyckoff event may be published per bar.

Supporting sequence roles may remain in metadata.

---

# 55. Schematic Confidence

The engine shall calculate schematic confidence from:

```text
0 to 100
```

Possible components:

| Component | Suggested Weight |
|---|---:|
| Prior trend compatibility | 10 |
| Range quality | 15 |
| Event sequence quality | 20 |
| Volume and VPA agreement | 15 |
| Liquidity-event agreement | 15 |
| Structural confirmation | 15 |
| Follow-through | 10 |

Weights remain provisional.

---

# 56. Phase Confidence

Phase Confidence shall represent the quality of the current phase classification.

Possible components:

- Required prior phase exists
- Minimum duration
- Required event count
- Boundary interaction quality
- Schematic compatibility
- Structural state
- VPA confirmation
- Break or rejection behaviour

Phase Confidence shall be separate from Schematic Confidence.

---

# 57. Event Confidence

Each Wyckoff event may have its own confidence value.

Example Spring confidence factors:

| Component | Suggested Weight |
|---|---:|
| Confirmed range | 15 |
| Sell-Side Sweep | 20 |
| Return into range | 20 |
| VPA support | 15 |
| Penetration quality | 10 |
| Structural reversal evidence | 10 |
| Follow-through or Test | 10 |

---

# 58. Confidence Penalties

Confidence may be reduced by:

- No confirmed range
- Weak boundary definition
- Missing volume
- Neutral sequence context
- Excessive penetration
- Immediate acceptance outside the range
- Conflicting BOS/CHOCH state
- Contradictory VPA event
- Insufficient history
- Gap distortion
- Multiple overlapping ranges
- Ambiguous prior trend
- Repeated boundary crossing

---

# 59. Trend Integration

The Trend Engine shall provide:

- Current trend direction
- Trend strength
- Trend transition
- Higher-timeframe trend
- Prior trend context

Possible uses:

- Bearish trend before accumulation
- Bullish trend before distribution
- Bullish trend before reaccumulation
- Bearish trend before redistribution
- Phase E confirmation
- Schematic confidence adjustment

Trend shall not independently classify the schematic.

---

# 60. Market Structure Integration

The Market Structure Engine shall provide:

- Swing Highs
- Swing Lows
- Equal Highs
- Equal Lows
- Structural direction
- Protection levels
- Structural transitions
- Pivot metadata

Possible uses:

- Range boundaries
- Internal rotations
- AR and AR low/high
- Secondary Tests
- Higher Low or Lower High development
- Phase D confirmation
- Range invalidation

---

# 61. BOS and CHOCH Integration

The BOS and CHOCH Engine shall provide:

- Bullish BOS
- Bearish BOS
- Bullish CHOCH
- Bearish CHOCH
- Failed breaks
- Acceptance
- Rejection
- Structural-event confidence

Possible relationships:

| Structural Event | Wyckoff Use |
|---|---|
| Bullish CHOCH after Spring | Accumulation confirmation |
| Bearish CHOCH after Upthrust | Distribution confirmation |
| Bullish BOS above range | SOS or Phase D |
| Bearish BOS below range | SOW or Phase D |
| Failed bearish break | Spring support |
| Failed bullish break | Upthrust support |

---

# 62. Liquidity Integration

The Liquidity Engine shall provide:

- Buy-Side Sweep
- Sell-Side Sweep
- Bull Trap
- Bear Trap
- Accepted liquidity break
- Equal-level interaction
- Level priority
- Event confidence

Possible mappings:

```text
Sell-Side Sweep at Range Low
+
Return Inside
=
Spring Candidate
```

```text
Buy-Side Sweep at Range High
+
Return Inside
=
Upthrust Candidate
```

```text
Accepted Buy-Side Break
+
Bullish Follow-Through
=
SOS Candidate
```

---

# 63. RVOL Integration

RVOL shall support:

- Climax detection
- Preliminary stopping evidence
- Test comparison
- Effort-versus-result interpretation
- SOS and SOW quality
- LPS and LPSY volume contraction
- Spring and Upthrust confidence
- Phase transition confidence

Missing RVOL shall reduce confidence but shall not prevent price-range detection.

---

# 64. VPA Integration

The VPA Engine shall provide:

- Selling Climax
- Buying Climax
- Stopping Volume
- No Supply
- No Demand
- Bullish Absorption
- Bearish Absorption
- Churn
- Test
- Successful Test
- Failed Test
- Strength Confirmation
- Weakness Confirmation
- Effort and Result states

Possible mappings:

| VPA Event | Wyckoff Interpretation |
|---|---|
| Selling Climax | SC candidate |
| Buying Climax | BC candidate |
| Stopping Volume | PS or support evidence |
| No Supply | Test or LPS evidence |
| No Demand | Test or LPSY evidence |
| Bullish Absorption | Accumulation evidence |
| Bearish Absorption | Distribution evidence |
| Churn | Phase B evidence |
| Strength Confirmation | SOS evidence |
| Weakness Confirmation | SOW evidence |

---

# 65. Institutional Activity Integration

The Institutional Activity Engine may consume:

- Schematic type
- Current phase
- Range boundaries
- Spring or Upthrust
- SOS or SOW
- LPS or LPSY
- Event and phase confidence
- Absorption evidence
- Volume context
- Structural confirmation

The Wyckoff Engine shall not claim actual institutional activity.

It shall expose structured evidence.

---

# 66. Scoring Integration

Possible scoring effects:

- Confirmed Spring may support bullish reversal evidence.
- Successful Test after Spring may add stronger bullish evidence.
- SOS may support bullish continuation.
- LPS may strengthen bullish continuation.
- Upthrust may support bearish reversal evidence.
- Successful Test after Upthrust may add stronger bearish evidence.
- SOW may support bearish continuation.
- LPSY may strengthen bearish continuation.
- Phase B may reduce directional confidence.
- Phase E may increase directional confidence.
- Undetermined range shall reduce conviction.
- Missing volume shall reduce Wyckoff contribution rather than create a penalty.

Final values belong to the Signal Scoring specification.

---

# 67. Outputs

The Wyckoff Engine shall expose:

- Engine enabled status
- Range candidate status
- Range confirmed status
- Active range ID
- Range start bar
- Range upper boundary
- Range lower boundary
- Range midline
- Range width
- Range width in ATR
- Upper touch count
- Lower touch count
- Range confidence
- Range lifecycle state
- Current Wyckoff phase
- Phase candidate
- Phase confidence
- Schematic type
- Schematic confidence
- Prior trend context
- Preliminary Support event
- Preliminary Supply event
- Selling Climax event
- Buying Climax event
- Automatic Rally event
- Automatic Reaction event
- Secondary Test event
- Spring event
- Upthrust event
- Test after Spring event
- Test after Upthrust event
- Sign of Strength event
- Sign of Weakness event
- Last Point of Support event
- Last Point of Supply event
- Range breakout event
- Range breakdown event
- Failed range breakout event
- Failed range breakdown event
- Range invalidation event
- Last Wyckoff event type
- Last Wyckoff event direction
- Last Wyckoff event bar index
- Last Wyckoff event confidence
- Sufficient-history status

---

# 68. Suggested Enumerations

## Schematic Type

```text
 0 = Unavailable
 1 = Undetermined Range
 2 = Accumulation Candidate
 3 = Accumulation
 4 = Distribution Candidate
 5 = Distribution
 6 = Reaccumulation Candidate
 7 = Reaccumulation
 8 = Redistribution Candidate
 9 = Redistribution
10 = Failed Accumulation
11 = Failed Distribution
12 = Bullish Range Breakout
13 = Bearish Range Breakdown
```

## Phase State

```text
 0 = Unavailable
 1 = Pre-Range
 2 = Phase A Candidate
 3 = Phase A
 4 = Phase B
 5 = Phase C Candidate
 6 = Phase C
 7 = Phase D Candidate
 8 = Phase D
 9 = Phase E
10 = Invalidated
11 = Completed
```

## Wyckoff Event Type

```text
 0 = None
 1 = Preliminary Support
 2 = Preliminary Supply
 3 = Selling Climax
 4 = Buying Climax
 5 = Automatic Rally
 6 = Automatic Reaction
 7 = Secondary Test Low
 8 = Secondary Test High
 9 = Spring
10 = Upthrust
11 = Test after Spring
12 = Test after Upthrust
13 = Sign of Strength
14 = Sign of Weakness
15 = Last Point of Support
16 = Last Point of Supply
17 = Bullish Range Breakout
18 = Bearish Range Breakdown
19 = Failed Bullish Range Breakout
20 = Failed Bearish Range Breakdown
21 = Range Invalidated
22 = Range Completed
```

Internal enumeration values shall remain stable after implementation begins.

---

# 69. Dashboard Integration

The dashboard shall be able to display:

- Current schematic
- Current phase
- Phase confidence
- Range confidence
- Last Wyckoff event
- Event confidence
- Range status
- Breakout status

Suggested output:

```text
Wyckoff: Accumulation
Phase: D
Confidence: 78
Event: Sign of Strength
```

For ambiguity:

```text
Wyckoff: Undetermined Range
Phase: B
Confidence: 54
```

When no valid range exists:

```text
Wyckoff: N/A
```

---

# 70. Chart Visualisation

Optional visual elements may include:

- Range boundary lines
- Boundary zones
- Range midline
- Phase background shading
- Phase labels
- Wyckoff event labels
- Spring and Upthrust markers
- SOS and SOW markers
- LPS and LPSY markers
- Breakout or breakdown markers
- Range invalidation marker

Visual defaults shall remain conservative to reduce clutter.

---

# 71. Suggested Event Abbreviations

```text
PS = Preliminary Support
PSY = Preliminary Supply
SC = Selling Climax
BC = Buying Climax
AR = Automatic Rally
ARx = Automatic Reaction
ST = Secondary Test
SP = Spring
UT = Upthrust
TSP = Test after Spring
TUT = Test after Upthrust
SOS = Sign of Strength
SOW = Sign of Weakness
LPS = Last Point of Support
LPSY = Last Point of Supply
```

Final abbreviations shall be approved in the Dashboard and Settings specifications.

---

# 72. Alert Integration

The engine shall support alerts for:

- Trading Range confirmed
- Accumulation candidate
- Accumulation confirmed
- Distribution candidate
- Distribution confirmed
- Reaccumulation confirmed
- Redistribution confirmed
- Phase transition
- Selling Climax
- Buying Climax
- Spring
- Upthrust
- Test after Spring
- Test after Upthrust
- Sign of Strength
- Sign of Weakness
- Last Point of Support
- Last Point of Supply
- Bullish range breakout
- Bearish range breakdown
- Failed range breakout
- Failed range breakdown
- Range invalidated

Alerts shall be filtered by:

- Event enabled state
- Confirmed bar
- Minimum confidence
- Optional schematic confirmation
- Optional phase confirmation

Alert messages shall include:

- Symbol
- Chart timeframe
- Event type
- Schematic
- Phase
- Range boundary
- Event confidence
- Phase confidence
- RVOL where available
- Timestamp where supported

Example:

```text
BTCUSD 4H — Spring confirmed below range low 64200 — Accumulation Phase C — Confidence 82
```

---

# 73. Repainting Policy

The Wyckoff Engine shall not intentionally repaint confirmed events.

Requirements:

- Confirmed upstream events shall be used.
- Range confirmation shall occur only when minimum evidence exists.
- No lookahead shall be used.
- Confirmed event history shall remain immutable.
- Later evidence may update current phase or schematic confidence.
- A later invalidation shall create a separate event.
- Historical event labels shall not move to earlier bars.
- Developing ranges and phases shall remain explicitly provisional.

---

# 74. Real-Time Behaviour

During a developing bar:

- Range boundaries may be tested.
- Spring or Upthrust candidates may appear and disappear.
- RVOL and VPA classification may change.
- Phase confidence may change.
- A breakout may remain provisional.
- Re-entry may alter current active-state interpretation.

Production event publication shall use confirmed bars.

Optional developing states may appear in the dashboard or debug mode.

---

# 75. Historical State Stability

Once confirmed:

- Range start bar shall remain stable.
- Confirmed range boundaries shall not change retroactively except under an explicitly approved bounded-adjustment policy.
- Event bar indexes shall remain stable.
- Event types shall remain stable.
- Event confidence at confirmation shall remain stored.
- Later schematic invalidation shall not erase prior events.

---

# 76. Same-Bar Event Handling

A single bar may produce:

- Liquidity Sweep
- CHOCH
- VPA event
- Spring or Upthrust
- Phase transition

The engine shall use deterministic ordering.

Suggested order:

1. Upstream event resolution
2. Range interaction
3. Spring or Upthrust evaluation
4. Breakout or breakdown evaluation
5. Wyckoff event classification
6. Phase transition
7. Schematic-confidence update
8. Event publication

---

# 77. Gap Handling

A gap through a range boundary shall not automatically qualify as a Spring or Upthrust.

Gap interaction may produce:

- Accepted range breakout
- Accepted range breakdown
- Gap rejection
- Failed gap breakout
- Unclassified interaction

Spring and Upthrust require actual rejection behaviour and suitable sequence context.

Gap bars may receive reduced confidence.

---

# 78. Overlapping Range Handling

Multiple candidate ranges may overlap.

The engine shall apply deterministic selection using factors such as:

- Recency
- Touch count
- Range confidence
- Structural significance
- Duration
- Width suitability
- Current price containment

The first production version should expose one primary active range.

Limited historical ranges may remain stored.

---

# 79. Nested Range Handling

A smaller range may form within a larger range.

Possible policies:

- Track only the primary range.
- Track one external and one internal range.
- Prefer the most recent structurally significant range.

The first production version should track one active primary range unless implementation limits permit an internal layer.

---

# 80. Object Management

Wyckoff objects shall use bounded collections.

When limits are reached:

- Completed or invalidated range objects shall be deleted first.
- Old inactive event labels shall be removed before active-range elements.
- Current active range boundaries shall be preserved.
- Internal references shall be removed safely.
- Historical metadata may remain deeper than visible objects within safe bounds.

---

# 81. Performance Requirements

The engine shall:

- Reuse all upstream calculations.
- Avoid independent swing or RVOL recalculation.
- Evaluate only active range candidates and bounded historical ranges.
- Avoid full-history rescanning on every bar.
- Use bounded arrays.
- Avoid drawing objects when visuals are disabled.
- Build dashboard strings only when needed.
- Limit sequence-history depth.
- Use incremental range-state updates.

---

# 82. Error Handling

The engine shall safely handle:

- No active range
- Insufficient history
- Missing volume
- Zero ATR
- Gap bars
- Extremely narrow ranges
- Extremely wide ranges
- Overlapping ranges
- Nested ranges
- Same-bar opposing boundary breaches
- Repeated range re-entry
- Ambiguous trend context
- Conflicting VPA evidence
- Conflicting schematic evidence
- Object-limit pressure
- Symbol changes
- Timeframe changes

No edge case shall cause a runtime error.

---

# 83. Testing Requirements

The Wyckoff Engine shall be tested across:

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

- Clean accumulation range
- Clean distribution range
- Reaccumulation
- Redistribution
- Range without Phase C
- Spring and successful Test
- Upthrust and successful Test
- SOS breakout
- SOW breakdown
- LPS
- LPSY
- Failed breakout
- Failed breakdown
- Long Phase B
- Missing volume
- Gap breakout
- Overlapping range
- Nested range
- Neutral prior trend
- Insufficient history

---

# 84. Functional Test Cases

## Test WYCK-001 — Range Candidate

Given:

- Price oscillates between repeated upper and lower structural zones.
- Minimum duration is not yet satisfied.

Expected:

- Range Candidate is active.
- Range Confirmed is false.
- No confirmed phase is published.

---

## Test WYCK-002 — Range Confirmation

Given:

- Minimum duration is satisfied.
- Minimum touches exist on both sides.
- Range width is valid.
- No accepted external break exists.
- Range confidence exceeds threshold.

Expected:

- Trading Range is confirmed.
- Boundaries and start bar are stored.

---

## Test WYCK-003 — Accumulation Candidate

Given:

- Prior trend is Bearish.
- Selling Climax or Stopping Volume occurs.
- Automatic Rally develops.
- Range forms.
- Sell-Side rejection evidence exists.

Expected:

- Accumulation Candidate is published.
- Full Accumulation remains unconfirmed until additional evidence appears.

---

## Test WYCK-004 — Distribution Candidate

Given:

- Prior trend is Bullish.
- Buying Climax occurs.
- Automatic Reaction develops.
- Range forms.
- Buy-Side rejection evidence exists.

Expected:

- Distribution Candidate is published.

---

## Test WYCK-005 — Spring

Given:

- A confirmed range exists.
- Price trades below the lower boundary.
- Penetration is within limits.
- Sell-Side Sweep confirms.
- Price closes back inside.
- Acceptance below does not occur.

Expected:

- Spring event is confirmed once.
- Phase C Candidate becomes active.
- Range lower boundary remains unchanged.

---

## Test WYCK-006 — Upthrust

Given:

- A confirmed range exists.
- Price trades above the upper boundary.
- Buy-Side Sweep confirms.
- Price closes back inside.

Expected:

- Upthrust event is confirmed once.

---

## Test WYCK-007 — Spring Test

Given:

- A Spring is confirmed.
- Price revisits the lower range.
- RVOL is lower than on the Spring.
- Spread is Narrow.
- Support holds.
- Bullish follow-through occurs.

Expected:

- Test after Spring is confirmed.
- Accumulation and Phase C confidence increase.

---

## Test WYCK-008 — Upthrust Test

Given:

- An Upthrust is confirmed.
- Price revisits the upper range.
- RVOL is reduced.
- Resistance holds.
- Bearish follow-through occurs.

Expected:

- Test after Upthrust is confirmed.

---

## Test WYCK-009 — Sign of Strength

Given:

- Accumulation or Reaccumulation context exists.
- Price closes above the range.
- Bullish BOS confirms.
- Accepted Buy-Side Break confirms.
- VPA indicates Strength Confirmation.
- Follow-through persists.

Expected:

- Sign of Strength is confirmed.
- Phase D becomes active or confirmed.

---

## Test WYCK-010 — Sign of Weakness

Given:

- Distribution or Redistribution context exists.
- Price closes below the range.
- Bearish BOS confirms.
- Accepted Sell-Side Break confirms.
- VPA indicates Weakness Confirmation.

Expected:

- Sign of Weakness is confirmed.

---

## Test WYCK-011 — Last Point of Support

Given:

- SOS is confirmed.
- Price pulls back on reduced RVOL.
- Former resistance or support holds.
- No Supply or bullish Test occurs.
- Bullish continuation resumes.

Expected:

- Last Point of Support is confirmed.

---

## Test WYCK-012 — Last Point of Supply

Given:

- SOW is confirmed.
- Price rallies on reduced RVOL.
- Resistance holds.
- No Demand or bearish Test occurs.

Expected:

- Last Point of Supply is confirmed.

---

## Test WYCK-013 — Phase B Direct to Phase D

Given:

- A confirmed range exists.
- No Spring or Upthrust occurs.
- Strong accepted breakout develops.

Expected:

- Engine may transition from Phase B to Phase D.
- Missing Phase C does not invalidate the schematic.

---

## Test WYCK-014 — Failed Bullish Range Breakout

Given:

- Price initially accepts above the range.
- Price returns inside within the Re-entry Window.
- Bull Trap confirms.

Expected:

- Original breakout remains stored.
- Failed Bullish Range Breakout is generated separately.
- Distribution confidence may increase.

---

## Test WYCK-015 — Failed Bearish Range Breakdown

Given:

- Price initially accepts below the range.
- Price returns inside.
- Bear Trap confirms.

Expected:

- Failed Bearish Range Breakdown is generated separately.
- Accumulation confidence may increase.

---

## Test WYCK-016 — Phase E

Given:

- Range breakout or breakdown remains accepted.
- Trend aligns.
- Continuation structure develops.
- Price does not materially re-enter the range.

Expected:

- Phase E is confirmed.
- Range may become Completed.

---

## Test WYCK-017 — Missing Volume

Given:

- Range price structure is valid.
- Volume is unavailable.

Expected:

- Range may still be detected.
- Volume-dependent events receive reduced confidence or remain unavailable.
- No runtime error occurs.

---

## Test WYCK-018 — Duplicate Spring Suppression

Given:

- One Spring is confirmed.
- Price remains near the lower boundary for several bars.

Expected:

- Spring is not repeatedly emitted.
- Test logic may activate separately.

---

## Test WYCK-019 — Range Invalidation

Given:

- Active range expands beyond maximum width or is structurally superseded.

Expected:

- Range Invalidation event is generated.
- Historical events remain stored.
- Active range state becomes Invalidated.

---

## Test WYCK-020 — Historical Stability

Given:

- A Wyckoff event and phase were confirmed.
- Additional future bars are loaded.

Expected:

- Original event bar, type, level, and confirmation confidence remain unchanged.

---

# 85. Acceptance Criteria

The Wyckoff Engine shall be complete when:

- Trading-range candidates and confirmed ranges operate correctly.
- Range boundaries remain stable after confirmation.
- Phase A through Phase E states are supported.
- Accumulation and Distribution remain distinct.
- Reaccumulation and Redistribution are supported.
- Spring and Upthrust require valid range and liquidity context.
- Tests after Spring and Upthrust operate correctly.
- SOS, SOW, LPS, and LPSY classifications work.
- Phase C remains optional.
- Failed breakouts produce separate events.
- Schematic, phase, range, and event confidence values are available.
- Duplicate events are suppressed.
- Missing volume is handled safely.
- Confirmed historical events remain stable.
- Alerts obey confidence rules.
- Object counts remain bounded.
- TradingView compilation succeeds.
- Required tests pass.

---

# 86. Open Design Decisions

The following items remain unresolved:

1. Final range-detection mode.
2. Final minimum range duration.
3. Final range-touch requirements.
4. Final range-width limits.
5. Final boundary-tolerance method.
6. Final range-confirmation confidence threshold.
7. Final boundary-adjustment policy after confirmation.
8. Whether Spring and Upthrust may redefine range extremes.
9. Final breakout-confirmation bars.
10. Final range re-entry window.
11. Final Spring and Upthrust penetration limits.
12. Whether a Test is mandatory after Spring or Upthrust.
13. Final Test window.
14. Final Phase A requirements.
15. Final Phase B requirements.
16. Final Phase C requirements.
17. Final Phase D requirements.
18. Final Phase E requirements.
19. Final event-priority order.
20. Final range-confidence weights.
21. Final schematic-confidence weights.
22. Final phase-confidence weights.
23. Final event-confidence weights.
24. Whether Preliminary Support and Preliminary Supply are included in Version 1.0.
25. Whether Automatic Rally and Automatic Reaction require explicit swing confirmation.
26. Whether one bar may hold multiple Wyckoff sequence roles.
27. Whether Phase B may contain labelled minor Springs and Upthrusts.
28. Whether LPS and LPSY may occur multiple times.
29. Final failed-breakout effect on schematic confidence.
30. Final range-completion policy.
31. Final overlapping-range selection policy.
32. Whether nested ranges are supported in Version 1.0.
33. Whether one external and one internal range are tracked.
34. Maximum stored range history.
35. Maximum stored event-sequence depth.
36. Missing-volume confidence penalty.
37. Gap-event policy.
38. Same-bar phase-transition policy.
39. Whether re-entry after Phase E may reactivate the prior range.
40. Whether schematic classification is shown before confirmation threshold is met.

---

# 87. Future Enhancements

Possible future enhancements include:

- Multiple concurrent Wyckoff ranges
- Internal and external ranges
- Schematic 1 and Schematic 2 variants
- Creek and Ice terminology
- Jump Across the Creek
- Back Up to the Edge of the Creek
- Fall Through the Ice
- Multi-timeframe Wyckoff context
- Composite schematic matching
- Event-sequence similarity scoring
- Statistical range outcomes
- Automatic narrative generation
- Adaptive phase thresholds
- Session-specific range models
- Volume-profile integration
- Point-and-figure cause estimates
- Range duration projections
- Schematic heatmaps
- Historical schematic replay