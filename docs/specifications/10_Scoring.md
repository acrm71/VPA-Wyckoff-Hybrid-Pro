# Signal Scoring Engine Specification

---

## Document Information

**Document ID:** SPEC-110

**Version:** 0.1 Draft

**Status:** Draft

**Module:** Signal Scoring Engine

**Dependencies:**

- Framework
- Settings
- Trend Engine
- Relative Volume Engine
- Market Structure Engine
- BOS and CHOCH Engine
- Liquidity Engine
- Volume Price Analysis Engine
- Wyckoff Engine
- Institutional Activity Engine
- Shared Evidence Model
- Dashboard
- Alerts

**Dependent Modules:**

- Dashboard
- Alerts
- Strategy or execution layer in future releases

---

# 1. Purpose

The Signal Scoring Engine aggregates approved analytical evidence into a final, explainable assessment of bullish opportunity, bearish opportunity, setup quality, and entry readiness.

Its primary responsibilities are to calculate:

- Bullish composite score
- Bearish composite score
- Bullish continuation score
- Bearish continuation score
- Bullish reversal score
- Bearish reversal score
- Setup quality
- Entry readiness
- Directional dominance
- Signal confidence
- Evidence conflict
- No-trade conditions
- Long setup state
- Short setup state
- Neutral state
- Watch state
- Ready state
- Triggered state
- Invalidated state

The engine shall combine upstream evidence without duplicating the analytical logic owned by those modules.

It shall not place trades.

---

# 2. Objectives

The Signal Scoring Engine shall:

- Consume standardised evidence from all approved upstream modules.
- Calculate bullish and bearish scores independently.
- Distinguish continuation from reversal setups.
- Separate setup quality from entry timing.
- Control correlated and duplicate evidence.
- Apply source and category contribution caps.
- Reduce scores when evidence conflicts.
- Support missing-data-safe scoring.
- Produce one primary signal state.
- Produce one directional signal bias.
- Produce one signal-confidence value.
- Explain the strongest positive and negative contributors.
- Avoid rapid signal-state flicker.
- Preserve confirmed historical signal transitions.
- Support conservative no-trade classification.
- Remain deterministic and testable.
- Provide stable outputs to Dashboard and Alerts.
- Prepare the architecture for a future strategy layer without embedding order execution.

---

# 3. Design Principles

## 3.1 Scoring Is the Final Interpretation Layer

Upstream modules observe and classify market behaviour.

The Signal Scoring Engine shall determine how those observations combine into a tradable setup assessment.

It shall not recalculate:

- Trend
- RVOL
- Structure
- BOS or CHOCH
- Liquidity
- VPA
- Wyckoff
- Institutional Activity

## 3.2 Bullish and Bearish Scores Are Independent

The engine shall calculate bullish and bearish scores separately.

A high bullish score shall not automatically imply a low bearish score.

This allows genuine conflict to remain visible.

## 3.3 Setup Quality Is Not Entry Readiness

A high-quality setup may not yet have a valid trigger.

The engine shall therefore separate:

```text
Setup Quality
```

from:

```text
Entry Readiness
```

## 3.4 Continuation and Reversal Are Distinct

A bullish continuation setup and bullish reversal setup may use different evidence, thresholds, and timing rules.

The engine shall not treat them as interchangeable.

## 3.5 No Trade Is a Valid Output

The engine shall prefer:

```text
No Trade
```

over forcing a directional setup when:

- Evidence is weak.
- Conflict is high.
- Structure is unclear.
- Entry is late.
- Required context is unavailable.
- Risk is excessive.
- Signal quality is below threshold.

## 3.6 Evidence Must Be Explainable

Every score shall be traceable to contributing evidence.

The engine shall expose the strongest supporting and conflicting contributors.

## 3.7 Correlation Shall Be Controlled

Multiple modules may describe the same source event.

The scoring model shall avoid counting correlated observations at full weight.

## 3.8 Historical Signal Events Are Immutable

Once a confirmed signal-state transition is published, it shall not be rewritten.

Later evidence shall produce new transitions.

---

# 4. Core Definitions

## 4.1 Composite Score

A Composite Score is the normalised total of all eligible evidence supporting one direction.

Required composite scores:

- Bullish Composite Score
- Bearish Composite Score

## 4.2 Continuation Score

A Continuation Score measures evidence supporting continuation of the established trend or structure.

## 4.3 Reversal Score

A Reversal Score measures evidence supporting reversal of the established trend or structure.

## 4.4 Setup Quality

Setup Quality measures the structural and contextual quality of a potential trade idea.

It may include:

- Directional evidence
- Structural location
- Liquidity context
- Volume quality
- Wyckoff context
- Institutional alignment
- Conflict
- Risk location
- Signal freshness

## 4.5 Entry Readiness

Entry Readiness measures whether a high-quality setup has a sufficiently confirmed trigger to be actionable.

It may include:

- Confirmed BOS or CHOCH
- Liquidity event
- VPA confirmation
- Retest or Test
- Follow-through
- Price proximity
- Trigger freshness
- Invalidation clarity

## 4.6 Signal Confidence

Signal Confidence measures the reliability and internal agreement of the current signal classification.

It shall not be represented as a guaranteed probability of success.

## 4.7 Directional Dominance

Directional Dominance is the difference between bullish and bearish composite scores.

Suggested formula:

```text
Directional Dominance =
Absolute Value of Bullish Score - Bearish Score
```

Directional sign shall be retained separately.

## 4.8 Conflict Score

Conflict Score measures the extent to which credible bullish and bearish evidence coexist.

## 4.9 No-Trade State

No Trade is a deliberate signal state indicating that the engine does not currently consider either direction sufficiently qualified.

## 4.10 Setup Lifecycle

A setup may progress through:

```text
Unavailable
Neutral
Watch
Qualified
Ready
Triggered
Managing
Invalidated
Expired
Completed
```

The initial indicator release may stop at Triggered or Invalidated.

---

# 5. Scoring Architecture

The Signal Scoring Engine shall use the following pipeline:

```text
Collect Evidence
        │
        ▼
Validate Evidence
        │
        ▼
Remove Duplicates
        │
        ▼
Apply Correlation Controls
        │
        ▼
Apply Source and Category Caps
        │
        ▼
Calculate Directional Scores
        │
        ▼
Calculate Continuation and Reversal Scores
        │
        ▼
Calculate Conflict
        │
        ▼
Calculate Setup Quality
        │
        ▼
Calculate Entry Readiness
        │
        ▼
Resolve Signal State
        │
        ▼
Apply Hysteresis
        │
        ▼
Publish Outputs
```

---

# 6. Inputs

## 6.1 Enable Signal Scoring

Type:

Boolean

Default:

Enabled

When disabled:

- No signal scores shall be calculated.
- Signal state shall be Unavailable.
- Signal visuals shall be hidden.
- Signal alerts shall be disabled.
- Upstream analytical engines may continue operating.

