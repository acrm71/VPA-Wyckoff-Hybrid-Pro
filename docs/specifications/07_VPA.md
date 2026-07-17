# Volume Price Analysis Specification

---

## Document Information

**Document ID:** SPEC-107

**Version:** 0.1 Draft

**Status:** Draft

**Module:** Volume Price Analysis Engine

**Dependencies:**

- Framework
- Settings
- Relative Volume Engine
- Trend Engine
- Market Structure Engine
- BOS and CHOCH Engine
- Liquidity Engine
- Dashboard
- Alerts

**Dependent Modules:**

- Wyckoff
- Institutional Activity
- Signal Scoring

---

# 1. Purpose

The Volume Price Analysis Engine evaluates the relationship between price movement and trading volume on each confirmed bar.

Its primary responsibilities are to analyse:

- Candle spread
- Candle body
- Wick structure
- Close location
- Relative volume
- Volume expansion
- Volume contraction
- Effort versus result
- No Demand
- No Supply
- Stopping Volume
- Buying Climax
- Selling Climax
- Churn
- Absorption
- Test bars
- Strength confirmation
- Weakness confirmation

The engine shall classify descriptive VPA events using a unified evaluation pipeline.

It shall not independently generate trade entries.

---

# 2. Objectives

The VPA Engine shall:

- Evaluate every eligible confirmed bar consistently.
- Use one shared source of Relative Volume.
- Analyse volume in relation to price spread and candle structure.
- Distinguish high-effort/high-result from high-effort/low-result behaviour.
- Classify recognised VPA events without using isolated pattern rules alone.
- Incorporate trend, structure, BOS/CHOCH, and liquidity context.
- Publish one primary event per bar where possible.
- Retain secondary evidence as metadata.
- Produce event-confidence values.
- Avoid intentional historical repainting.
- Operate safely when volume is unavailable.
- Maintain deterministic event-priority rules.
- Keep visual and alert output bounded.

---

# 3. Design Principles

## 3.1 Unified Classification Pipeline

Each eligible bar shall pass through one common sequence:

1. Bar validity
2. Volume availability
3. RVOL analysis
4. Spread analysis
5. Body analysis
6. Close-location analysis
7. Wick analysis
8. Directional analysis
9. Trend context
10. Structure context
11. Liquidity context
12. Candidate-event evaluation
13. Priority resolution
14. Confidence calculation
15. Event publication

## 3.2 Effort and Result

Volume shall represent effort.

Price movement, spread, body, close location, and follow-through shall represent result.

The VPA Engine shall focus on the relationship between the two.

## 3.3 Context Before Interpretation

A candle pattern shall not be interpreted in isolation.

The same candle may have different significance depending on:

- Trend
- Market Structure
- Liquidity location
- Structural event
- Recent volume behaviour
- Price range location

## 3.4 Descriptive, Not Predictive

VPA events shall describe observed behaviour.

For example:

```text
No Demand
```

shall mean that the bar exhibits weak-demand characteristics in context.

It shall not independently mean:

```text
Enter Short
```

## 3.5 One Primary Event

Each analysed bar shall publish at most one primary VPA classification.

Additional compatible observations may be retained as supporting evidence.

## 3.6 Immutable Confirmed Events

Once a confirmed VPA event is published, it shall not be removed or rewritten because of later bars.

Later confirmation or failure shall create separate events or update only non-historical state fields.

## 3.7 Volume Independence

When volume is unavailable:

- Volume-dependent classifications shall be unavailable.
- Price-only metadata may remain available.
- The engine shall not treat missing volume as low volume.
- No runtime failure shall occur.

---

# 4. Core Definitions

## 4.1 Volume

Volume is the raw volume value supplied by the chart symbol.

Depending on market type, it may represent:

- Exchange-traded volume
- Tick volume
- Contract volume
- Broker-provided volume
- Synthetic or limited volume

The engine shall not assume all volume sources are equivalent.

## 4.2 Relative Volume

Relative Volume is supplied by the RVOL Engine.

It compares current volume with a configured baseline.

## 4.3 Spread

Spread is the total range of the candle:

```text
Spread = High - Low
```

## 4.4 Body

Body is the absolute distance between open and close:

```text
Body = Absolute Value of Close - Open
```

## 4.5 Upper Wick

```text
Upper Wick = High - Maximum(Open, Close)
```

## 4.6 Lower Wick

```text
Lower Wick = Minimum(Open, Close) - Low
```

## 4.7 Close Location Value

Close Location Value measures where the close occurs within the candle range.

Suggested normalised form:

```text
CLV = ((Close - Low) - (High - Close)) / (High - Low)
```

Expected range:

```text
-1 to +1
```

Interpretation:

```text
+1 = Close at High
 0 = Close at Midpoint
-1 = Close at Low
```

## 4.8 Effort

Effort is represented primarily by:

- Raw volume
- Relative Volume
- Volume classification
- Volume expansion
- Volume persistence

## 4.9 Result

Result is represented by:

- Spread
- Body
- Directional progress
- Close location
- Break distance
- Follow-through
- Structural effect

## 4.10 Effort Versus Result

Effort Versus Result compares the amount of volume with the amount and quality of price movement produced.

---

# 5. Inputs

## 5.1 Enable VPA Engine

Type:

Boolean

Default:

Enabled

When disabled:

- No VPA events shall be generated.
- VPA visuals shall be hidden.
- VPA alerts shall be disabled.
- VPA scoring contributions shall be removed.
- Dependent modules shall receive unavailable VPA state.

---

## 5.2 Analysis Basis

Type:

Selection

Options:

- Current Bar
- Current Bar with Prior-Bar Context
- Current Bar with Multi-Bar Context

Suggested production default:

Current Bar with Prior-Bar Context

---

## 5.3 Spread Baseline Method

Type:

Selection

Options:

- SMA
- EMA
- Median
- ATR
- Hybrid

Suggested initial implementation:

SMA or ATR

---

## 5.4 Spread Baseline Length

Type:

Integer

Suggested default:

20

Minimum:

2

Purpose:

Defines the lookback used to classify candle spread.

---

## 5.5 Spread Classification Thresholds

Type:

Grouped float inputs

Suggested provisional thresholds:

```text
Very Narrow: < 0.50 × Baseline Spread
Narrow:      < 0.75 × Baseline Spread
Normal:      0.75 to 1.25 × Baseline Spread
Wide:        > 1.25 × Baseline Spread
Very Wide:   > 1.75 × Baseline Spread
```

Final thresholds remain an open design decision.

---

## 5.6 Body Strength Threshold

Type:

Float

Suggested default:

0.60

Purpose:

Defines the minimum body-to-spread ratio for a strong-bodied candle.

```text
Body Strength = Body / Spread
```

---

## 5.7 Narrow Body Threshold

Type:

Float

Suggested default:

0.25

Purpose:

Defines a narrow-body or indecision candle.

---

## 5.8 Close Location Threshold

Type:

Float

Suggested default:

0.60

Purpose:

Defines upper-close and lower-close regions.

Example:

```text
CLV ≥ +0.60 = Close Near High
CLV ≤ -0.60 = Close Near Low
```

---

## 5.9 Long Wick Threshold

Type:

Float

Suggested default:

0.40

Purpose:

Defines the wick-to-spread ratio required for a long wick.

---

## 5.10 Low RVOL Threshold

Type:

Float

Suggested default:

0.70

Purpose:

Defines low relative volume for No Demand, No Supply, and Test candidates.

---

## 5.11 High RVOL Threshold

Type:

Float

Suggested default:

1.50

Purpose:

Defines elevated effort for stopping, climax, churn, and absorption candidates.

---

## 5.12 Extreme RVOL Threshold

Type:

Float

Suggested default:

2.50

Purpose:

Defines extreme participation.

This threshold shall remain configurable.

---

## 5.13 Require Two-Bar Volume Comparison

Type:

Boolean

Suggested default:

Enabled

Purpose:

Allows No Demand and No Supply to require volume lower than one or more prior bars.

---

## 5.14 Prior Volume Comparison Bars

Type:

Integer

Suggested default:

2

Minimum:

1

---

## 5.15 Follow-Through Window

Type:

Integer

Suggested default:

2

Purpose:

Defines the number of bars used for optional event confirmation or failure analysis.

---

## 5.16 Require Context for Advanced Events

Type:

Boolean

Suggested default:

Enabled

Purpose:

When enabled, advanced events such as Stopping Volume and Absorption require structural or liquidity context.

---

## 5.17 Show VPA Labels

Type:

Boolean

Suggested default:

Enabled

---

## 5.18 Show Supporting Evidence

Type:

Boolean

Suggested default:

Disabled

Purpose:

Allows compact labels or tooltips to display supporting evidence.

---

## 5.19 Colour Bars by VPA Event

Type:

Boolean

Suggested default:

Disabled

---

## 5.20 Maximum Visible VPA Labels

Type:

Integer

Suggested default:

50

---

## 5.21 Enable VPA Alerts

Type:

Boolean

Suggested default:

Enabled

---

## 5.22 Minimum Alert Confidence

Type:

Integer

Suggested default:

70

Range:

```text
0 to 100
```

---

# 6. Input Validation

The engine shall validate that:

- Spread Baseline Length is at least 2.
- Body thresholds are between 0 and 1.
- Wick threshold is between 0 and 1.
- Close-location threshold is between 0 and 1.
- RVOL thresholds are non-negative.
- Extreme RVOL is not below High RVOL.
- High RVOL is not below Low RVOL.
- Prior Volume Comparison Bars is at least 1.
- Follow-Through Window is non-negative.
- Maximum visible labels is positive.
- Minimum alert confidence is between 0 and 100.

Invalid settings shall not cause runtime failure.

---

# 7. Bar Eligibility

A bar shall be eligible for full VPA analysis when:

- The VPA Engine is enabled.
- The bar is confirmed under production settings.
- High, low, open, and close are valid.
- Spread is greater than or equal to zero.
- Required baseline history exists.
- Volume is available for volume-dependent classifications.

A zero-spread bar shall be handled safely.

---

# 8. Candle Direction

Suggested directional classification:

```text
+1 = Up Bar
-1 = Down Bar
 0 = Neutral Bar
```

An Up Bar may be defined as:

```text
Close > Open
```

A Down Bar may be defined as:

```text
Close < Open
```

A Neutral Bar may include:

```text
Close = Open
```

Optional close-versus-prior-close direction may be retained separately.

---

# 9. Spread Analysis

The engine shall calculate:

```text
Current Spread = High - Low
```

and compare it with the selected spread baseline.

Suggested spread ratio:

```text
Spread Ratio = Current Spread / Baseline Spread
```

Required spread classes:

- Unavailable
- Very Narrow
- Narrow
- Normal
- Wide
- Very Wide

Spread classification shall be independent from volume classification.

---

# 10. Body Analysis

The engine shall calculate:

```text
Body Ratio = Body / Spread
```

Required body classes:

- Zero Body
- Narrow Body
- Normal Body
- Strong Body

The engine may also retain:

- Bullish body
- Bearish body
- Neutral body

A large spread with a narrow body may indicate rejection, churn, or two-sided activity.

---

# 11. Close-Location Analysis

Required close-location classes:

- Close Near High
- Close Upper Middle
- Close Near Middle
- Close Lower Middle
- Close Near Low

Suggested simplified first implementation:

- High
- Middle
- Low

Close location shall be derived from CLV or equivalent range-normalised logic.

---

# 12. Wick Analysis

The engine shall calculate:

- Upper Wick Ratio
- Lower Wick Ratio
- Dominant Wick
- Wick Imbalance

Suggested formulas:

```text
Upper Wick Ratio = Upper Wick / Spread
Lower Wick Ratio = Lower Wick / Spread
```

Required wick classes may include:

- No Significant Wick
- Long Upper Wick
- Long Lower Wick
- Two-Sided Wicks
- Balanced Wicks

---

# 13. Volume Analysis

The engine shall consume from the RVOL Engine:

- RVOL value
- RVOL percentage
- RVOL classification
- Volume availability state
- Volume expansion state
- Volume contraction state

Required VPA volume classes:

- Unavailable
- Very Low
- Low
- Normal
- High
- Extreme

The VPA Engine shall not independently recalculate the primary RVOL baseline.

---

# 14. Effort Classification

Suggested effort states:

- Unavailable
- Very Low Effort
- Low Effort
- Normal Effort
- High Effort
- Extreme Effort

Effort shall be based primarily on RVOL and may be supplemented by:

- Raw volume rank
- Expansion versus prior bar
- Multi-bar persistence

---

# 15. Result Classification

Suggested result states:

- No Result
- Weak Result
- Normal Result
- Strong Result
- Extreme Result

Result may combine:

- Spread ratio
- Body ratio
- Close location
- Directional progress
- Structural break distance
- Follow-through

The first implementation may use spread and body only.

---

# 16. Effort-versus-Result Matrix

The engine shall support a matrix similar to:

| Effort | Result | Interpretation |
|---|---|---|
| Low | Low | Inactivity or lack of participation |
| Low | High | Efficient movement or thin liquidity |
| High | High | Strong participation with progress |
| High | Low | Absorption, churn, or opposition |
| Extreme | Extreme | Climax or major expansion |
| Extreme | Low | Severe absorption or distribution |

The matrix shall create candidate evidence, not final event classification by itself.

---

# 17. Primary VPA Event Categories

Required primary categories:

1. Unavailable
2. Neutral
3. No Demand
4. No Supply
5. Stopping Volume
6. Buying Climax
7. Selling Climax
8. Absorption
9. Churn
10. Effort Versus Result
11. Test
12. Successful Test
13. Failed Test
14. Strength Confirmation
15. Weakness Confirmation

Some categories may be deferred from the first alpha but shall remain architecturally supported.

---

# 18. No Demand

A No Demand candidate shall generally require:

- Up Bar or upward price progress.
- Narrow or Very Narrow spread.
- Low or Very Low RVOL.
- Volume lower than the configured prior bars where enabled.
- Close not strongly near the high.
- No strong bullish structural confirmation on the same bar.

Context may strengthen the classification when:

- Price is near a Lower High.
- Price is near Buy-Side Liquidity after rejection.
- Trend is bearish.
- Structure is bearish or transitional.
- The bar follows weakness.

No Demand shall represent weak buying participation.

It shall not independently confirm bearish reversal.

---

# 19. No Supply

A No Supply candidate shall generally require:

- Down Bar or downward price progress.
- Narrow or Very Narrow spread.
- Low or Very Low RVOL.
- Volume lower than the configured prior bars where enabled.
- Close not strongly near the low.
- No strong bearish structural confirmation on the same bar.

Context may strengthen the classification when:

- Price is near a Higher Low.
- Price follows a Sell-Side Liquidity Sweep.
- Trend is bullish.
- Structure is bullish or transitional.
- The bar follows strength.

No Supply shall represent weak selling participation.

---

# 20. Stopping Volume

Stopping Volume is a high-effort event where downward movement appears to meet significant opposing demand.

A candidate may require:

- Downward movement or bearish pressure.
- High or Extreme RVOL.
- Wide or Very Wide spread, or unusually high effort.
- Close away from the low.
- Significant lower wick or strong recovery.
- Limited result relative to effort.
- Relevant support, Swing Low, Equal Low, or Sell-Side Liquidity context.

Supporting context may include:

- Sell-Side Liquidity Sweep
- Failed Bearish Break
- Bullish CHOCH
- Prior bearish trend
- Climactic volume expansion

Stopping Volume shall not automatically imply a completed reversal.

---

# 21. Buying Climax

A Buying Climax candidate may require:

- Strong upward movement.
- High or Extreme RVOL.
- Wide or Very Wide spread.
- Extended bullish trend or mature markup context.
- Close off the high, large upper wick, or weak follow-through.
- Buy-Side Liquidity interaction or major structural high.

Buying Climax represents potentially exhaustive buying activity.

It shall remain distinct from healthy bullish expansion.

---

# 22. Selling Climax

A Selling Climax candidate may require:

- Strong downward movement.
- High or Extreme RVOL.
- Wide or Very Wide spread.
- Extended bearish trend or markdown context.
- Close off the low, large lower wick, or weak follow-through.
- Sell-Side Liquidity interaction or major structural low.

Selling Climax represents potentially exhaustive selling activity.

---

# 23. Absorption

Absorption occurs when high effort produces limited directional result because opposing orders absorb aggressive activity.

## 23.1 Bullish Absorption

