# Institutional Activity Engine Specification

---

## Document Information

**Document ID:** SPEC-109

**Version:** 0.1 Draft

**Status:** Draft

**Module:** Institutional Activity Engine

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
- Shared Evidence Model
- Dashboard
- Alerts

**Dependent Modules:**

- Signal Scoring
- Dashboard
- Alerts

---

# 1. Purpose

The Institutional Activity Engine aggregates observable market evidence and estimates whether current behaviour is consistent with professional or large-participant activity.

The engine shall not claim to directly observe institutional orders, identities, intent, or position size.

Its primary responsibilities are to aggregate evidence relating to:

- Bullish institutional participation
- Bearish institutional participation
- Accumulation
- Distribution
- Absorption
- Expansion
- Continuation
- Reversal
- Level defence
- Liquidity acquisition
- Failed structural movement
- Markup
- Markdown
- Evidence conflict
- Evidence persistence
- Institutional confidence

The engine shall publish one primary institutional interpretation, supporting evidence, directional scores, and confidence.

It shall not independently generate trade entries.

---

# 2. Objectives

The Institutional Activity Engine shall:

- Consume standardised evidence from upstream analytical modules.
- Avoid duplicating upstream detection logic.
- Aggregate bullish and bearish evidence independently.
- Aggregate accumulation and distribution evidence independently.
- Distinguish continuation evidence from reversal evidence.
- Account for event recency, confidence, quality, and context.
- Prevent repeated evidence from being counted excessively.
- Resolve contradictory evidence deterministically.
- Publish one primary institutional state.
- Retain the strongest supporting and conflicting evidence.
- Expose explainable intermediate scores.
- Reduce confidence when upstream data is unavailable.
- Avoid presenting probabilistic inference as confirmed fact.
- Preserve historical confirmed state events.
- Operate within Pine Script performance and storage constraints.
- Provide stable inputs to the Signal Scoring Engine.

---

# 3. Design Principles

## 3.1 Institutional Activity Is an Inference

The engine shall interpret market behaviour as being consistent or inconsistent with institutional participation.

It shall not output factual claims such as:

```text
Institutions are buying
```

Preferred interpretation:

```text
Bullish Institutional Bias
Confidence: 82
```

## 3.2 Evidence Aggregation, Not Pattern Duplication

The engine shall consume outputs from specialised modules rather than reimplement:

- Trend detection
- Swing detection
- BOS and CHOCH detection
- Liquidity Sweep detection
- VPA classification
- Wyckoff phase detection
- RVOL calculation

## 3.3 Directional Scores Remain Independent

Bullish evidence and bearish evidence shall be accumulated independently.

A high bullish score shall not automatically force the bearish score to zero.

This allows the engine to represent genuine conflict.

## 3.4 Evidence Quality Matters

Evidence contribution shall depend on:

- Source confidence
- Source reliability
- Event relevance
- Structural location
- Recency
- Persistence
- Confirmation
- Duplication
- Context alignment

## 3.5 One Primary State

The engine shall publish at most one primary institutional state at a time.

Secondary evidence categories shall remain available as metadata.

## 3.6 Explainability

Every published institutional state shall be traceable to its strongest supporting evidence.

## 3.7 Conservative Conflict Handling

When bullish and bearish evidence are similar, the engine shall publish Neutral, Balanced, or Conflict rather than forcing direction.

## 3.8 Immutable Confirmed State Events

When the institutional state changes and a confirmed state-transition event is published, that historical event shall remain unchanged.

Later bars may publish new transitions.

---

# 4. Core Definitions

## 4.1 Evidence

Evidence is a structured observation supplied by an upstream analytical module.

Each evidence item shall contain, where applicable:

- Source module
- Source event type
- Direction
- Confidence
- Base weight
- Effective weight
- Creation bar
- Confirmation bar
- Price level
- Structural context
- Liquidity context
- Lifecycle state
- Expiry or decay information
- Unique source identifier

## 4.2 Bullish Evidence

Bullish Evidence supports the interpretation that large or informed participation may favour higher prices.

## 4.3 Bearish Evidence

Bearish Evidence supports the interpretation that large or informed participation may favour lower prices.

## 4.4 Accumulation Evidence

Accumulation Evidence supports the interpretation that supply may be absorbed and inventory may be transferring before potential markup.

## 4.5 Distribution Evidence

Distribution Evidence supports the interpretation that demand may be absorbed or supply may be entering before potential markdown.

## 4.6 Continuation Evidence

Continuation Evidence supports directional persistence within the established market trend or structure.

## 4.7 Reversal Evidence

Reversal Evidence supports a potential directional change relative to the preceding trend or structure.

## 4.8 Absorption Evidence

Absorption Evidence indicates high effort with limited result at a meaningful level, consistent with opposing orders absorbing aggressive activity.

## 4.9 Expansion Evidence

Expansion Evidence indicates strong price result supported by participation, structure, and follow-through.

## 4.10 Defence Evidence

Defence Evidence indicates repeated rejection, absorption, or holding behaviour around a meaningful price level.

## 4.11 Conflict

Conflict exists when credible bullish and bearish evidence are both present and neither side has sufficient dominance.

## 4.12 Institutional Confidence

Institutional Confidence represents the quality and dominance of the current institutional interpretation.

It shall not represent a guaranteed probability of future price movement.

---

# 5. Shared Evidence Model

Each upstream module should publish compatible evidence records.

Conceptual evidence structure:

```text
Evidence
{
    Source Module
    Event Type
    Direction
    Category
    Confidence
    Base Weight
    Effective Weight
    Creation Bar
    Confirmation Bar
    Price Level
    Context Flags
    Unique ID
    Lifecycle State
}
```

Pine Script implementation may use:

- Parallel arrays
- User-defined types where supported
- Enumerations
- Bounded event queues
- Scalar current-bar evidence values

The implementation form may differ from the conceptual model, but semantics shall remain consistent.

---

# 6. Evidence Sources

The engine shall consume evidence from:

## 6.1 Trend Engine

Possible evidence:

- Bullish trend
- Bearish trend
- Strong trend
- Trend transition
- Higher-timeframe alignment
- Trend conflict

## 6.2 Relative Volume Engine

Possible evidence:

- High participation
- Extreme participation
- Volume expansion
- Volume contraction
- Unavailable volume

RVOL alone shall remain non-directional.

## 6.3 Market Structure Engine

Possible evidence:

- Higher High
- Higher Low
- Lower High
- Lower Low
- Bullish structure
- Bearish structure
- Equal High
- Equal Low
- Protection-level defence

## 6.4 BOS and CHOCH Engine

Possible evidence:

- Bullish BOS
- Bearish BOS
- Bullish CHOCH
- Bearish CHOCH
- Failed bullish break
- Failed bearish break
- Accepted structural break
- Structural rejection

## 6.5 Liquidity Engine

Possible evidence:

- Sell-Side Liquidity Sweep
- Buy-Side Liquidity Sweep
- Bear Trap
- Bull Trap
- Accepted Buy-Side Break
- Accepted Sell-Side Break
- High-priority liquidity cleared
- Repeated level defence

## 6.6 Volume Price Analysis Engine

Possible evidence:

- Bullish Absorption
- Bearish Absorption
- Stopping Volume
- Buying Climax
- Selling Climax
- No Supply
- No Demand
- Strength Confirmation
- Weakness Confirmation
- Successful Test
- Failed Test
- Churn

## 6.7 Wyckoff Engine

Possible evidence:

- Accumulation
- Distribution
- Reaccumulation
- Redistribution
- Spring
- Upthrust
- Test after Spring
- Test after Upthrust
- Sign of Strength
- Sign of Weakness
- Last Point of Support
- Last Point of Supply
- Phase D
- Phase E
- Failed range breakout
- Failed range breakdown

---

# 7. Inputs

## 7.1 Enable Institutional Engine

Type:

Boolean

Default:

Enabled

When disabled:

- No institutional state shall be calculated.
- Institutional visuals shall be hidden.
- Institutional alerts shall be disabled.
- Institutional scoring contribution shall be removed.
- Dependent modules shall receive unavailable state.

---

## 7.2 Evidence Aggregation Mode

Type:

Selection

Options:

- Weighted Sum
- Weighted Sum with Decay
- Category Consensus
- Hybrid

Suggested production default:

Hybrid

---

## 7.3 Evidence Lookback Bars

Type:

Integer

Suggested default:

20

Minimum:

1

Purpose:

Defines the maximum age of active evidence unless a source-specific lifecycle overrides it.

---

## 7.4 Evidence Decay Mode

Type:

Selection

Options:

- None
- Linear
- Exponential
- Step

Suggested default:

Linear

---

## 7.5 Evidence Half-Life

Type:

Integer

Suggested default:

10

Purpose:

Controls the speed at which older evidence loses influence.

For non-exponential modes, this input may be mapped to an equivalent decay horizon.

---

## 7.6 Minimum Evidence Confidence

Type:

Integer

Suggested default:

50

Range:

```text
0 to 100
```

Evidence below this threshold shall not contribute unless explicitly allowed.

---

## 7.7 Minimum Institutional Confidence

Type:

Integer

Suggested default:

60

Purpose:

Defines the minimum confidence required to publish a directional institutional state.

---

## 7.8 Directional Dominance Threshold

Type:

Integer

Suggested default:

15

Purpose:

Defines the minimum difference between bullish and bearish scores required for directional dominance.

---

## 7.9 Strong Bias Threshold

Type:

Integer

Suggested default:

75

Purpose:

Defines the minimum directional score for a strong institutional state.

---

## 7.10 Conflict Threshold

Type:

Integer

Suggested default:

10

Purpose:

When bullish and bearish scores differ by less than this amount and both are meaningful, Conflict may be published.

---

## 7.11 Maximum Evidence Items

Type:

Integer

Suggested default:

50

Purpose:

Limits active evidence storage.

---

## 7.12 Maximum Evidence Per Source Event

Type:

Integer

Suggested default:

1

Purpose:

Prevents the same source event from contributing repeatedly.

---

## 7.13 Require Multi-Module Confirmation

Type:

Boolean

Suggested default:

Enabled

Purpose:

Requires qualifying evidence from more than one upstream module before a strong institutional state may be published.

---

## 7.14 Minimum Confirming Modules

Type:

Integer

Suggested default:

3

Minimum:

1

---

## 7.15 Require Structural Context

Type:

Boolean

Suggested default:

Enabled

Purpose:

Requires Market Structure, BOS/CHOCH, Liquidity, or Wyckoff evidence for high-confidence directional states.

---

## 7.16 Require Volume Context

Type:

Boolean

Suggested default:

Disabled

Purpose:

When enabled, strong institutional classifications require valid RVOL or VPA volume evidence.

Basic inference shall remain available without volume unless strict mode is enabled.

---

## 7.17 Include Trend Evidence

Type:

Boolean

Suggested default:

Enabled

---

## 7.18 Include Structure Evidence

Type:

Boolean

Suggested default:

Enabled

---

## 7.19 Include Liquidity Evidence

Type:

Boolean

Suggested default:

Enabled

---

## 7.20 Include VPA Evidence

Type:

Boolean

Suggested default:

Enabled

---

## 7.21 Include Wyckoff Evidence

Type:

Boolean

Suggested default:

Enabled

---

## 7.22 Show Institutional State

Type:

Boolean

Suggested default:

Enabled

---

## 7.23 Show Evidence Labels

Type:

Boolean

Suggested default:

Disabled

---

## 7.24 Show Confidence

Type:

Boolean

Suggested default:

Enabled

---

## 7.25 Show Conflict State

Type:

Boolean

Suggested default:

Enabled

---

## 7.26 Maximum Visible State Labels

Type:

Integer

Suggested default:

20

---

## 7.27 Enable Institutional Alerts

Type:

Boolean

Suggested default:

Enabled

---

## 7.28 Minimum Alert Confidence

Type:

Integer

Suggested default:

70

Range:

```text
0 to 100
```

---

# 8. Input Validation

The engine shall validate that:

- Evidence Lookback Bars is positive.
- Evidence Half-Life is positive.
- Confidence thresholds are between 0 and 100.
- Directional Dominance Threshold is non-negative.
- Conflict Threshold is non-negative.
- Maximum Evidence Items is positive.
- Maximum Evidence Per Source Event is positive.
- Minimum Confirming Modules is positive.
- Maximum Visible State Labels is positive.
- Strong Bias Threshold is not below Minimum Institutional Confidence.

Invalid settings shall not cause runtime failure.

---

# 9. Evidence Categories

Each evidence item shall belong to one or more categories.

Suggested categories:

```text
Directional
Accumulation
Distribution
Continuation
Reversal
Absorption
Expansion
Defence
Exhaustion
Conflict
Participation
```