---

## 6.2 Scoring Profile

Type:

Selection

Options:

- Conservative
- Balanced
- Aggressive
- Custom

Suggested production default:

Balanced

Profiles may adjust:

- Module weights
- Minimum confidence
- Dominance thresholds
- Conflict tolerance
- Setup-quality thresholds
- Entry-readiness thresholds
- Persistence requirements

---

## 6.3 Setup Mode

Type:

Selection

Options:

- Continuation Only
- Reversal Only
- Continuation and Reversal
- Automatic

Suggested production default:

Automatic

Automatic mode shall evaluate all supported setup types and select the strongest eligible setup.

---

## 6.4 Evidence Lookback Bars

Type:

Integer

Suggested default:

20

Minimum:

1

Purpose:

Defines the maximum default age for short-lived scoring evidence.

Source lifecycle rules may override this value.

---

## 6.5 Minimum Evidence Confidence

Type:

Integer

Suggested default:

50

Range:

```text
0 to 100
```

Evidence below this threshold shall not contribute unless explicitly permitted.

---

## 6.6 Minimum Bullish Score

Type:

Integer

Suggested default:

60

Purpose:

Minimum bullish composite score for a qualified bullish setup.

---

## 6.7 Minimum Bearish Score

Type:

Integer

Suggested default:

60

Purpose:

Minimum bearish composite score for a qualified bearish setup.

---

## 6.8 Strong Signal Threshold

Type:

Integer

Suggested default:

80

Purpose:

Defines a strong directional setup.

---

## 6.9 Directional Dominance Threshold

Type:

Integer

Suggested default:

15

Purpose:

Minimum score separation required for a directional signal.

---

## 6.10 Maximum Conflict for Directional Setup

Type:

Integer

Suggested default:

40

Purpose:

Directional setups above this conflict value shall be reduced, withheld, or classified as No Trade.

---

## 6.11 Minimum Setup Quality

Type:

Integer

Suggested default:

65

Purpose:

Minimum setup-quality score for a qualified setup.

---

## 6.12 Minimum Entry Readiness

Type:

Integer

Suggested default:

70

Purpose:

Minimum entry-readiness score for Ready state.

---

## 6.13 Minimum Signal Confidence

Type:

Integer

Suggested default:

65

Purpose:

Minimum confidence required to publish a confirmed directional signal.

---

## 6.14 Minimum Confirming Modules

Type:

Integer

Suggested default:

3

Minimum:

1

Purpose:

Defines the number of distinct upstream modules required for a confirmed setup.

---

## 6.15 Require Institutional Alignment

Type:

Boolean

Suggested default:

Enabled for Conservative profile

Suggested Balanced default:

Disabled

When enabled:

- Directional setup shall require Institutional state alignment or minimum Institutional confidence.

---

## 6.16 Require Trend Alignment for Continuation

Type:

Boolean

Suggested default:

Enabled

---

## 6.17 Require Structural Trigger

Type:

Boolean

Suggested default:

Enabled

Purpose:

Requires BOS, CHOCH, accepted break, failed break, or approved structural event before Ready or Triggered state.

---

## 6.18 Require Liquidity Context

Type:

Boolean

Suggested default:

Disabled

When enabled:

- A qualifying liquidity event or meaningful liquidity location shall be required.

---

## 6.19 Require VPA Confirmation

Type:

Boolean

Suggested default:

Disabled

---

## 6.20 Require Wyckoff Context

Type:

Boolean

Suggested default:

Disabled

---

## 6.21 Require Volume Availability

Type:

Boolean

Suggested default:

Disabled

When enabled:

- Valid RVOL or VPA volume context is required for confirmed signals.

---

## 6.22 Entry Trigger Mode

Type:

Selection

Options:

- Structural Confirmation
- Liquidity Rejection
- VPA Confirmation
- Retest Confirmation
- Composite Trigger

Suggested production default:

Composite Trigger

---

## 6.23 Entry Proximity Limit

Type:

Float

Suggested provisional default:

```text
1.0 ATR
```

Purpose:

Prevents a signal from remaining Ready when price has moved too far from the relevant trigger or invalidation level.

---

## 6.24 Maximum Signal Age

Type:

Integer

Suggested default:

10

Purpose:

Defines how long a setup may remain Ready without triggering before it expires.

---

## 6.25 State Confirmation Bars

Type:

Integer

Suggested default:

1

Range:

```text
1 to 5
```

Purpose:

Defines the number of confirmed bars a new state must persist before publication.

---

## 6.26 State Exit Margin

Type:

Integer

Suggested default:

10

Purpose:

Provides hysteresis by allowing an active state to persist until its score falls sufficiently below its entry threshold.

---

## 6.27 Enable No-Trade Filters

Type:

Boolean

Suggested default:

Enabled

---

## 6.28 Show Signal Score

Type:

Boolean

Suggested default:

Enabled

---

## 6.29 Show Setup Quality

Type:

Boolean

Suggested default:

Enabled

---

## 6.30 Show Entry Readiness

Type:

Boolean

Suggested default:

Enabled

---

## 6.31 Show Signal Labels

Type:

Boolean

Suggested default:

Enabled

---

## 6.32 Show Evidence Breakdown

Type:

Boolean

Suggested default:

Disabled

---

## 6.33 Maximum Visible Signal Labels

Type:

Integer

Suggested default:

25

---

## 6.34 Enable Signal Alerts

Type:

Boolean

Suggested default:

Enabled

---

## 6.35 Minimum Alert Confidence

Type:

Integer

Suggested default:

70

---

## 6.36 Alert on State Change Only

Type:

Boolean

Suggested default:

Enabled

---

# 7. Input Validation

The engine shall validate that:

- Evidence Lookback Bars is positive.
- All score and confidence thresholds are between 0 and 100.
- Strong Signal Threshold is not below directional minimum thresholds.
- Directional Dominance Threshold is non-negative.
- Maximum Conflict is between 0 and 100.
- Minimum Confirming Modules is positive.
- Entry Proximity Limit is non-negative.
- Maximum Signal Age is positive.
- State Confirmation Bars is between approved limits.
- State Exit Margin is non-negative.
- Maximum Visible Signal Labels is positive.

Invalid inputs shall not cause runtime failure.

---

# 8. Evidence Sources

The Signal Scoring Engine shall consume evidence from:

- Trend
- RVOL
- Market Structure
- BOS and CHOCH
- Liquidity
- VPA
- Wyckoff
- Institutional Activity

The engine shall use the Shared Evidence Model where practical.

---

# 9. Module Contribution Philosophy

Each module shall contribute only within its area of expertise.

Suggested roles:

| Module | Primary Scoring Role |
|---|---|
| Trend | Direction and continuation context |
| RVOL | Participation quality |
| Market Structure | Structural direction and location |
| BOS/CHOCH | Trigger, continuation, and reversal confirmation |
| Liquidity | Sweep, trap, rejection, and acceptance context |
| VPA | Effort-versus-result and behavioural confirmation |
| Wyckoff | Multi-bar schematic and phase context |
| Institutional | Aggregated higher-level interpretation |