Possible requirements:

- High or Extreme RVOL.
- Bearish pressure or downward test.
- Narrow or Normal spread relative to effort.
- Close in the upper portion.
- Lower wick or repeated failure to move lower.
- Support, Sell-Side Liquidity, Swing Low, or structural-protection context.

## 23.2 Bearish Absorption

Possible requirements:

- High or Extreme RVOL.
- Bullish pressure or upward test.
- Narrow or Normal spread relative to effort.
- Close in the lower portion.
- Upper wick or repeated failure to move higher.
- Resistance, Buy-Side Liquidity, Swing High, or structural-protection context.

Absorption classification shall require high-effort/low-result evidence.

---

# 24. Churn

Churn represents high trading activity with limited net progress and substantial two-sided participation.

A Churn candidate may require:

- High or Extreme RVOL.
- Narrow or Normal spread relative to volume.
- Narrow body.
- Two-sided wicks or indecisive close.
- Limited directional result.
- No clear dominant rejection side.

Churn shall remain directionally neutral unless context strongly favours accumulation or distribution.

---

# 25. Effort Versus Result Event

An explicit Effort Versus Result event may be published when:

- A major imbalance between effort and result exists.
- No more specific higher-priority VPA classification applies.

Possible subtypes:

- High Effort, Low Result
- Low Effort, High Result
- High Effort, High Result
- Low Effort, Low Result

The subtype shall be retained as metadata.

---

# 26. Test Bar

A Test candidate generally represents a low-volume probe into an area where supply or demand is being assessed.

## 26.1 Bullish Test Candidate

Possible requirements:

- Down Bar or lower intrabar probe.
- Narrow spread.
- Low RVOL.
- Close near the high.
- Lower wick.
- Occurs after strength, stopping volume, spring, or Sell-Side Sweep.
- Does not break meaningful support with acceptance.

## 26.2 Bearish Test Candidate

Possible inverse logic:

- Up Bar or higher probe.
- Narrow spread.
- Low RVOL.
- Close near the low.
- Upper wick.
- Occurs after weakness, upthrust, or Buy-Side Sweep.

The first production version may support bullish tests only if required for Wyckoff compatibility.

---

# 27. Successful Test

A Test may be classified as Successful when:

- The original Test event is confirmed.
- Follow-through occurs in the expected direction within the configured window.
- The tested structural or liquidity level holds.
- No contradictory high-volume break occurs.

A successful bullish test may require:

```text
Higher close or bullish structural progress after the test
```

A successful bearish test may use inverse logic.

The original Test event shall remain immutable.

Successful Test shall be emitted separately or stored as a later status event.

---

# 28. Failed Test

A Test may be classified as Failed when:

- Price breaks through the tested level.
- Follow-through moves against the expected test outcome.
- Volume expands against the test.
- Acceptance occurs beyond the protected level.

A Failed Test shall be published separately.

---

# 29. Strength Confirmation

Strength Confirmation may be used when:

- High effort produces strong bullish result.
- Candle closes near the high.
- Spread and body are strong.
- Bullish BOS or accepted Buy-Side Break occurs.
- Trend and structure support bullish continuation.

This event shall remain distinct from Buying Climax.

The key distinction is:

```text
Strength Confirmation = effort produces sustained bullish result
Buying Climax = effort may indicate exhaustion or poor continuation
```

---

# 30. Weakness Confirmation

Weakness Confirmation may be used when:

- High effort produces strong bearish result.
- Candle closes near the low.
- Spread and body are strong.
- Bearish BOS or accepted Sell-Side Break occurs.
- Trend and structure support bearish continuation.

This event shall remain distinct from Selling Climax.

---

# 31. Event Candidate Evaluation

Each bar may produce multiple candidate events internally.

Example:

```text
High RVOL
Narrow Spread
Long Lower Wick
Sell-Side Sweep
Close Near High
```

Possible candidates:

- Bullish Absorption
- Stopping Volume
- Churn
- Effort Versus Result

A deterministic priority system shall select one primary event.

Supporting candidates may be retained as secondary evidence.

---

# 32. Event Priority

Suggested initial priority order:

1. Selling Climax
2. Buying Climax
3. Stopping Volume
4. Absorption
5. Successful Test
6. Failed Test
7. Test
8. Strength Confirmation
9. Weakness Confirmation
10. No Supply
11. No Demand
12. Churn
13. Effort Versus Result
14. Neutral

This order is provisional.

Priority shall not imply that one event is always more important in trading terms.

It exists only to resolve overlapping classifications.

---

# 33. Primary and Secondary Evidence

The primary event shall be the single published bar classification.

Secondary evidence may include boolean or enumerated flags such as:

- High Effort
- Low Result
- Long Upper Wick
- Long Lower Wick
- Close Near High
- Close Near Low
- Trend Aligned
- Structure Aligned
- At Liquidity
- BOS Confirmed
- CHOCH Confirmed
- Sweep Confirmed
- Acceptance Confirmed
- Volume Unavailable

---

# 34. Trend Integration

The Trend Engine shall provide:

- Trend direction
- Trend strength
- Trend alignment
- HTF confirmation
- Trend transition state

Trend shall affect VPA context and confidence.

Examples:

| VPA Event | Trend Context | Possible Interpretation |
|---|---|---|
| No Demand | Bearish | Continuation weakness |
| No Demand | Bullish | Pullback hesitation |
| No Supply | Bullish | Continuation support |
| No Supply | Bearish | Temporary pause |
| Buying Climax | Extended Bullish | Exhaustion risk |
| Selling Climax | Extended Bearish | Exhaustion risk |
| Strength Confirmation | Bullish | Continuation evidence |
| Weakness Confirmation | Bearish | Continuation evidence |