One evidence item may contribute to multiple score dimensions.

Example:

```text
Spring
```

may contribute to:

- Bullish
- Accumulation
- Reversal
- Liquidity
- Defence

---

# 10. Evidence Direction

Suggested directional values:

```text
+1 = Bullish
 0 = Neutral
-1 = Bearish
```

Non-directional evidence such as high RVOL shall use direction zero until combined with contextual evidence.

---

# 11. Base Evidence Weight

Each source event shall have a configurable or internally defined Base Weight.

Suggested provisional bullish weights:

| Evidence | Weight |
|---|---:|
| Bullish Trend | 6 |
| Strong Bullish Trend | 10 |
| Higher Low | 6 |
| Bullish BOS | 10 |
| Bullish CHOCH | 12 |
| Failed Bearish Break | 13 |
| Sell-Side Sweep | 12 |
| Bear Trap | 15 |
| Bullish Absorption | 15 |
| Stopping Volume | 12 |
| Selling Climax | 10 |
| No Supply | 7 |
| Successful Bullish Test | 11 |
| Spring | 18 |
| Test after Spring | 14 |
| Sign of Strength | 17 |
| Last Point of Support | 15 |
| Accumulation | 16 |
| Reaccumulation | 14 |
| Phase D Bullish | 12 |
| Phase E Bullish | 16 |

Suggested provisional bearish weights:

| Evidence | Weight |
|---|---:|
| Bearish Trend | 6 |
| Strong Bearish Trend | 10 |
| Lower High | 6 |
| Bearish BOS | 10 |
| Bearish CHOCH | 12 |
| Failed Bullish Break | 13 |
| Buy-Side Sweep | 12 |
| Bull Trap | 15 |
| Bearish Absorption | 15 |
| Buying Climax | 10 |
| No Demand | 7 |
| Successful Bearish Test | 11 |
| Upthrust | 18 |
| Test after Upthrust | 14 |
| Sign of Weakness | 17 |
| Last Point of Supply | 15 |
| Distribution | 16 |
| Redistribution | 14 |
| Phase D Bearish | 12 |
| Phase E Bearish | 16 |

Weights remain provisional and belong to later calibration.

---

# 12. Confidence Scaling

Effective evidence weight shall be scaled by source confidence.

Suggested formula:

```text
Confidence Factor = Source Confidence / 100
```

```text
Confidence-Adjusted Weight =
Base Weight × Confidence Factor
```

Evidence below Minimum Evidence Confidence may contribute zero.

---

# 13. Recency Scaling

Evidence weight may decay with age.

Suggested linear model:

```text
Age Factor =
Maximum(0, 1 - Evidence Age / Evidence Lookback Bars)
```

Suggested exponential concept:

```text
Age Factor =
0.5 ^ (Evidence Age / Half-Life)
```

Final formula remains an open design decision.

Persistent states such as confirmed Wyckoff Phase E may use slower decay than isolated bar events.

---

# 14. Context Scaling

Evidence may receive a context multiplier.

Possible context multipliers:

- Major structural level
- Higher-timeframe alignment
- High-priority liquidity
- Confirmed range boundary
- Trend alignment
- Trend conflict
- Volume confirmation
- Follow-through
- Failed follow-through
- Repeated defence
- Gap distortion

Suggested effective-weight formula:

```text
Effective Weight =
Base Weight
× Confidence Factor
× Age Factor
× Context Factor
× Uniqueness Factor
```

---

# 15. Evidence Uniqueness

Evidence shall not be counted repeatedly when multiple modules describe the same underlying market event.

Example:

```text
Sell-Side Sweep
+
Spring
+
Stopping Volume
```

may all derive from the same source interaction.

All may contribute, but correlation controls shall prevent triple-counting at full weight.

Possible approaches:

- Source-event group cap
- Category cap
- Shared unique ID
- Correlation discount
- Highest-weight-only rule
- Hybrid contribution rule

Suggested first implementation:

- Retain all evidence.
- Apply full weight to the strongest item.
- Apply reduced weight to correlated secondary items.

---

# 16. Correlated Evidence Groups

Suggested correlation groups:

## 16.1 Bullish Reversal Group

- Sell-Side Sweep
- Bear Trap
- Failed Bearish Break
- Spring
- Stopping Volume
- Selling Climax
- Bullish Absorption
- Bullish CHOCH

## 16.2 Bearish Reversal Group

- Buy-Side Sweep
- Bull Trap
- Failed Bullish Break
- Upthrust
- Buying Climax
- Bearish Absorption
- Bearish CHOCH

## 16.3 Bullish Continuation Group

- Bullish BOS
- Accepted Buy-Side Break
- Strength Confirmation
- Sign of Strength
- Last Point of Support
- Phase D Bullish
- Phase E Bullish

## 16.4 Bearish Continuation Group

- Bearish BOS
- Accepted Sell-Side Break
- Weakness Confirmation
- Sign of Weakness
- Last Point of Supply
- Phase D Bearish
- Phase E Bearish

---

# 17. Category Caps

To prevent one category from dominating the entire engine, contribution caps may be applied.

Example provisional caps:

| Category | Maximum Contribution |
|---|---:|
| Trend | 15 |
| Structure | 20 |
| Liquidity | 25 |
| VPA | 25 |
| Wyckoff | 30 |
| RVOL Participation | 10 |

Total scores may then be normalised to:

```text
0 to 100
```

Final caps remain an open decision.

---

# 18. Bullish Evidence Score

The engine shall calculate a Bullish Evidence Score.

Suggested conceptual formula:

```text
Bullish Score =
Sum of Effective Bullish Evidence
-
Applicable Contradiction Penalties
```

The result shall be clamped or normalised to:

```text
0 to 100
```

---

# 19. Bearish Evidence Score

The Bearish Evidence Score shall use equivalent logic.

Bullish and bearish scores shall be calculated independently before conflict resolution.

---

# 20. Accumulation Score

The Accumulation Score may include:

- Sell-Side Sweep
- Bear Trap
- Spring
- Test after Spring
- Stopping Volume
- Selling Climax
- Bullish Absorption
- No Supply
- Accumulation schematic
- Bullish Phase C
- Bullish Phase D
- Sign of Strength
- Last Point of Support
- Repeated support defence

---

# 21. Distribution Score

The Distribution Score may include:

- Buy-Side Sweep
- Bull Trap
- Upthrust
- Test after Upthrust
- Buying Climax
- Bearish Absorption
- No Demand
- Distribution schematic
- Bearish Phase C
- Bearish Phase D
- Sign of Weakness
- Last Point of Supply
- Repeated resistance defence

---

# 22. Continuation Score

Bullish Continuation evidence may include:

- Strong Bullish Trend
- Bullish BOS
- Accepted Buy-Side Break
- Strength Confirmation
- Sign of Strength
- Last Point of Support
- Reaccumulation
- Bullish Phase E

Bearish Continuation evidence shall use inverse logic.

---

# 23. Reversal Score

Bullish Reversal evidence may include:

- Prior bearish trend
- Sell-Side Sweep
- Bear Trap
- Bullish CHOCH
- Stopping Volume
- Selling Climax
- Spring
- Successful Test

Bearish Reversal evidence shall use inverse logic.

---

# 24. Absorption Score

Bullish Absorption evidence may include:

- High RVOL with limited downside result
- Bullish Absorption VPA event
- Repeated Sell-Side rejection
- Protection-level defence
- Spring
- Stopping Volume
- No Supply after strength

Bearish Absorption shall use inverse logic.

---

# 25. Expansion Score

Bullish Expansion evidence may include:

- High RVOL
- Wide bullish spread
- Close near high
- Bullish BOS
- Accepted breakout
- Strength Confirmation
- Sign of Strength
- Follow-through

Bearish Expansion shall use inverse logic.

---

# 26. Institutional State Categories

Required primary states:

1. Unavailable
2. Neutral
3. Balanced Conflict
4. Bullish Institutional Bias
5. Strong Bullish Institutional Bias
6. Bearish Institutional Bias
7. Strong Bearish Institutional Bias
8. Accumulation Bias
9. Distribution Bias
10. Bullish Absorption Bias
11. Bearish Absorption Bias
12. Markup Confirmation
13. Markdown Confirmation
14. Bullish Reversal Evidence
15. Bearish Reversal Evidence

The final production list may be simplified.

---

# 27. Primary State Resolution

The engine shall select one primary state using deterministic priority.

Suggested resolution order:

1. Unavailable
2. Balanced Conflict
3. Markup Confirmation
4. Markdown Confirmation
5. Accumulation Bias
6. Distribution Bias
7. Bullish Absorption Bias
8. Bearish Absorption Bias
9. Bullish Reversal Evidence
10. Bearish Reversal Evidence
11. Strong Bullish Institutional Bias
12. Strong Bearish Institutional Bias
13. Bullish Institutional Bias
14. Bearish Institutional Bias
15. Neutral

This order is provisional.

Specific states shall require category-specific conditions, not score magnitude alone.

---

# 28. Unavailable State

Institutional state shall be Unavailable when:

- Engine is disabled.
- Insufficient upstream history exists.
- Too few enabled modules provide valid evidence.
- Required strict context is unavailable.
- No valid evidence model can be calculated.

Missing volume alone shall not necessarily force Unavailable.

---

# 29. Neutral State

Neutral shall be published when:

- Bullish evidence is weak.
- Bearish evidence is weak.
- No specialised state qualifies.
- Confidence is below the directional threshold.

---

# 30. Balanced Conflict State

Balanced Conflict may be published when:

- Bullish and bearish scores are both meaningful.
- Score difference is below Conflict Threshold.
- Supporting modules disagree.
- No higher-priority state has sufficient confirmation.

Balanced Conflict shall reduce downstream directional confidence.

---

# 31. Bullish Institutional Bias

A Bullish Institutional Bias may require:

- Bullish score exceeds Minimum Institutional Confidence.
- Bullish score exceeds bearish score by Directional Dominance Threshold.
- Minimum confirming-module count is met.
- No stronger specialised state applies.

---

# 32. Strong Bullish Institutional Bias

A Strong Bullish Institutional Bias may require:

- Bullish score exceeds Strong Bias Threshold.
- Bullish dominance is clear.
- Structural or Wyckoff confirmation exists.
- Evidence is not concentrated in one weak source.
- Conflict score is low.

---

# 33. Bearish Institutional Bias

Bearish states shall use inverse logic.

---

# 34. Accumulation Bias

Accumulation Bias may require:

- Accumulation Score exceeds threshold.
- Bullish evidence exceeds bearish evidence.
- At least one structural or liquidity reversal event exists.
- At least one VPA or Wyckoff accumulation event exists.
- Prior bearish or neutral context is plausible.
- Markup is not yet fully confirmed.

Possible evidence:

- Spring
- Sell-Side Sweep
- Bullish Absorption
- Stopping Volume
- Successful Test
- Accumulation schematic
- Phase C or early Phase D

---

# 35. Distribution Bias

Distribution Bias shall use inverse logic.

Possible evidence:

- Upthrust
- Buy-Side Sweep
- Bearish Absorption
- Buying Climax
- Successful bearish Test
- Distribution schematic
- Phase C or early Phase D

---

# 36. Bullish Absorption Bias

Bullish Absorption Bias may require:

- Bullish Absorption Score exceeds threshold.
- High effort with limited downside result exists.
- Price is at a meaningful structural or liquidity level.
- Downside acceptance is absent.
- Repeated defence may be present.
- Bullish expansion is not yet confirmed.

---

# 37. Bearish Absorption Bias

Bearish Absorption Bias shall use inverse logic.

---

# 38. Markup Confirmation

Markup Confirmation may require:

- Bullish continuation score is high.
- Bullish BOS or accepted range breakout exists.
- Strength Confirmation or Sign of Strength exists.
- Trend aligns bullish.
- Phase D or Phase E accumulation/reaccumulation context exists.
- Bearish conflict is limited.

Markup Confirmation shall represent confirmed bullish expansion rather than early accumulation.

---

# 39. Markdown Confirmation

Markdown Confirmation shall use inverse logic.

---

# 40. Bullish Reversal Evidence

Bullish Reversal Evidence may require:

- Prior bearish trend or structure.
- Bullish reversal score exceeds threshold.
- Bullish CHOCH, Spring, Bear Trap, or failed bearish break exists.
- Full bullish continuation is not yet confirmed.

---

# 41. Bearish Reversal Evidence

Bearish Reversal Evidence shall use inverse logic.

---

# 42. Conflict Resolution

Conflict resolution shall consider:

- Bullish score
- Bearish score
- Score difference
- Evidence count
- Confirming-module count
- Highest-confidence evidence
- Evidence recency
- Structural hierarchy
- Wyckoff phase
- Trend direction
- Continuation versus reversal context