No single module shall independently determine the final signal.

---

# 10. Provisional Module Caps

Suggested maximum contributions:

| Module | Maximum Contribution |
|---|---:|
| Trend | 15 |
| RVOL | 10 |
| Market Structure | 15 |
| BOS and CHOCH | 20 |
| Liquidity | 20 |
| VPA | 20 |
| Wyckoff | 25 |
| Institutional Activity | 25 |

These are contribution limits before final normalisation.

The total theoretical contribution may exceed 100 before normalisation.

Final caps remain an open design decision.

---

# 11. Evidence Categories

Suggested scoring categories:

- Direction
- Structure
- Participation
- Liquidity
- Behaviour
- Sequence
- Institutional
- Continuation
- Reversal
- Trigger
- Risk
- Conflict

Each evidence item may contribute to more than one dimension, subject to correlation controls.

---

# 12. Base Evidence Weights

Evidence weights shall be centrally governed.

Illustrative bullish continuation weights:

| Evidence | Weight |
|---|---:|
| Strong Bullish Trend | 10 |
| Bullish Structure | 8 |
| Higher Low | 6 |
| Bullish BOS | 12 |
| Accepted Buy-Side Break | 14 |
| Strength Confirmation | 12 |
| Sign of Strength | 15 |
| Last Point of Support | 12 |
| Reaccumulation | 12 |
| Markup Confirmation | 15 |

Illustrative bearish continuation weights shall use inverse logic.

Illustrative bullish reversal weights:

| Evidence | Weight |
|---|---:|
| Sell-Side Sweep | 10 |
| Bear Trap | 13 |
| Failed Bearish Break | 12 |
| Bullish CHOCH | 14 |
| Stopping Volume | 10 |
| Bullish Absorption | 12 |
| Selling Climax | 8 |
| Spring | 16 |
| Test after Spring | 13 |
| Bullish Reversal Institutional State | 14 |

Illustrative bearish reversal weights shall use inverse logic.

All weights remain provisional.

---

# 13. Confidence Adjustment

Each contribution shall be adjusted by source confidence.

Suggested formula:

```text
Confidence Factor =
Source Confidence / 100
```

```text
Confidence-Adjusted Contribution =
Base Weight × Confidence Factor
```

Candidate evidence may receive an additional reduction.

---

# 14. Lifecycle Adjustment

Suggested lifecycle multipliers:

| Lifecycle | Multiplier |
|---|---:|
| Candidate | 0.50 |
| Confirmed | 1.00 |
| Reinforced | 1.15 |
| Weakening | 0.60 |
| Failed | 0.00 or opposing contribution |
| Expired | 0.00 |

Final multipliers remain provisional.

---

# 15. Recency Adjustment

Short-lived evidence shall decay over time.

Suggested formula:

```text
Age Factor =
Maximum(0, 1 - Age / Maximum Age)
```

Persistent state evidence may use slower decay or no decay while active.

Examples:

- VPA event: fast decay
- Liquidity Sweep: medium decay
- BOS or CHOCH: medium decay
- Wyckoff Phase: persistent while active
- Trend state: current-state contribution
- Institutional state: persistent while active

---

# 16. Context Adjustment

Evidence contribution may increase when aligned with:

- Higher-timeframe trend
- Major structural level
- High-priority liquidity
- Wyckoff phase
- Institutional state
- Volume confirmation
- Follow-through
- Retest confirmation
- Clear invalidation

Evidence contribution may decrease when:

- Against strong trend
- Mid-range
- Far from trigger
- Volume unavailable
- Gap distorted
- Conflicting structure
- Late or extended
- Repeatedly tested without result

---

# 17. Correlation Control

The engine shall prevent correlated observations from dominating the score.

Example bullish reversal cluster:

```text
Sell-Side Sweep
Bear Trap
Failed Bearish Break
Spring
Stopping Volume
Bullish Absorption
Bullish CHOCH
```

These may all relate to one reversal interaction.

Suggested policy:

1. Highest-confidence event receives full contribution.
2. Second correlated event receives reduced contribution.
3. Remaining events receive minimal contribution or are capped.
4. Different modules still count toward confirming-module requirements.
5. Correlation control shall not remove explainability.

---

# 18. Duplicate Evidence Control

Duplicate evidence shall be identified using:

- Unique event ID
- Source module
- Event type
- Source bar
- Price level
- Correlation group

One confirmed source event shall not add a new contribution on every subsequent bar.

Persistent states shall update through state logic rather than repeated insertion.

---

# 19. Bullish Composite Score

The Bullish Composite Score shall aggregate all eligible bullish evidence.

Conceptual formula:

```text
Raw Bullish Score =
Sum of Adjusted Bullish Contributions
```

Then apply:

- Module caps
- Category caps
- Correlation controls
- Conflict penalties
- Missing-data policy
- Normalisation

Final range:

```text
0 to 100
```

---

# 20. Bearish Composite Score

The Bearish Composite Score shall use inverse logic.

Bullish and bearish scores shall remain independently observable.

---

# 21. Score Normalisation

Possible approaches:

- Fixed theoretical maximum
- Enabled-module maximum
- Dynamic active-evidence maximum
- Hybrid normalisation

Suggested production approach:

```text
Enabled-Module Normalisation
```

This prevents disabled or unavailable modules from automatically lowering all scores.

The engine shall avoid inflating confidence when too few modules are available.

---

# 22. Bullish Continuation Score

Bullish Continuation Score may include:

- Bullish trend
- Bullish structure
- Higher Low
- Bullish BOS
- Accepted Buy-Side Break
- Strength Confirmation
- Sign of Strength
- Last Point of Support
- Reaccumulation
- Bullish Phase D or Phase E
- Markup Confirmation
- Constructive RVOL

---

# 23. Bearish Continuation Score

Bearish Continuation Score shall use inverse logic.

---

# 24. Bullish Reversal Score

Bullish Reversal Score may include:

- Prior bearish trend
- Sell-Side Sweep
- Bear Trap
- Failed Bearish Break
- Bullish CHOCH
- Stopping Volume
- Selling Climax
- Bullish Absorption
- Spring
- Test after Spring
- Accumulation Bias
- Bullish Reversal Institutional state

---

# 25. Bearish Reversal Score

Bearish Reversal Score shall use inverse logic.

---

# 26. Setup-Type Resolution

Supported setup types:

1. None
2. Bullish Continuation
3. Bearish Continuation
4. Bullish Reversal
5. Bearish Reversal
6. Bullish Range Breakout
7. Bearish Range Breakdown
8. Bullish Retest
9. Bearish Retest

The first release may expose only the four primary continuation and reversal types.

The selected setup type shall be the highest-quality eligible setup after conflict resolution.

---

# 27. Conflict Score