Trend shall not alter the underlying candle measurements.

---

# 35. Market Structure Integration

The Market Structure Engine shall provide:

- Current structure direction
- Last swing classification
- Active Swing High
- Active Swing Low
- Bullish protection level
- Bearish protection level
- Equal High
- Equal Low

Structure may increase event confidence when the event occurs at a relevant level.

Examples:

- No Supply near a Higher Low
- No Demand near a Lower High
- Stopping Volume near a Lower Low
- Buying Climax near a Higher High
- Absorption at a protection level

---

# 36. BOS and CHOCH Integration

The BOS and CHOCH Engine shall provide:

- BOS events
- CHOCH events
- Wick breaches
- Rejections
- Failed breaks
- Acceptance events
- Event confidence
- Reference-level metadata

Possible combinations:

```text
High RVOL
+
Wide Bullish Spread
+
Close Near High
+
Bullish BOS
=
Strength Confirmation candidate
```

```text
High RVOL
+
Narrow Spread
+
Failed Bullish Break
=
Bearish Absorption or Churn candidate
```

---

# 37. Liquidity Integration

The Liquidity Engine shall provide:

- Buy-Side Sweep
- Sell-Side Sweep
- Accepted liquidity break
- Bull Trap
- Bear Trap
- Nearest liquidity levels
- Liquidity source
- Liquidity event confidence

Possible combinations:

```text
Sell-Side Sweep
+
High RVOL
+
Close Near High
+
Long Lower Wick
=
Stopping Volume or Bullish Absorption
```

```text
Buy-Side Sweep
+
Extreme RVOL
+
Close Near Low
+
Long Upper Wick
=
Buying Climax or Bearish Absorption
```

Liquidity context shall be considered strong supporting evidence.

---

# 38. Relative Volume Integration

The RVOL Engine shall remain the authoritative source for:

- Baseline volume
- RVOL ratio
- RVOL class
- Volume expansion
- Volume contraction
- Volume availability

The VPA Engine shall not maintain a conflicting RVOL calculation.

Raw prior-bar volume comparisons may still be used for textbook No Demand and No Supply rules.

---

# 39. Wyckoff Integration

The VPA Engine shall provide Wyckoff with:

- Primary VPA event
- Event direction
- Event confidence
- Effort state
- Result state
- Spread class
- Close location
- Wick class
- Test status
- Climax status
- Absorption status
- Strength or weakness confirmation
- Liquidity context
- Structural context

Possible Wyckoff mappings:

| VPA Event | Wyckoff Use |
|---|---|
| Selling Climax | Selling Climax candidate |
| Buying Climax | Buying Climax candidate |
| Stopping Volume | Preliminary Support or stopping action |
| No Supply | Test or Last Point of Support evidence |
| No Demand | Last Point of Supply evidence |
| Successful Test | Secondary Test confirmation |
| Strength Confirmation | Sign of Strength |
| Weakness Confirmation | Sign of Weakness |
| Bullish Absorption | Accumulation evidence |
| Bearish Absorption | Distribution evidence |

The Wyckoff Engine shall apply range and phase context.

---

# 40. Institutional Activity Integration

The Institutional Engine may interpret combinations such as:

- High RVOL with low result
- Absorption at major structure
- Climax at liquidity
- Failed BOS on extreme effort
- Successful low-volume test
- Repeated high-volume churn
- Strength after accumulation
- Weakness after distribution

The VPA Engine shall provide measurements and classifications.

It shall not assert actual institutional participation as a fact.

---

# 41. Scoring Integration

Possible scoring effects:

- No Supply may support bullish evidence.
- No Demand may support bearish evidence.
- Stopping Volume may support bullish reversal evidence.
- Buying Climax may support bearish reversal risk.
- Selling Climax may support bullish reversal risk.
- Bullish Absorption may support bullish evidence.
- Bearish Absorption may support bearish evidence.
- Strength Confirmation may support bullish continuation.
- Weakness Confirmation may support bearish continuation.
- Successful Test may receive stronger weight than an unconfirmed Test.
- Churn may reduce directional confidence.
- Unavailable volume shall remove VPA contribution rather than apply a penalty.

Final score values belong to the Signal Scoring specification.

---

# 42. Event Confidence

The VPA Engine shall produce event confidence from:

```text
0 to 100
```

Possible components:

| Component | Suggested Weight |
|---|---:|
| Volume quality | 20 |
| Spread fit | 15 |
| Close-location fit | 15 |
| Wick fit | 10 |
| Trend context | 10 |
| Structure context | 10 |
| Liquidity context | 10 |
| Follow-through or confirmation | 10 |

Weights are provisional.

Confidence shall represent classification quality, not expected profit.

---

# 43. Confidence Penalties

Confidence may be reduced by:

- Missing structural context
- Neutral or conflicting trend
- Weak volume quality
- Marginal thresholds
- Zero or abnormal spread
- Conflicting wick and close evidence
- Gap-driven candle distortion
- Illiquid symbol behaviour
- Multiple equally strong candidate events
- Insufficient history

---

# 44. Event Direction

Suggested directional values:

```text
+1 = Bullish
 0 = Neutral or unavailable
-1 = Bearish
```

Examples:

```text
No Supply = Bullish contextual direction
No Demand = Bearish contextual direction
Bullish Absorption = Bullish
Bearish Absorption = Bearish
Churn = Neutral
Buying Climax = Bearish risk
Selling Climax = Bullish risk
```