The engine shall not simply subtract bearish evidence from bullish evidence and discard the losing side.

Both sides shall remain available.

---

# 43. Structural Hierarchy

When evidence conflicts, higher-level evidence may receive priority.

Suggested hierarchy:

1. Confirmed Wyckoff range departure and Phase E
2. Accepted structural break with follow-through
3. Confirmed Spring or Upthrust sequence
4. CHOCH with structural follow-through
5. Liquidity Sweep
6. VPA bar classification
7. Isolated trend or swing evidence

This hierarchy remains provisional.

---

# 44. Continuation Versus Reversal Conflict

A common conflict occurs when:

- Trend remains bullish.
- A Buy-Side Sweep or Buying Climax appears.
- Structure has not yet reversed.

In this case:

- Bearish reversal evidence may increase.
- Bullish continuation evidence may remain active.
- The primary state may remain Bullish Institutional Bias with reduced confidence.
- Balanced Conflict may be published if evidence becomes sufficiently close.
- Bearish bias shall require stronger structural confirmation.

Equivalent inverse handling shall apply in bearish trends.

---

# 45. Evidence Persistence

Different evidence types shall persist for different durations.

Possible persistence classes:

- Bar Event
- Short-Lived Event
- Structural Event
- Range Event
- Phase State
- Persistent Context

Suggested examples:

| Evidence | Persistence |
|---|---|
| No Demand | Short |
| Liquidity Sweep | Short to Medium |
| CHOCH | Medium |
| BOS | Medium |
| Spring | Medium |
| Wyckoff Phase D | Persistent while active |
| Phase E | Persistent while accepted |
| Trend | Persistent current state |

---

# 46. Evidence Expiry

Evidence may expire when:

- Age exceeds source-specific limit.
- Source level becomes invalid.
- Opposing acceptance supersedes it.
- Range completes or invalidates.
- A new state transition replaces the prior context.
- Maximum storage is reached.

Expiry shall remove current influence but shall not rewrite historical event records.

---

# 47. Repeated Level Defence

Repeated defence may be inferred when:

- Multiple interactions occur near the same level.
- Price repeatedly rejects beyond the level.
- High effort produces limited result.
- The level remains structurally relevant.
- Acceptance through the level does not occur.

Repeated defence may increase:

- Accumulation score at support
- Distribution score at resistance
- Absorption score
- Institutional confidence

Duplicate interactions from the same bar shall not count separately.

---

# 48. Participation Quality

Participation quality may be evaluated from:

- RVOL
- Volume expansion
- Persistence of elevated RVOL
- VPA event quality
- Spread result
- Follow-through
- Volume availability quality

High RVOL without directional context shall not create bullish or bearish institutional bias.

---

# 49. Follow-Through

Evidence strength may increase when expected follow-through appears.

Examples:

- Spring followed by Test and SOS
- Upthrust followed by Test and SOW
- Bullish BOS followed by higher acceptance
- Bearish BOS followed by lower acceptance
- Absorption followed by directional expansion

Failure of expected follow-through may reduce or expire evidence.

The original upstream event shall remain unchanged.

---

# 50. Evidence Confirmation Levels

Suggested confirmation states:

```text
0 = Unavailable
1 = Candidate
2 = Confirmed
3 = Reinforced
4 = Weakening
5 = Failed
6 = Expired
```

Candidate evidence may receive reduced weight.

Confirmed or Reinforced evidence may receive full or enhanced weight.

Failed evidence may contribute to the opposing side where appropriate.

---

# 51. Institutional Confidence

Institutional Confidence shall use:

```text
0 to 100
```

Possible components:

| Component | Suggested Weight |
|---|---:|
| Dominant directional score | 25 |
| Score separation | 20 |
| Multi-module agreement | 15 |
| Structural confirmation | 15 |
| Volume and VPA quality | 10 |
| Wyckoff context | 10 |
| Recency and follow-through | 5 |

Weights remain provisional.

---

# 52. Confidence Penalties

Confidence may be reduced by:

- Strong opposing evidence
- Evidence concentration in one module
- Missing structural context
- Missing volume where required
- Old evidence
- Correlated duplicate evidence
- Neutral trend
- Ambiguous Wyckoff phase
- Failed follow-through
- Gap distortion
- Insufficient history
- Rapid state oscillation

---

# 53. State Hysteresis

To reduce state flicker, state transitions should use hysteresis.

Possible rules:

- A new state must exceed the current state by a minimum score margin.
- A state must persist for a configured number of confirmed bars.
- Strong states require a lower exit threshold than entry threshold.
- One-bar weak contradictions shall not reverse the state.

Suggested input:

```text
State Confirmation Bars = 1 to 3
```

Final hysteresis policy remains open.

---

# 54. State Transition Events

The engine may emit events when institutional state changes.

Examples:

- Neutral to Bullish Bias
- Bullish Bias to Strong Bullish Bias
- Bullish Bias to Conflict
- Conflict to Bearish Bias
- Accumulation Bias to Markup Confirmation
- Distribution Bias to Markdown Confirmation
- Institutional state invalidated

State transitions shall be confirmed at bar close by default.

---

# 55. Upstream Processing Order

The engine shall consume upstream outputs after they have been resolved.

Required processing order:

1. Trend
2. RVOL
3. Market Structure
4. BOS and CHOCH
5. Liquidity
6. VPA
7. Wyckoff
8. Institutional Activity
9. Signal Scoring
10. Dashboard
11. Alerts

---

# 56. Trend Integration

Trend evidence shall influence:

- Continuation versus reversal interpretation
- Directional score
- Context multipliers
- State confidence
- Markup or Markdown confirmation

Trend alone shall not create an institutional state stronger than basic bias.

---

# 57. Market Structure Integration

Market Structure shall provide:

- Structural direction
- Swing classifications
- Protection levels
- Equal levels
- Structural transitions
- Repeated level defence context

Structure shall help determine whether institutional evidence has produced meaningful price effect.

---

# 58. BOS and CHOCH Integration

BOS and CHOCH shall provide:

- Directional break events
- Reversal evidence
- Continuation evidence
- Acceptance
- Rejection
- Failure
- Structural confidence

CHOCH may strengthen reversal evidence.

BOS may strengthen continuation evidence.

Neither shall prove institutional participation independently.

---

# 59. Liquidity Integration

Liquidity shall provide:

- Sweeps
- Accepted breaks
- Traps
- Level priority
- Repeated liquidity tests
- Level-cleared state

Liquidity evidence shall be strongest when aligned with:

- Structure
- VPA
- Wyckoff
- Follow-through

---

# 60. VPA Integration

VPA shall provide:

- Effort-versus-result evidence
- Absorption
- Climax
- Stopping activity
- No Demand
- No Supply
- Strength
- Weakness
- Tests
- Churn

VPA events shall provide behavioural interpretation but require structural context for high confidence.

---

# 61. Wyckoff Integration

Wyckoff shall provide:

- Schematic
- Phase
- Range context
- Ordered event sequence
- Spring
- Upthrust
- SOS
- SOW
- LPS
- LPSY
- Confidence

Wyckoff evidence may receive higher persistence because it represents multi-bar sequence context.

---

# 62. Scoring Integration

The Signal Scoring Engine shall consume:

- Primary institutional state
- Institutional direction
- Institutional confidence
- Bullish score
- Bearish score
- Accumulation score
- Distribution score
- Continuation score
- Reversal score
- Conflict score
- Confirming-module count
- Strongest evidence
- Evidence freshness
- State-transition event

The Institutional Engine shall remain an input to scoring rather than the final trade-decision authority.

---

# 63. Outputs

The Institutional Activity Engine shall expose:

- Engine enabled status
- Evidence model available status
- Primary institutional state
- Institutional direction
- Institutional confidence
- Bullish evidence score
- Bearish evidence score
- Bullish-bearish score difference
- Accumulation score
- Distribution score
- Bullish continuation score
- Bearish continuation score
- Bullish reversal score
- Bearish reversal score
- Bullish absorption score
- Bearish absorption score
- Bullish expansion score
- Bearish expansion score
- Conflict score
- Confirming-module count
- Active evidence count
- Strongest supporting evidence
- Second strongest supporting evidence
- Strongest conflicting evidence
- Last institutional-state change
- Last state-change bar
- State persistence bars
- Evidence freshness
- Volume context available
- Structural context available
- Wyckoff context available
- Sufficient-history status

---

# 64. Suggested Enumerations

## Institutional State

```text
 0 = Unavailable
 1 = Neutral
 2 = Balanced Conflict
 3 = Bullish Institutional Bias
 4 = Strong Bullish Institutional Bias
 5 = Bearish Institutional Bias
 6 = Strong Bearish Institutional Bias
 7 = Accumulation Bias
 8 = Distribution Bias
 9 = Bullish Absorption Bias
10 = Bearish Absorption Bias
11 = Markup Confirmation
12 = Markdown Confirmation
13 = Bullish Reversal Evidence
14 = Bearish Reversal Evidence
```

## Evidence Category

```text
 0 = Unavailable
 1 = Directional
 2 = Accumulation
 3 = Distribution
 4 = Continuation
 5 = Reversal
 6 = Absorption
 7 = Expansion
 8 = Defence
 9 = Exhaustion
10 = Participation
11 = Conflict
```

## Evidence Source

```text
0 = Unavailable
1 = Trend
2 = RVOL
3 = Market Structure
4 = BOS and CHOCH
5 = Liquidity
6 = VPA
7 = Wyckoff
```

## Evidence Lifecycle

```text
0 = Unavailable
1 = Candidate
2 = Confirmed
3 = Reinforced
4 = Weakening
5 = Failed
6 = Expired
```

Internal enumeration values shall remain stable after implementation begins.

---

# 65. Dashboard Integration

The dashboard shall be able to display:

- Primary institutional state
- Direction
- Confidence
- Bullish score
- Bearish score
- Conflict status
- Strongest evidence
- Confirming-module count

Suggested output:

```text
Institutional: Accumulation Bias
Confidence: 81
Bullish: 84
Bearish: 28
```

Markup example:

```text
Institutional: Markup Confirmation
Confidence: 87
Evidence: SOS + Bullish BOS
```

Conflict example:

```text
Institutional: Balanced Conflict
Bullish: 68
Bearish: 64
```

Unavailable example:

```text
Institutional: N/A
```

---

# 66. Chart Visualisation

Optional institutional visuals may include:

- State-transition labels
- Background state shading
- Confidence markers
- Accumulation and Distribution markers
- Markup and Markdown markers
- Conflict markers
- Strongest-evidence annotations

Default visualisation shall remain minimal.

The engine shall not place labels for every evidence item by default.

---

# 67. Suggested Abbreviations

```text
IB+ = Bullish Institutional Bias
IB- = Bearish Institutional Bias
SIB+ = Strong Bullish Institutional Bias
SIB- = Strong Bearish Institutional Bias
ACC = Accumulation Bias
DIST = Distribution Bias
ABS+ = Bullish Absorption Bias
ABS- = Bearish Absorption Bias
MU = Markup Confirmation
MD = Markdown Confirmation
REV+ = Bullish Reversal Evidence
REV- = Bearish Reversal Evidence
CON = Balanced Conflict
```

Final abbreviations shall be approved in the Dashboard and Settings specifications.

---

# 68. Alert Integration

The engine shall support alerts for:

- Bullish Institutional Bias
- Strong Bullish Institutional Bias
- Bearish Institutional Bias
- Strong Bearish Institutional Bias
- Accumulation Bias
- Distribution Bias
- Bullish Absorption Bias
- Bearish Absorption Bias
- Markup Confirmation
- Markdown Confirmation
- Bullish Reversal Evidence
- Bearish Reversal Evidence
- Balanced Conflict
- Institutional state change
- Institutional confidence threshold crossed

Alerts shall be filtered by:

- Engine enabled state
- Confirmed bar
- Minimum confidence
- Minimum confirming modules
- Optional state persistence requirement
- Optional score-dominance requirement
- State-change-only mode

Alert messages shall include:

- Symbol
- Chart timeframe
- Institutional state
- Direction
- Confidence
- Bullish score
- Bearish score
- Strongest evidence
- Confirming-module count
- Wyckoff context where available
- Timestamp where supported

Example:

```text
AAPL 1H — Accumulation Bias — Confidence 82 — Spring, Bullish Absorption, Bullish CHOCH
```

---

# 69. Repainting Policy

The Institutional Activity Engine shall not intentionally repaint confirmed state-transition events.

Requirements:

- Confirmed upstream events shall be used.
- Production state transitions shall occur on confirmed bars.
- Historical state-change events shall remain immutable.
- Evidence weight may decay on later bars.
- Current state may evolve as new evidence arrives.
- Later state changes shall create new transition events.
- No historical state event shall be moved to an earlier bar.
- Developing intrabar state shall remain provisional.