Conflict Score may use:

- Minimum of bullish and bearish scores
- Number of opposing modules
- Strength of strongest conflicting evidence
- Contradictory trend and structure
- Continuation versus reversal disagreement
- Institutional Conflict state
- Ambiguous Wyckoff phase

Suggested conceptual calculation:

```text
Conflict Score =
Opposing Evidence Strength
+
Directional Similarity Penalty
+
Module Disagreement Penalty
```

Final range:

```text
0 to 100
```

---

# 28. Conflict Penalty

Directional scores may receive a penalty when strong opposing evidence exists.

Suggested formula:

```text
Adjusted Directional Score =
Composite Score × (1 - Conflict Penalty Factor)
```

The engine shall retain original unadjusted scores for explainability.

---

# 29. Setup Quality

Setup Quality shall measure the quality of the market context independent of immediate entry timing.

Possible components:

| Component | Suggested Weight |
|---|---:|
| Directional evidence | 20 |
| Structural clarity | 15 |
| Liquidity context | 15 |
| VPA quality | 10 |
| Wyckoff context | 10 |
| Institutional alignment | 15 |
| Risk location | 10 |
| Evidence freshness | 5 |

Final result:

```text
0 to 100
```

---

# 30. Structural Clarity

Structural Clarity may be derived from:

- Clear structural direction
- Confirmed BOS or CHOCH
- Valid protection level
- Unambiguous range boundary
- Limited overlapping structure
- No neutral transition conflict

Low Structural Clarity shall reduce Setup Quality.

---

# 31. Liquidity Quality

Liquidity Quality may be derived from:

- High-priority liquidity interaction
- Sweep or trap
- Accepted break
- Clear nearest opposing liquidity
- Distance from unresolved liquidity
- Repeated level defence
- Clean invalidation level

---

# 32. Risk Location

Risk Location shall assess whether the setup occurs near a meaningful invalidation level.

Possible favourable conditions:

- Near structural protection
- Near range boundary
- Near Spring or Upthrust level
- Near sweep extreme
- Near confirmed retest level

Possible unfavourable conditions:

- Mid-range
- Far from invalidation
- After excessive extension
- Near major opposing liquidity
- After multiple late continuation bars

Risk Location is contextual and shall not calculate position size in Version 1.0.

---

# 33. Entry Readiness

Entry Readiness shall measure trigger confirmation.

Possible components:

| Component | Suggested Weight |
|---|---:|
| Structural trigger | 25 |
| Liquidity trigger | 15 |
| VPA trigger | 15 |
| Retest or Test | 15 |
| Follow-through | 10 |
| Price proximity | 10 |
| Trigger freshness | 5 |
| Invalidation clarity | 5 |

Final result:

```text
0 to 100
```

---

# 34. Structural Trigger

Valid structural triggers may include:

- Bullish BOS
- Bearish BOS
- Bullish CHOCH
- Bearish CHOCH
- Accepted range breakout
- Accepted range breakdown
- Failed break reversal
- Protection-level defence followed by confirmation

Trigger eligibility shall depend on setup type.

---

# 35. Liquidity Trigger

Valid liquidity triggers may include:

- Sell-Side Sweep for bullish reversal
- Buy-Side Sweep for bearish reversal
- Bear Trap
- Bull Trap
- Accepted Buy-Side Break for bullish continuation
- Accepted Sell-Side Break for bearish continuation

---

# 36. VPA Trigger

Valid VPA triggers may include:

- Strength Confirmation
- Weakness Confirmation
- Successful Test
- Bullish Absorption
- Bearish Absorption
- No Supply
- No Demand
- Stopping Volume with confirmation

VPA events shall not independently create Ready state when structural confirmation is required.

---

# 37. Retest Trigger

Retest triggers may include:

- Test after Spring
- Test after Upthrust
- Last Point of Support
- Last Point of Supply
- BOS retest
- Range-break retest
- Liquidity-level retest
- Protection-level retest

---

# 38. Follow-Through

Follow-through may strengthen Entry Readiness when:

- Price closes in the expected direction.
- Structure extends.
- Trigger level holds.
- Opposing evidence remains limited.
- RVOL or spread supports the move.
- No immediate failure occurs.

Late follow-through may reduce proximity quality.

---

# 39. Signal Confidence

Signal Confidence shall measure internal agreement and signal reliability.

Possible components:

| Component | Suggested Weight |
|---|---:|
| Setup Quality | 25 |
| Entry Readiness | 20 |
| Directional dominance | 15 |
| Multi-module agreement | 15 |
| Institutional alignment | 10 |
| Evidence confidence | 10 |
| Signal freshness | 5 |

Conflict shall reduce confidence.

---

# 40. Signal Direction

Suggested values:

```text
+1 = Long
 0 = Neutral or No Trade
-1 = Short
```

---

# 41. Signal States

Required primary states:

1. Unavailable
2. Neutral
3. No Trade
4. Bullish Watch
5. Bearish Watch
6. Bullish Qualified
7. Bearish Qualified
8. Long Ready
9. Short Ready
10. Long Triggered
11. Short Triggered
12. Bullish Conflict
13. Bearish Conflict
14. Invalidated
15. Expired

A simplified initial user-facing set may be used while retaining richer internal states.

---

# 42. Unavailable State

Signal state shall be Unavailable when:

- Scoring Engine is disabled.
- Too few upstream modules are available.
- Required strict inputs are unavailable.
- Insufficient history exists.
- Score calculation is not valid.

---

# 43. Neutral State

Neutral shall be used when:

- Neither direction reaches Watch threshold.
- No meaningful setup exists.
- Conflict is low but evidence is insufficient.

---

# 44. No-Trade State

No Trade shall be used when meaningful evidence exists but trading conditions are unsuitable.

Possible reasons:

- High conflict
- Mid-range location
- Excessive extension
- Low structural clarity
- Signal too old
- Price too far from trigger
- Major opposing liquidity nearby
- Missing required confirmation
- Insufficient reward context
- Abnormal gap
- Event duplication
- Rapid state instability

The engine shall expose the primary No-Trade reason.

---

# 45. Watch State

Bullish Watch or Bearish Watch may require:

- Directional score above a lower watch threshold.
- Setup Quality below Qualified threshold or trigger incomplete.
- Directional dominance is present.
- Conflict is acceptable.

Watch shall represent developing opportunity.

---

# 46. Qualified State

Bullish Qualified or Bearish Qualified may require:

- Composite directional score exceeds threshold.
- Setup Quality exceeds minimum.
- Minimum confirming modules are present.
- Conflict is below limit.
- Entry Readiness remains below Ready threshold.

Qualified means the setup is valid but not yet actionable.

---

# 47. Ready State

Long Ready or Short Ready may require:

- Qualified setup exists.
- Entry Readiness exceeds threshold.
- Signal Confidence exceeds threshold.
- Required structural trigger is confirmed.
- Price remains within proximity limit.
- No hard No-Trade filter applies.