Directional value shall remain contextual rather than predictive certainty.

---

# 45. Outputs

The VPA Engine shall expose:

- Engine enabled status
- Volume availability
- Current spread
- Spread ratio
- Spread class
- Body ratio
- Body class
- Close Location Value
- Close-location class
- Upper Wick Ratio
- Lower Wick Ratio
- Wick class
- Candle direction
- Effort state
- Result state
- Effort-versus-result subtype
- Primary VPA event
- Primary event direction
- Primary event confidence
- Supporting evidence flags
- No Demand event
- No Supply event
- Stopping Volume event
- Buying Climax event
- Selling Climax event
- Bullish Absorption event
- Bearish Absorption event
- Churn event
- Test event
- Successful Test event
- Failed Test event
- Strength Confirmation event
- Weakness Confirmation event
- Last VPA event bar index
- Last VPA event price
- Sufficient-history status

---

# 46. Suggested Enumerations

## Spread Class

```text
0 = Unavailable
1 = Very Narrow
2 = Narrow
3 = Normal
4 = Wide
5 = Very Wide
```

## Close Location

```text
0 = Unavailable
1 = Near Low
2 = Lower Middle
3 = Middle
4 = Upper Middle
5 = Near High
```

## Wick Class

```text
0 = Unavailable
1 = No Significant Wick
2 = Long Upper Wick
3 = Long Lower Wick
4 = Two-Sided Wicks
5 = Balanced Wicks
```

## Effort State

```text
0 = Unavailable
1 = Very Low
2 = Low
3 = Normal
4 = High
5 = Extreme
```

## Result State

```text
0 = Unavailable
1 = No Result
2 = Weak
3 = Normal
4 = Strong
5 = Extreme
```

## VPA Event Type

```text
 0 = Unavailable
 1 = Neutral
 2 = No Demand
 3 = No Supply
 4 = Stopping Volume
 5 = Buying Climax
 6 = Selling Climax
 7 = Bullish Absorption
 8 = Bearish Absorption
 9 = Churn
10 = Effort Versus Result
11 = Bullish Test
12 = Bearish Test
13 = Successful Bullish Test
14 = Successful Bearish Test
15 = Failed Bullish Test
16 = Failed Bearish Test
17 = Strength Confirmation
18 = Weakness Confirmation
```

Internal enumeration values shall remain stable after implementation begins.

---

# 47. Dashboard Integration

The dashboard shall be able to display:

- Current VPA event
- Event confidence
- Effort state
- Result state
- Spread class
- RVOL class
- Liquidity context
- Structure context

Suggested output:

```text
VPA: Bullish Absorption
Confidence: 82
Effort: High
Result: Weak
```

For low-volume weakness:

```text
VPA: No Demand
Confidence: 73
```

When volume is unavailable:

```text
VPA: N/A
Volume: Unavailable
```

---

# 48. Chart Visualisation

Optional VPA visuals may include:

- Event labels
- Compact abbreviations
- Bar colouring
- Confidence display
- Supporting evidence tooltip
- Effort-versus-result marker
- Test-status marker

Suggested abbreviations:

```text
ND = No Demand
NS = No Supply
SV = Stopping Volume
BC = Buying Climax
SC = Selling Climax
ABS+ = Bullish Absorption
ABS- = Bearish Absorption
CH = Churn
T = Test
ST = Successful Test
FT = Failed Test
STR = Strength
WEAK = Weakness
```

Final abbreviations shall be approved in the Dashboard and Settings specifications.

---

# 49. Event Placement

VPA labels shall normally be placed on the analysed confirmation bar.

Bullish-context labels may appear below the bar.

Bearish-context labels may appear above the bar.

Neutral labels may appear above or below according to visual-clutter rules.

---

# 50. Alert Integration

The engine shall support alerts for:

- No Demand
- No Supply
- Stopping Volume
- Buying Climax
- Selling Climax
- Bullish Absorption
- Bearish Absorption
- Churn
- Bullish Test
- Bearish Test
- Successful Test
- Failed Test
- Strength Confirmation
- Weakness Confirmation

Alerts shall be filtered by:

- Event enabled state
- Minimum confidence
- Confirmed bar status
- Optional trend alignment
- Optional structure alignment

Alert messages shall include:

- Symbol
- Chart timeframe
- Event type
- Event direction
- Confidence
- RVOL
- Spread class
- Trend context
- Structure context
- Liquidity context
- Timestamp where supported

Example:

```text
ES 15m — Bullish Absorption at Sell-Side Liquidity — RVOL 2.1 — Confidence 84
```

---

# 51. Repainting Policy

The VPA Engine shall not intentionally repaint confirmed events.

Requirements:

- Production events shall use confirmed bars.
- RVOL values shall use approved non-lookahead calculations.
- Confirmed structural and liquidity context shall be used.
- Event history shall remain immutable.
- Follow-through shall produce later confirmation or failure events.
- No historical event shall be moved to an earlier bar.
- Developing intrabar classifications shall remain provisional.

---

# 52. Real-Time Behaviour

During a developing bar:

- Spread changes.
- Body size changes.
- Wick ratios change.
- Close location changes.
- RVOL changes.
- Candidate classification may change.
- Confidence may change.

The production default shall publish confirmed events only after bar close.

Optional developing-state output may be shown in debug mode.

---

# 53. Follow-Through Handling

Some events require later validation.

Examples:

- Test
- Climax
- Stopping Volume
- Strength Confirmation
- Weakness Confirmation

The engine shall not rewrite the original event.

Later bars may generate:

- Successful Test
- Failed Test
- Confirmation
- Failure
- No Follow-Through