---

# 70. Real-Time Behaviour

During a developing bar:

- RVOL changes.
- VPA candidates change.
- Liquidity interaction may remain provisional.
- Evidence scores may fluctuate.
- Institutional state may appear to change.
- Confidence may fluctuate.

Production state publication and alerts shall use confirmed bars by default.

Developing state may appear in debug mode.

---

# 71. Historical Stability

Once a state transition is confirmed:

- Transition bar shall remain unchanged.
- Published state shall remain unchanged.
- Confirmation confidence shall remain stored.
- Strongest evidence at confirmation shall remain stored where practical.
- Later evidence shall not rewrite the event.

Current-state values may naturally differ from historical transition values.

---

# 72. Duplicate Evidence Suppression

The engine shall identify evidence duplication using:

- Source module
- Event type
- Source bar
- Source level
- Unique event ID
- Correlation group

One source event shall not repeatedly contribute on every subsequent bar.

Persistent state evidence shall update through lifecycle logic rather than duplicate insertion.

---

# 73. Missing Data Handling

When one upstream module is unavailable:

- Its evidence contribution shall be omitted.
- No negative penalty shall be applied solely for absence unless strict mode requires it.
- Confirming-module count shall reflect available modules.
- Confidence may be reduced.
- The engine may remain operational if minimum evidence requirements are satisfied.

When too many critical modules are unavailable:

```text
Institutional State = Unavailable
```

---

# 74. Gap Handling

Gap bars may produce distorted apparent expansion or reversal.

The engine shall reduce confidence where:

- Price gaps across a structural or liquidity level.
- No traded wick interaction exists.
- VPA spread interpretation is unreliable.
- Acceptance remains unconfirmed.

Gap-specific upstream metadata shall be retained where available.

---

# 75. Rapid State Reversal Handling

Rapid alternating state transitions may indicate:

- Volatility
- Ambiguous range behaviour
- Insufficient hysteresis
- Poor evidence calibration
- Conflicting timeframes

The engine shall apply:

- State hysteresis
- Minimum confidence
- Dominance threshold
- Optional persistence bars
- Conflict state

to reduce flicker.

---

# 76. Object Management

Institutional state labels and markers shall use bounded collections.

When limits are reached:

- Oldest inactive transition labels shall be deleted first.
- Current state visual shall be preserved.
- Historical metadata may remain deeper than visible objects.
- Internal references shall be removed safely.

---

# 77. Performance Requirements

The engine shall:

- Consume upstream values without recalculating them.
- Use bounded evidence arrays.
- Avoid full-history rescanning.
- Process only new or active evidence.
- Apply incremental score updates where practical.
- Limit string construction.
- Avoid drawing objects when visuals are disabled.
- Use category caps efficiently.
- Remove expired evidence safely.
- Maintain deterministic runtime.

---

# 78. Error Handling

The engine shall safely handle:

- No evidence
- Missing volume
- Missing Wyckoff context
- Neutral trend
- Conflicting evidence
- Duplicate evidence
- Expired evidence
- Correlated evidence
- Zero score
- Score overflow
- Insufficient history
- Gap bars
- Rapid state changes
- Object-limit pressure
- Symbol changes
- Timeframe changes
- Disabled upstream modules

No edge case shall cause a runtime error.

---

# 79. Testing Requirements

The Institutional Activity Engine shall be tested across:

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
- Accumulation
- Distribution
- Reaccumulation
- Redistribution
- Bullish absorption
- Bearish absorption
- Balanced conflict
- Missing volume
- Missing Wyckoff state
- One-module evidence only
- Multi-module confirmation
- Correlated evidence
- Duplicate evidence
- Evidence decay
- State hysteresis
- Gap breakout
- Limited history

---

# 80. Functional Test Cases

## Test INST-001 — Bullish Evidence Aggregation

Given:

- Bullish BOS is confirmed.
- Sell-Side Sweep is confirmed.
- Bullish Absorption is confirmed.
- Bullish score exceeds threshold.
- Bearish score remains low.
- Minimum confirming modules are present.

Expected:

- Bullish Institutional Bias is published.
- Confidence exceeds the minimum threshold.
- Supporting evidence is retained.

---

## Test INST-002 — Bearish Evidence Aggregation

Given:

- Bearish BOS is confirmed.
- Buy-Side Sweep is confirmed.
- Bearish Absorption is confirmed.
- Distribution context exists.

Expected:

- Bearish Institutional Bias or Distribution Bias is published according to priority rules.

---

## Test INST-003 — Accumulation Bias

Given:

- Sell-Side Sweep occurs.
- Spring is confirmed.
- Bullish Absorption occurs.
- Test after Spring succeeds.
- Markup is not yet confirmed.

Expected:

- Accumulation Bias is the primary state.
- Accumulation score is elevated.
- Markup Confirmation is withheld.

---

## Test INST-004 — Distribution Bias

Given:

- Buy-Side Sweep occurs.
- Upthrust is confirmed.
- Bearish Absorption occurs.
- Test after Upthrust succeeds.

Expected:

- Distribution Bias is published.

---

## Test INST-005 — Markup Confirmation

Given:

- Accumulation or Reaccumulation context exists.
- Sign of Strength is confirmed.
- Bullish BOS and accepted breakout exist.
- Bullish trend aligns.
- Follow-through persists.

Expected:

- Markup Confirmation is published.
- Bullish continuation score is high.

---

## Test INST-006 — Markdown Confirmation

Given:

- Distribution or Redistribution context exists.
- Sign of Weakness is confirmed.
- Bearish BOS and accepted breakdown exist.

Expected:

- Markdown Confirmation is published.

---

## Test INST-007 — Balanced Conflict

Given:

- Bullish score equals 72.
- Bearish score equals 68.
- Both sides have credible multi-module evidence.
- Difference is below Conflict Threshold.

Expected:

- Balanced Conflict is published.
- Neither bullish nor bearish primary bias is forced.

---

## Test INST-008 — Multi-Module Requirement

Given:

- Strong evidence is present from only one module.
- Require Multi-Module Confirmation is enabled.
- Minimum Confirming Modules equals 3.

Expected:

- Strong institutional state is withheld.
- Neutral or lower-confidence state is published.

---

## Test INST-009 — Duplicate Evidence Suppression

Given:

- One Sell-Side Sweep remains active for several bars.
- The same unique event ID is received repeatedly.

Expected:

- It contributes once according to lifecycle rules.
- Bullish score does not increase repeatedly.

---

## Test INST-010 — Correlated Evidence Discount

Given:

- Sell-Side Sweep, Spring, and Stopping Volume all arise from one source interaction.

Expected:

- The strongest event receives full contribution.
- Correlated evidence receives reduced contribution or group-capped contribution.
- Total score remains within approved limits.

---

## Test INST-011 — Evidence Decay

Given:

- A bullish event is confirmed.
- No reinforcing evidence appears.
- Evidence age increases beyond the configured horizon.

Expected:

- Effective bullish contribution declines.
- Historical event remains stored.
- Current institutional state may weaken.

---

## Test INST-012 — Repeated Defence

Given:

- Price repeatedly tests the same support area.
- High effort produces limited downside result.
- Acceptance below does not occur.

Expected:

- Bullish defence or absorption contribution increases within approved caps.
- Duplicate same-bar interactions are ignored.

---

## Test INST-013 — Bullish Reversal Evidence

Given:

- Prior trend is bearish.
- Sell-Side Sweep occurs.
- Bullish CHOCH confirms.
- Spring or Bullish Absorption is present.
- Bullish continuation is not yet established.

Expected:

- Bullish Reversal Evidence is published.
- Markup Confirmation is withheld.

---

## Test INST-014 — Continuation Versus Reversal Conflict

Given:

- Trend remains strongly bullish.
- Buying Climax and Buy-Side Sweep occur.
- No bearish CHOCH exists.

Expected:

- Bearish reversal score increases.
- Strong bearish institutional state is withheld.
- Current bullish state weakens or Conflict is published.

---

## Test INST-015 — Missing Volume

Given:

- Volume is unavailable.
- Bullish structure, liquidity, and Wyckoff evidence remain valid.
- Require Volume Context is disabled.

Expected:

- Institutional state remains calculable.
- Confidence is reduced where appropriate.
- No runtime error occurs.

---

## Test INST-016 — Strict Volume Requirement

Given:

- Require Volume Context is enabled.
- Volume and VPA evidence are unavailable.

Expected:

- Strong institutional state is withheld.
- State becomes Neutral or Unavailable according to minimum-evidence policy.

---

## Test INST-017 — State Hysteresis

Given:

- Bullish Bias is active.
- One weak bearish bar temporarily narrows the score difference.
- Bearish evidence does not exceed transition requirements.

Expected:

- Bullish Bias remains active.
- No unnecessary state-change event occurs.

---

## Test INST-018 — State Transition

Given:

- Bullish Bias is active.
- Strong bearish evidence accumulates.
- Bearish score exceeds bullish score by the required margin.
- Confirmation requirements are met.

Expected:

- Bearish state transition occurs once.
- Historical bullish transition remains stored.

---

## Test INST-019 — Alert Filter

Given:

- Accumulation Bias confirms with confidence 66.
- Minimum Alert Confidence equals 70.

Expected:

- State is stored.
- No alert is triggered.

---

## Test INST-020 — Historical Stability

Given:

- Markup Confirmation was published.
- Additional future bars are loaded.

Expected:

- Original event bar, state, confidence, and supporting evidence remain unchanged.

---

# 81. Acceptance Criteria

The Institutional Activity Engine shall be complete when:

- Standardised upstream evidence can be consumed.
- Bullish and bearish evidence scores are calculated independently.
- Accumulation and Distribution scores are available.
- Continuation and Reversal scores are available.
- Absorption and Expansion scores are available.
- Duplicate evidence is suppressed.
- Correlated evidence is discounted or capped.
- Evidence decay operates correctly.
- Conflict resolution is deterministic.
- One primary state is published.
- Strong supporting and conflicting evidence remain available.
- Multi-module confirmation works.
- State hysteresis reduces flicker.
- Missing modules are handled safely.
- Confirmed state-transition history remains stable.
- Dashboard outputs are available.
- Alerts obey confidence and persistence filters.
- Object and evidence storage remain bounded.
- TradingView compilation succeeds.
- Required tests pass.

---

# 82. Open Design Decisions

The following items remain unresolved:

1. Final evidence aggregation mode.
2. Final shared evidence implementation structure.
3. Final evidence categories.
4. Final base weights.
5. Final category caps.
6. Final source-module caps.
7. Final confidence scaling formula.
8. Final evidence-decay formula.
9. Final decay horizons by evidence type.
10. Final context multipliers.
11. Final correlation groups.
12. Final correlated-evidence discount.
13. Final duplicate-evidence identifier.
14. Final Bullish and Bearish score normalisation.
15. Final Accumulation and Distribution thresholds.
16. Final Continuation and Reversal thresholds.
17. Final Absorption and Expansion thresholds.
18. Final directional dominance threshold.
19. Final conflict threshold.
20. Final Strong Bias threshold.
21. Final minimum confirming-module count.
22. Whether strong states require Wyckoff evidence.
23. Whether strong states require volume evidence.
24. Final state-priority order.
25. Final state hysteresis policy.
26. Final state confirmation bars.
27. Whether current trend receives persistent evidence weight.
28. Whether Phase E evidence decays.
29. Whether failed evidence contributes to the opposing direction.
30. Whether repeated defence has its own primary state.
31. Whether Markup and Markdown require Wyckoff context.
32. Whether Accumulation and Distribution can be published without a confirmed range.
33. Maximum active evidence items.
34. Maximum stored state-transition history.
35. Maximum supporting evidence exposed to Dashboard.
36. Gap confidence penalty.
37. Missing-volume confidence penalty.
38. Missing-Wyckoff confidence penalty.
39. Whether Neutral and Balanced Conflict remain separate states.
40. Whether institutional states are displayed intrabar in debug mode.

---

# 83. Future Enhancements

Possible future enhancements include:

- Multi-timeframe institutional evidence
- Evidence clustering by price zone
- Institutional activity heatmaps
- Separate internal and external structure evidence
- Order-block evidence integration
- Fair-value-gap evidence integration
- Volume-profile evidence
- Market-profile evidence
- Delta and footprint evidence where available
- Session-aware institutional behaviour
- Dynamic evidence calibration
- Statistical outcome validation
- Adaptive source weighting
- Evidence attribution reports
- Historical state replay
- Regime-specific institutional models
- Machine-assisted offline weight calibration
- Cross-asset confirmation