---

# 48. Triggered State

Long Triggered or Short Triggered may be published when:

- Ready state existed or equivalent same-bar conditions are met.
- Approved trigger event confirms.
- Signal age remains valid.
- Signal is not duplicated.
- The trigger is published at bar close.

Triggered does not place an order.

It provides a confirmed signal event for alerts and future strategy integration.

---

# 49. Invalidated State

A setup may become Invalidated when:

- Structural invalidation level breaks with acceptance.
- Opposing signal becomes dominant.
- Trigger immediately fails.
- Range or schematic invalidates.
- Required context disappears.
- Signal exceeds allowed risk or distance.
- Hard invalidation event occurs.

Invalidation shall create a separate state event.

---

# 50. Expired State

A setup may expire when:

- Maximum Signal Age is exceeded.
- Entry never becomes Ready.
- Price moves too far from the intended location.
- Evidence decays below threshold.
- A new setup supersedes it.

Expired is not equivalent to Invalidated.

---

# 51. Signal-State Priority

Suggested state-priority order:

1. Unavailable
2. Invalidated
3. Triggered
4. Ready
5. Qualified
6. Conflict
7. No Trade
8. Watch
9. Neutral
10. Expired transition event

Priority is provisional and shall be resolved carefully for same-bar events.

---

# 52. Long Continuation Setup

A Long Continuation setup may require:

- Bullish trend.
- Bullish structure.
- Bullish continuation score above threshold.
- Bullish BOS or accepted breakout.
- Constructive RVOL or VPA.
- No major bearish reversal evidence.
- Acceptable risk location.
- Optional LPS or reaccumulation context.

---

# 53. Short Continuation Setup

Short Continuation shall use inverse logic.

---

# 54. Long Reversal Setup

A Long Reversal setup may require:

- Prior bearish trend or bearish structure.
- Sell-Side Sweep, Bear Trap, Spring, or failed bearish break.
- Bullish CHOCH or equivalent structural transition.
- Bullish reversal score above threshold.
- Bullish VPA or Institutional support.
- Clear invalidation near the reversal extreme.

---

# 55. Short Reversal Setup

Short Reversal shall use inverse logic.

---

# 56. Range Breakout Setup

A bullish range-breakout setup may require:

- Confirmed range.
- Accepted breakout above the range.
- Bullish BOS.
- Constructive participation.
- Strength Confirmation or SOS.
- No immediate Bull Trap.
- Price not excessively extended.

A range-breakdown setup shall use inverse logic.

---

# 57. Retest Setup

A bullish retest setup may require:

- Prior bullish breakout or reversal.
- Pullback to a valid support or trigger level.
- Reduced opposing effort.
- No Supply, successful Test, LPS, or bullish absorption.
- Level holds.
- Bullish trigger confirms.

A bearish retest shall use inverse logic.

---

# 58. No-Trade Filters

Potential hard filters:

- Engine unavailable
- Insufficient history
- Structural state unavailable
- Active Balanced Conflict above limit
- Price in middle of confirmed range
- Excessive distance from invalidation
- Signal older than maximum age
- Opposing score above hard threshold
- Trigger already consumed
- Same-direction duplicate signal
- Gap distortion
- Abnormal spread or zero range
- Required module unavailable
- Daily or session restriction in future versions

---

# 59. Mid-Range Filter

When price is inside a confirmed trading range:

- Long setups near the upper boundary may be penalised.
- Short setups near the lower boundary may be penalised.
- Mid-range signals may be classified as No Trade.
- Spring, Upthrust, LPS, LPSY, SOS, or SOW context may override the general range filter.

---

# 60. Extension Filter

A setup may be considered extended when:

- Price has moved too far from the trigger.
- Multiple expansion bars have already occurred.
- Distance from the relevant MA or structure exceeds limit.
- Nearest invalidation is too distant.
- Major opposing liquidity is near.

Extension shall reduce Setup Quality, Entry Readiness, or create No Trade.

---

# 61. Opposing-Liquidity Filter

A directional setup may be penalised when strong opposing liquidity is too close.

Examples:

- Long setup immediately below high-priority Buy-Side Liquidity
- Short setup immediately above high-priority Sell-Side Liquidity

The engine shall not assume liquidity is always a target rather than resistance or support.

Context shall determine the penalty.

---

# 62. Module Agreement

The engine shall count distinct confirming modules.

Example bullish confirmation:

```text
Trend
Market Structure
Liquidity
VPA
Wyckoff
Institutional
```

Multiple events from one module count as one confirming module unless an approved policy states otherwise.

---

# 63. Missing-Module Handling

When a module is disabled or unavailable:

- Its possible contribution shall be removed from the normalisation base.
- It shall not automatically contribute a negative score.
- Minimum confirming-module requirements shall still apply.
- Strict requirements may force No Trade or Unavailable.
- Confidence may be reduced.

The engine shall not falsely inflate scores from only one available module.

---

# 64. Institutional Integration

The Institutional Activity Engine shall provide:

- Primary institutional state
- Direction
- Confidence
- Bullish score
- Bearish score
- Accumulation score
- Distribution score
- Continuation score
- Reversal score
- Conflict
- Confirming-module count

Institutional output shall be treated as aggregated evidence.

Correlation control shall prevent institutional evidence from fully duplicating all underlying evidence already counted directly.

Suggested policy:

- Institutional state may provide a capped consensus bonus.
- It shall not replace all upstream evidence.
- Its contribution shall be reduced where direct evidence from the same source cluster already dominates.

---

# 65. Confidence Versus Score

The engine shall distinguish:

```text
Directional Score
```

from:

```text
Signal Confidence
```

Example:

```text
Bullish Score: 84
Bearish Score: 52
Signal Confidence: 67
```

This may occur when bullish evidence is strong but meaningful conflict remains.

A high score shall not guarantee high confidence.

---

# 66. State Hysteresis

The engine shall reduce state flicker using:

- Entry threshold
- Lower exit threshold
- State Confirmation Bars
- Directional reversal margin
- Maximum conflict
- Minimum persistence
- Trigger lockout

Example:

```text
Enter Bullish Qualified at 65
Remain Bullish Qualified until score falls below 55
```

Final hysteresis values remain configurable or profile controlled.

---

# 67. Direction-Reversal Policy

A direct Long Ready to Short Ready transition should require stronger evidence than a transition to Neutral or Conflict.

Suggested sequence:

```text
Long Ready
→ Bullish Conflict or No Trade
→ Bearish Qualified
→ Short Ready
```

A same-bar direct reversal may be allowed only after a hard invalidation and strong opposing trigger.

---

# 68. Trigger Lockout

After a signal triggers:

- Duplicate same-direction triggers shall be suppressed for a configured window.
- A new signal may require:
  - New structure
  - New liquidity interaction
  - New setup ID
  - Prior setup completion or invalidation

Suggested input:

```text
Signal Cooldown Bars
```