A bounded pending-event structure may be used.

---

# 54. Same-Bar Structural Events

A bar may simultaneously produce:

- BOS or CHOCH
- Liquidity Sweep
- VPA event
- RVOL expansion

The VPA Engine shall use already-resolved upstream events according to a deterministic processing order.

Suggested order:

1. RVOL
2. Market Structure
3. BOS/CHOCH
4. Liquidity
5. VPA
6. Wyckoff
7. Institutional
8. Scoring

This order shall be reflected in implementation architecture.

---

# 55. Gap Handling

Gap bars may distort:

- Spread
- Body
- Close location
- Result interpretation

The engine shall retain gap metadata where available.

A gap bar may still produce a VPA event, but confidence may be reduced unless the classification explicitly supports gap behaviour.

A gap shall not automatically be classified as high-result activity without considering the traded intrabar range.

---

# 56. Zero-Range Handling

When:

```text
High = Low
```

the engine shall:

- Avoid division by zero.
- Mark CLV as unavailable or neutral.
- Mark wick ratios as unavailable.
- Retain volume metadata.
- Avoid advanced VPA classification unless an approved zero-range event exists.

---

# 57. Missing-Volume Handling

When volume is unavailable:

- Volume availability shall be Unavailable.
- RVOL shall be Unavailable.
- Volume-dependent events shall not confirm.
- Spread, body, close location, and wick measurements may still be calculated.
- Primary VPA event shall be Unavailable or Price Context Only according to final display policy.
- No low-volume classification shall be inferred.

---

# 58. Object Management

VPA labels shall use bounded collections.

When limits are exceeded:

- The oldest inactive label shall be deleted.
- Current high-confidence events shall remain visible where practical.
- Internal event history may exceed visible history within safe bounds.
- Deleted object references shall be removed safely.

---

# 59. Performance Requirements

The engine shall:

- Reuse RVOL outputs.
- Reuse ATR and structural context from owning modules.
- Avoid repeated baseline calculations.
- Use scalar calculations where possible.
- Use bounded pending-event storage.
- Avoid evaluating disabled event classes where practical.
- Avoid label creation when visuals are disabled.
- Build strings only when required.
- Avoid full-history rescanning.

---

# 60. Error Handling

The engine shall safely handle:

- Missing volume
- Zero spread
- Zero baseline spread
- Insufficient history
- Extreme gaps
- Illiquid symbols
- Abnormal volume spikes
- Conflicting VPA candidates
- Missing structure
- Missing liquidity context
- Neutral trend
- Same-bar BOS and sweep
- Object-limit pressure
- Symbol changes
- Timeframe changes

No edge case shall cause a runtime error.

---

# 61. Testing Requirements

The VPA Engine shall be tested across:

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

- High-volume wide-spread up bar
- High-volume wide-spread down bar
- High-volume narrow-spread bar
- Low-volume narrow up bar
- Low-volume narrow down bar
- Long upper-wick bar
- Long lower-wick bar
- Two-sided churn bar
- Sell-Side Sweep with high volume
- Buy-Side Sweep with high volume
- Bullish BOS with high volume
- Bearish BOS with high volume
- Missing volume
- Zero-range bar
- Gap bar
- Extended bullish trend
- Extended bearish trend
- Sideways range
- Limited history

---

# 62. Functional Test Cases

## Test VPA-001 — Spread Classification

Given:

- Baseline spread equals 10.
- Current spread equals 5.
- Very Narrow threshold equals 0.50.

Expected:

- Spread classification follows the approved boundary policy.
- No division error occurs.

---

## Test VPA-002 — Close Near High

Given:

- High equals 110.
- Low equals 100.
- Close equals 109.

Expected:

- CLV is strongly positive.
- Close Location is Near High.

---

## Test VPA-003 — No Demand

Given:

- Current bar is an Up Bar.
- Spread is Narrow.
- RVOL is Low.
- Volume is below the prior two bars.
- Close is not near the high.
- No bullish structural breakout occurs.

Expected:

- No Demand is selected as the primary event.
- Direction is bearish context.
- Confidence reflects contextual evidence.

---

## Test VPA-004 — No Supply

Given:

- Current bar is a Down Bar.
- Spread is Narrow.
- RVOL is Low.
- Volume is below the prior two bars.
- Close is not near the low.

Expected:

- No Supply is selected as the primary event.

---

## Test VPA-005 — Stopping Volume

Given:

- Price moves lower.
- RVOL is High.
- Lower wick is long.
- Close is near the high.
- A Sell-Side Sweep occurs.

Expected:

- Stopping Volume or Bullish Absorption candidate is generated.
- Priority rules select the approved primary classification.
- Liquidity evidence is retained.

---

## Test VPA-006 — Buying Climax

Given:

- Trend is extended Bullish.
- RVOL is Extreme.
- Spread is Very Wide.
- Price reaches Buy-Side Liquidity.
- Close is materially off the high.

Expected:

- Buying Climax is selected when all approved conditions pass.

---

## Test VPA-007 — Selling Climax

Given:

- Trend is extended Bearish.
- RVOL is Extreme.
- Spread is Very Wide.
- Price reaches Sell-Side Liquidity.
- Close is materially off the low.

Expected:

- Selling Climax is selected.

---

## Test VPA-008 — Bullish Absorption

Given:

- RVOL is High.
- Bearish pressure occurs.
- Spread is Narrow relative to volume.
- Close is in the upper portion.
- Price is at Sell-Side Liquidity.

Expected:

- Bullish Absorption is selected.
- Effort is High.
- Result is Weak.

---

## Test VPA-009 — Bearish Absorption

Given:

- RVOL is High.
- Bullish pressure occurs.
- Spread is Narrow relative to volume.
- Close is in the lower portion.
- Price is at Buy-Side Liquidity.

Expected:

- Bearish Absorption is selected.

---

## Test VPA-010 — Churn

Given:

- RVOL is Extreme.
- Spread is Narrow.
- Body is Narrow.
- Both wicks are significant.
- No clear directional rejection dominates.

Expected:

- Churn is selected as a neutral event.

---

## Test VPA-011 — Strength Confirmation

Given:

- RVOL is High.
- Spread and bullish body are Wide and Strong.
- Close is Near High.
- Bullish BOS occurs.
- Trend is Bullish.

Expected:

- Strength Confirmation is selected.
- Buying Climax is not selected without exhaustion evidence.

---

## Test VPA-012 — Weakness Confirmation

Given:

- RVOL is High.
- Spread and bearish body are Wide and Strong.
- Close is Near Low.
- Bearish BOS occurs.
- Trend is Bearish.

Expected:

- Weakness Confirmation is selected.

---

## Test VPA-013 — Bullish Test

Given:

- A prior bullish strength event exists.
- Current bar probes lower.
- Spread is Narrow.
- RVOL is Low.
- Close is Near High.
- Support remains intact.

Expected:

- Bullish Test is generated.

---

## Test VPA-014 — Successful Test

Given:

- A Bullish Test was previously confirmed.
- Bullish follow-through occurs within the configured window.
- The tested level holds.

Expected:

- Successful Bullish Test is generated separately.
- Original Test event remains unchanged.

---

## Test VPA-015 — Failed Test

Given:

- A Bullish Test was confirmed.
- Price accepts below the tested level within the follow-through window.

Expected:

- Failed Bullish Test is generated separately.

---

## Test VPA-016 — Missing Volume

Given:

- Price data is valid.
- Volume is unavailable.

Expected:

- Volume-dependent VPA event is not generated.
- VPA state is Unavailable or approved equivalent.
- No runtime error occurs.

---

## Test VPA-017 — Zero-Range Bar

Given:

- High equals Low.

Expected:

- No division by zero occurs.
- CLV and wick ratios are safely unavailable.
- Advanced VPA classification is withheld.

---

## Test VPA-018 — Priority Resolution

Given:

- One bar satisfies both Churn and Bearish Absorption candidate rules.
- Bearish rejection context is strong.

Expected:

- Approved priority rules select one primary event.
- The other candidate remains supporting evidence where enabled.

---

## Test VPA-019 — Alert Confidence Filter

Given:

- A VPA event confirms with confidence 65.
- Minimum Alert Confidence equals 70.

Expected:

- Event is stored.
- No alert is triggered.

---

## Test VPA-020 — Historical Stability

Given:

- A VPA event has been confirmed.
- Additional future bars are loaded.

Expected:

- Original event type, confidence, direction, and bar index remain unchanged.

---

# 63. Acceptance Criteria

The VPA Engine shall be complete when:

- Spread, body, close location, and wick measurements are correct.
- RVOL is consumed from the shared RVOL Engine.
- Effort and Result states are produced.
- No Demand and No Supply classifications work.
- Stopping Volume, Climax, Absorption, and Churn classifications work according to approved rules.
- Test lifecycle events operate correctly.
- Strength and Weakness confirmations remain distinct from climax events.
- One primary event is published per bar.
- Supporting evidence remains available.
- Event confidence is produced.
- Missing volume is handled safely.
- Zero-range bars do not cause runtime errors.
- Confirmed events remain historically stable.
- Alerts obey confidence thresholds.
- Object counts remain bounded.
- TradingView compilation succeeds.
- Required tests pass.

---

# 64. Open Design Decisions

The following items remain unresolved:

1. Final spread baseline method.
2. Final spread classification thresholds.
3. Final body-strength thresholds.
4. Final close-location thresholds.
5. Final wick thresholds.
6. Final Low, High, and Extreme RVOL thresholds.
7. Final No Demand rule.
8. Final No Supply rule.
9. Final Stopping Volume rule.
10. Final Buying Climax rule.
11. Final Selling Climax rule.
12. Final Absorption rule.
13. Final Churn rule.
14. Whether bullish and bearish Test events are both included in Version 1.0.
15. Final test follow-through window.
16. Final event-priority order.
17. Final confidence weights.
18. Whether secondary candidate events are stored.
19. Whether gap bars receive dedicated classifications.
20. Whether close-versus-prior-close direction is used.
21. Whether spread uses ATR, average range, or both.
22. Whether event confidence requires follow-through.
23. Whether advanced events require liquidity context by default.
24. Whether Strength and Weakness Confirmation are separate VPA events.
25. Maximum pending-event history.
26. Maximum stored VPA-event history.
27. Whether bar colouring is included in Version 1.0.
28. Whether supporting evidence appears in labels or tooltips.
29. Whether Churn can carry directional bias.
30. Whether volume-source quality affects confidence.

---

# 65. Future Enhancements

Possible future enhancements include:

- Multi-bar VPA sequences
- Background strength and weakness state
- Ultra-high-volume analysis
- Volume dry-up sequences
- Supply and demand trend analysis
- Session-specific volume baselines
- Multi-timeframe VPA
- Statistical event validation
- Adaptive thresholds by volatility regime
- Gap-specific VPA patterns
- Composite operator activity models
- Event clustering
- Automatic VPA narrative
- Volume-profile integration
- Delta-volume integration where available
- Cumulative effort-versus-result analysis