Final cooldown policy remains open.

---

# 69. Setup Identifier

Each setup should receive a unique conceptual identifier based on:

- Direction
- Setup type
- Creation bar
- Primary structural level
- Source event ID
- Active range ID where applicable

The identifier shall support:

- Duplicate suppression
- Lifecycle tracking
- Alert suppression
- Historical testing
- Future strategy integration

---

# 70. Setup Lifecycle Storage

For each active setup, the engine may retain:

- Setup ID
- Direction
- Setup type
- Creation bar
- Qualification bar
- Ready bar
- Trigger bar
- Invalidation bar
- Expiry bar
- Primary trigger level
- Invalidation level
- Initial score
- Maximum score
- Setup Quality
- Entry Readiness
- Signal Confidence
- Strongest evidence
- Current lifecycle state

Storage shall remain bounded.

---

# 71. Score Persistence

Scores may persist while:

- Evidence remains active.
- Setup remains valid.
- Price remains within proximity limits.
- Structure and context remain compatible.

Scores shall decline when:

- Evidence decays.
- Conflict increases.
- Trigger ages.
- Price becomes extended.
- Required level fails.
- Opposing evidence appears.

---

# 72. Signal Explainability

The engine shall expose at least:

- Strongest supporting evidence
- Second strongest supporting evidence
- Strongest opposing evidence
- Primary setup type
- Primary No-Trade reason
- Confirming-module count
- Directional score difference
- Conflict score
- Missing required context

Example:

```text
Long Ready
Score: 82
Quality: 78
Readiness: 76
Drivers: Bullish BOS, Sell-Side Sweep, Bullish Absorption
Conflict: Buying Climax
```

---

# 73. Outputs

The Signal Scoring Engine shall expose:

- Engine enabled status
- Scoring profile
- Primary signal state
- Signal direction
- Signal confidence
- Bullish composite score
- Bearish composite score
- Bullish continuation score
- Bearish continuation score
- Bullish reversal score
- Bearish reversal score
- Directional dominance
- Conflict score
- Setup type
- Setup Quality
- Entry Readiness
- Confirming-module count
- Strongest supporting evidence
- Second strongest supporting evidence
- Strongest opposing evidence
- Primary No-Trade reason
- Setup ID
- Setup lifecycle state
- Setup creation bar
- Setup qualification bar
- Setup ready bar
- Setup trigger bar
- Setup invalidation level
- Setup trigger level
- Setup age
- Signal freshness
- Last signal-state change
- Last signal-state-change bar
- Sufficient-history status

---

# 74. Suggested Enumerations

## Signal Direction

```text
-1 = Short
 0 = Neutral
+1 = Long
```

## Signal State

```text
 0 = Unavailable
 1 = Neutral
 2 = No Trade
 3 = Bullish Watch
 4 = Bearish Watch
 5 = Bullish Qualified
 6 = Bearish Qualified
 7 = Long Ready
 8 = Short Ready
 9 = Long Triggered
10 = Short Triggered
11 = Bullish Conflict
12 = Bearish Conflict
13 = Invalidated
14 = Expired
15 = Completed
```

## Setup Type

```text
0 = None
1 = Bullish Continuation
2 = Bearish Continuation
3 = Bullish Reversal
4 = Bearish Reversal
5 = Bullish Range Breakout
6 = Bearish Range Breakdown
7 = Bullish Retest
8 = Bearish Retest
```

## Setup Lifecycle

```text
0 = Unavailable
1 = Watching
2 = Qualified
3 = Ready
4 = Triggered
5 = Invalidated
6 = Expired
7 = Completed
```

## No-Trade Reason

```text
 0 = None
 1 = Insufficient Score
 2 = High Conflict
 3 = Weak Structure
 4 = Mid Range
 5 = Extended Price
 6 = Opposing Liquidity
 7 = Missing Trigger
 8 = Missing Required Module
 9 = Signal Too Old
10 = Duplicate Signal
11 = Gap Distortion
12 = Poor Risk Location
13 = Insufficient History
14 = State Instability
```

Internal enumeration values shall remain stable after implementation begins.

---

# 75. Dashboard Integration

The dashboard shall display:

- Signal state
- Signal direction
- Signal confidence
- Bullish score
- Bearish score
- Setup type
- Setup Quality
- Entry Readiness
- Conflict
- Strongest evidence
- No-Trade reason where applicable

Suggested directional setup:

```text
Signal: Long Ready
Type: Bullish Reversal
Bullish: 84
Bearish: 29
Quality: 81
Readiness: 76
Confidence: 79
```

Qualified but not ready:

```text
Signal: Bullish Qualified
Type: Continuation
Quality: 75
Readiness: 54
```

No Trade:

```text
Signal: No Trade
Reason: High Conflict
Bullish: 71
Bearish: 67
```

---

# 76. Chart Visualisation

Optional chart visuals may include:

- Watch marker
- Qualified marker
- Ready marker
- Triggered marker
- Invalidation marker
- Score label
- Setup type
- Confidence
- Trigger level
- Invalidation level
- No-Trade marker

Default visualisation shall emphasise Ready, Triggered, and Invalidated events.

Watch-state labels should be disabled by default to reduce clutter.

---

# 77. Suggested Abbreviations

```text
BW = Bullish Watch
SW = Bearish Watch
BQ = Bullish Qualified
SQ = Bearish Qualified
LR = Long Ready
SR = Short Ready
LT = Long Triggered
ST = Short Triggered
NT = No Trade
INV = Invalidated
EXP = Expired
```

Final abbreviations shall be approved in the Dashboard specification.

---

# 78. Alert Integration

The engine shall support alerts for:

- Bullish Watch
- Bearish Watch
- Bullish Qualified
- Bearish Qualified
- Long Ready
- Short Ready
- Long Triggered
- Short Triggered
- Signal Invalidated
- Signal Expired
- No Trade due to high conflict
- Signal confidence threshold crossed
- Setup Quality threshold crossed
- Entry Readiness threshold crossed

Alerts shall be filtered by:

- Engine enabled state
- Confirmed bar
- Alert state selection
- Minimum confidence
- Minimum setup quality
- Minimum entry readiness
- State-change-only mode
- Duplicate suppression
- Signal cooldown
- Optional setup-type filter

Alert messages shall include:

- Symbol
- Chart timeframe
- Signal state
- Direction
- Setup type
- Bullish score
- Bearish score
- Setup Quality
- Entry Readiness
- Signal Confidence
- Primary trigger
- Invalidation level
- Strongest evidence
- No-Trade reason where applicable
- Timestamp where supported

Example:

```text
EURUSD 15m — Long Ready — Bullish Reversal — Score 83 — Quality 79 — Readiness 75 — Sell-Side Sweep + Bullish CHOCH
```

---

# 79. Repainting Policy

The Signal Scoring Engine shall not intentionally repaint confirmed signal transitions.

Requirements:

- Confirmed upstream evidence shall be used.
- Production state transitions shall occur on confirmed bars.
- Historical Ready, Triggered, Invalidated, and Expired events shall remain immutable.
- Current scores may change as evidence decays or new evidence appears.
- Later state changes shall create new events.
- No historical signal shall move to an earlier bar.
- Developing intrabar signals shall remain provisional.

---

# 80. Real-Time Behaviour

During a developing bar:

- Scores may fluctuate.
- RVOL may change.
- VPA event may change.
- Liquidity interaction may remain provisional.
- Entry Readiness may change.
- Ready or Triggered state may appear and disappear.

Production alerts and confirmed signal states shall use bar-close data by default.

Developing signal state may be shown in debug mode.

---

# 81. Same-Bar Signal Handling

A bar may simultaneously:

- Confirm a liquidity sweep.
- Confirm CHOCH.
- Produce a VPA event.
- Change Institutional state.
- Reach Ready threshold.
- Trigger a signal.

Required processing order:

1. Resolve all upstream modules.
2. Collect evidence.
3. Calculate scores.
4. Calculate Setup Quality.
5. Calculate Entry Readiness.
6. Resolve No-Trade filters.
7. Resolve signal state.
8. Apply hysteresis.
9. Publish state transition.
10. Evaluate alert eligibility.

---

# 82. Same-Bar Ready and Triggered Policy

A setup may move directly to Triggered on the same confirmed bar when:

- All Ready conditions are satisfied.
- The approved trigger occurs on that bar.
- The setup was not previously consumed.
- Same-bar triggering is enabled by policy.

Alternatively, the production policy may require Ready on one bar and Triggered on a later bar.

This remains an open design decision.

---

# 83. Gap Handling

Gap bars may distort:

- Score
- Entry Readiness
- Price proximity
- Structural trigger
- Risk location

A gap may:

- Reduce Setup Quality.
- Reduce Entry Readiness.
- Create No Trade.
- Require post-gap acceptance.
- Allow a valid breakout only after confirmation.

A gap shall not automatically create a Triggered signal.

---

# 84. Zero-Range Handling

When a zero-range bar occurs:

- Price-derived trigger quality may be unavailable.
- VPA contribution may be unavailable.
- Existing setup state may persist if still valid.
- No new trigger shall be generated from invalid measurements.
- No runtime error shall occur.

---

# 85. Object Management

Signal labels and levels shall use bounded collections.

When limits are reached:

- Expired and completed setup objects shall be removed first.
- Old Watch labels shall be removed before Triggered or Invalidated labels.
- Current active setup visuals shall be preserved.
- Internal references shall be removed safely.

---

# 86. Performance Requirements

The engine shall:

- Reuse upstream outputs.
- Avoid analytical recalculation.
- Process bounded evidence collections.
- Maintain a bounded number of active setups.
- Avoid full-history rescanning.
- Calculate strings only when required.
- Skip disabled setup types.
- Skip visual processing when disabled.
- Apply incremental lifecycle updates.
- Maintain deterministic runtime.

Suggested initial maximum:

```text
One primary active bullish setup
One primary active bearish setup
```

The user-facing engine shall resolve these to one primary state.

---

# 87. Error Handling

The engine shall safely handle:

- No evidence
- Missing modules
- Missing volume
- Neutral trend
- Conflicting scores
- Equal bullish and bearish scores
- Duplicate events
- Correlated events
- Zero normalisation base
- Score overflow
- Insufficient history
- Expired setup
- Invalid trigger level
- Missing invalidation level
- Gap bars
- Rapid state changes
- Object-limit pressure
- Symbol changes
- Timeframe changes
- Disabled upstream engines

No edge case shall cause a runtime error.

---

# 88. Testing Requirements

The Signal Scoring Engine shall be tested across:

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

- Strong bullish continuation
- Strong bearish continuation
- Bullish reversal
- Bearish reversal
- Range breakout
- Range breakdown
- Retest setup
- Strong conflict
- Weak evidence
- Missing volume
- Missing Wyckoff
- Institutional conflict
- Mid-range signal
- Extended signal
- Duplicate signal
- Evidence decay
- Ready without trigger
- Same-bar trigger
- Signal invalidation
- Signal expiry
- Gap breakout
- Limited history

---

# 89. Functional Test Cases

## Test SCORE-001 — Bullish Continuation

Given:

- Bullish trend is Strong.
- Bullish structure is confirmed.
- Bullish BOS confirms.
- Strength Confirmation exists.
- Institutional state is Bullish.
- Conflict remains low.

Expected:

- Bullish Continuation score exceeds threshold.
- Bullish Qualified or Long Ready is selected according to Entry Readiness.

---

## Test SCORE-002 — Bearish Continuation

Given inverse conditions.

Expected:

- Bearish Continuation setup is selected.

---

## Test SCORE-003 — Bullish Reversal

Given:

- Prior structure is bearish.
- Sell-Side Sweep occurs.
- Bullish CHOCH confirms.
- Bullish Absorption exists.
- Bullish Reversal score exceeds threshold.

Expected:

- Bullish Reversal setup is selected.
- Bullish continuation setup is not incorrectly selected.

---

## Test SCORE-004 — Bearish Reversal

Given inverse conditions.

Expected:

- Bearish Reversal setup is selected.

---

## Test SCORE-005 — Setup Quality Without Readiness

Given:

- Strong bullish contextual evidence exists.
- Setup Quality equals 80.
- No structural trigger has confirmed.
- Entry Readiness equals 48.

Expected:

- Bullish Qualified is published.
- Long Ready is withheld.

---

## Test SCORE-006 — Long Ready

Given:

- Bullish Qualified setup exists.
- Structural trigger confirms.
- Liquidity and VPA context support the setup.
- Entry Readiness exceeds threshold.
- Signal Confidence exceeds threshold.

Expected:

- Long Ready is published.

---

## Test SCORE-007 — Triggered Signal

Given:

- Long Ready exists.
- Approved bullish trigger confirms.
- Signal remains fresh.
- No duplicate setup exists.

Expected:

- Long Triggered is published once.
- Setup ID remains stable.

---

## Test SCORE-008 — High Conflict

Given:

- Bullish score equals 77.
- Bearish score equals 73.
- Both sides have credible evidence.
- Conflict exceeds the allowed limit.

Expected:

- Directional Ready state is withheld.
- No Trade or Conflict state is published.

---

## Test SCORE-009 — Correlated Evidence

Given:

- Sell-Side Sweep, Spring, Stopping Volume, and Bullish Absorption derive from one interaction.

Expected:

- Correlation controls reduce combined contribution.
- Bullish score remains within approved limits.
- All contributors remain explainable.

---

## Test SCORE-010 — Duplicate Suppression

Given:

- The same Bullish BOS event is provided for several bars.

Expected:

- It contributes once according to lifecycle rules.
- Score does not increase repeatedly.

---

## Test SCORE-011 — Module Cap

Given:

- Multiple strong VPA events are active.
- VPA module cap equals 20.

Expected:

- Total VPA contribution does not exceed 20.

---

## Test SCORE-012 — Missing Module Normalisation

Given:

- Wyckoff is disabled.
- All other required modules are available.
- Strict Wyckoff requirement is disabled.

Expected:

- Score normalisation excludes Wyckoff capacity.
- Signal remains calculable.
- Confidence reflects reduced module diversity.

---

## Test SCORE-013 — Minimum Confirming Modules

Given:

- Bullish score exceeds threshold using one module.
- Minimum Confirming Modules equals 3.

Expected:

- Confirmed directional setup is withheld.

---

## Test SCORE-014 — Mid-Range No Trade

Given:

- A valid range exists.
- Price is near the range midpoint.
- No Spring, Upthrust, SOS, SOW, LPS, or LPSY exists.

Expected:

- No Trade is published with Mid Range reason.

---

## Test SCORE-015 — Extended Signal

Given:

- A bullish setup qualifies.
- Price moves beyond Entry Proximity Limit before trigger.

Expected:

- Entry Readiness declines.
- Ready state is withheld or setup expires.
- No late Triggered signal occurs.

---

## Test SCORE-016 — Opposing Liquidity

Given:

- A Long setup is strong.
- High-priority Buy-Side Liquidity is immediately above.
- No accepted breakout exists.

Expected:

- Setup Quality or Entry Readiness is reduced.
- No Trade may be published according to policy.

---

## Test SCORE-017 — Signal Hysteresis

Given:

- Bullish Qualified is active.
- Bullish score briefly falls below entry threshold but remains above exit threshold.

Expected:

- Bullish Qualified remains active.
- No unnecessary state-change event occurs.

---

## Test SCORE-018 — Signal Invalidation

Given:

- Long Ready is active.
- Price accepts below the invalidation level.

Expected:

- Invalidated event is published.
- Historical Ready event remains unchanged.
- Setup cannot trigger afterward.

---

## Test SCORE-019 — Signal Expiry

Given:

- Bullish Qualified setup remains untriggered beyond Maximum Signal Age.

Expected:

- Expired event is published.
- Setup is removed from active scoring.

---

## Test SCORE-020 — Missing Volume

Given:

- Volume is unavailable.
- Require Volume Availability is disabled.
- Structure, liquidity, and trend evidence remain valid.

Expected:

- Signal remains calculable.
- VPA and RVOL contribution is omitted.
- Confidence may be reduced.

---

## Test SCORE-021 — Strict Volume Requirement

Given:

- Require Volume Availability is enabled.
- Volume is unavailable.

Expected:

- Ready and Triggered states are withheld.
- No Trade or Unavailable is published according to policy.

---

## Test SCORE-022 — Same-Bar Ready and Triggered

Given:

- All setup and trigger conditions confirm on one bar.
- Same-bar triggering is enabled.

Expected:

- Triggered state is published once.
- Ready evidence is retained in metadata.

---

## Test SCORE-023 — Alert Filter

Given:

- Long Ready confirms with confidence 66.
- Minimum Alert Confidence equals 70.

Expected:

- State is stored.
- No alert is triggered.

---

## Test SCORE-024 — Historical Stability

Given:

- A Long Triggered event was confirmed.
- Additional future bars are loaded.

Expected:

- Original setup ID, event bar, state, confidence, score, and trigger remain unchanged.

---

# 90. Acceptance Criteria

The Signal Scoring Engine shall be complete when:

- Bullish and bearish composite scores are calculated independently.
- Continuation and reversal scores are distinct.
- Module and category caps operate correctly.
- Duplicate and correlated evidence are controlled.
- Missing-module normalisation works.
- Conflict Score is available.
- Setup Quality is calculated independently from Entry Readiness.
- Long, Short, Neutral, No Trade, Watch, Qualified, Ready, Triggered, Invalidated, and Expired states are supported.
- No-Trade reasons are exposed.
- Multi-module confirmation operates correctly.
- Signal hysteresis reduces flicker.
- Setup identifiers and lifecycle state are stable.
- Signals expire and invalidate correctly.
- Confirmed signal events remain historically stable.
- Dashboard outputs are available.
- Alerts obey confidence, state, and duplicate filters.
- Evidence and object storage remain bounded.
- TradingView compilation succeeds.
- Required tests pass.

---

# 91. Open Design Decisions

The following items remain unresolved:

1. Final scoring profiles.
2. Final module weights.
3. Final module caps.
4. Final category caps.
5. Final evidence weights.
6. Final confidence multipliers.
7. Final lifecycle multipliers.
8. Final evidence-decay formula.
9. Final source-specific decay periods.
10. Final correlation groups.
11. Final correlated-evidence discount.
12. Final duplicate-evidence logic.
13. Final score-normalisation method.
14. Final minimum directional scores.
15. Final Strong Signal Threshold.
16. Final Directional Dominance Threshold.
17. Final Conflict Score formula.
18. Final conflict penalty.
19. Final Setup Quality formula.
20. Final Entry Readiness formula.
21. Final Signal Confidence formula.
22. Final setup-type priority.
23. Final signal-state priority.
24. Final Watch threshold.
25. Final Qualified threshold.
26. Final Ready threshold.
27. Whether same-bar Ready and Triggered is allowed.
28. Whether Ready must persist for one full bar.
29. Final State Confirmation Bars.
30. Final State Exit Margin.
31. Final direct direction-reversal policy.
32. Final signal cooldown.
33. Final Setup ID design.
34. Whether one bullish and one bearish setup may coexist.
35. Maximum active setups.
36. Maximum stored setup history.
37. Final Mid-Range Filter.
38. Final Entry Proximity Limit.
39. Final extension calculation.
40. Final opposing-liquidity penalty.
41. Whether Institutional alignment is required by default.
42. Whether Wyckoff context is required for reversal setups.
43. Whether volume is required for Strong signals.
44. Whether risk location can hard-block a setup.
45. Whether reward-to-liquidity context is calculated in Version 1.0.
46. Final No-Trade reason priority.
47. Final gap policy.
48. Final setup-expiry policy.
49. Whether expired setups may reactivate.
50. Whether Triggered state remains active after publication or immediately becomes Completed.

---

# 92. Future Enhancements

Possible future enhancements include:

- Strategy order execution
- Stop-loss calculation
- Target calculation
- Reward-to-risk estimation
- Position sizing
- Trade-management states
- Partial exit logic
- Trailing invalidation
- Multi-timeframe scoring
- Higher-timeframe setup filtering
- Session filters
- News-event filters
- Statistical score calibration
- Market-specific scoring profiles
- Adaptive thresholds
- Regime-specific scoring
- Portfolio-level signal filtering
- Cross-asset confirmation
- Performance attribution by evidence source
- Historical signal replay
- Outcome-labelled calibration datasets
- Machine-assisted offline weight optimisation