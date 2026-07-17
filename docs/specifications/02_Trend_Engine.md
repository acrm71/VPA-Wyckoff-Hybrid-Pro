# Trend Engine Specification

---

## Document Information

**Document ID:** SPEC-102

**Version:** 0.1 Draft

**Status:** Draft

**Module:** Trend Engine

**Dependencies:**

- Framework
- Settings
- Dashboard
- Scoring
- Alerts

---

# 1. Purpose

The Trend Engine determines the directional condition, strength, alignment, and maturity of the current market trend.

It shall provide a consistent trend context for all downstream analytical modules, including:

- Market Structure
- Break of Structure and Change of Character
- Volume Price Analysis
- Wyckoff Analysis
- Institutional Activity
- Signal Scoring
- Dashboard
- Alerts

The Trend Engine shall not generate standalone trade entries.

Its purpose is to classify the market environment and provide directional context.

---

# 2. Objectives

The Trend Engine shall:

- Identify bullish, bearish, and neutral market conditions.
- Measure trend strength.
- Evaluate moving-average alignment.
- Evaluate moving-average slope.
- Evaluate price location relative to trend references.
- Support higher-timeframe confirmation.
- Detect potential trend transitions.
- Provide stable, non-repainting outputs wherever technically possible.
- Expose clearly defined outputs to other modules.

---

# 3. Design Principles

The Trend Engine shall follow these principles:

## 3.1 Context Before Signal

Trend state shall be treated as market context rather than an automatic trading signal.

## 3.2 Confirmed Information

The engine shall prioritise confirmed bar data.

## 3.3 Multi-Factor Classification

Trend classification shall not rely on a single moving-average crossover.

## 3.4 Configurability

Users shall be able to adjust trend sensitivity without modifying source code.

## 3.5 Market Independence

The logic shall operate consistently across supported markets and timeframes.

## 3.6 Controlled Complexity

The engine shall remain interpretable and avoid unnecessary indicators or duplicated calculations.

---

# 4. Trend Model

The initial Trend Engine shall use a three-layer moving-average model:

1. Fast Trend Average
2. Medium Trend Average
3. Slow Trend Average

Default periods shall be defined in the Settings specification.

Suggested initial defaults:

- Fast: 20
- Medium: 50
- Slow: 200

These defaults remain provisional until the Settings specification is approved.

The engine shall support the following moving-average types:

- Exponential Moving Average
- Simple Moving Average
- Weighted Moving Average
- Hull Moving Average
- Running Moving Average

The initial production default shall be Exponential Moving Average.

---

# 5. Inputs

The Trend Engine shall support the following user inputs.

## 5.1 Enable Trend Engine

Type:

Boolean

Default:

Enabled

Purpose:

Enables or disables Trend Engine calculations and visual output.

Disabling the Trend Engine shall also disable its contribution to scoring.

---

## 5.2 Moving-Average Type

Type:

Selection

Options:

- EMA
- SMA
- WMA
- HMA
- RMA

Default:

EMA

---

## 5.3 Fast Length

Type:

Integer

Suggested default:

20

Minimum:

1

---

## 5.4 Medium Length

Type:

Integer

Suggested default:

50

Minimum:

2

---

## 5.5 Slow Length

Type:

Integer

Suggested default:

200

Minimum:

3

---

## 5.6 Slope Lookback

Type:

Integer

Purpose:

Defines the number of bars used to evaluate moving-average slope.

Suggested default:

5

---

## 5.7 Minimum Slope Threshold

Type:

Float

Purpose:

Prevents insignificant moving-average changes from being classified as meaningful trend slope.

The final implementation may normalise slope using:

- Price
- Percentage change
- Average True Range
- Instrument tick size

The selected normalisation method shall be documented before implementation.

---

## 5.8 Higher-Timeframe Confirmation

Type:

Boolean

Default:

Enabled

Purpose:

Allows the Trend Engine to compare the chart trend with a configurable higher timeframe.

---

## 5.9 Higher Timeframe

Type:

Timeframe selection

Suggested default:

Automatic

Supported modes:

- Automatic
- User selected

---

## 5.10 Require Higher-Timeframe Alignment

Type:

Boolean

Default:

Disabled

Purpose:

When enabled, bullish or bearish trend approval shall require higher-timeframe agreement.

When disabled, higher-timeframe trend remains contextual rather than mandatory.

---

## 5.11 Show Trend Averages

Type:

Boolean

Default:

Enabled

---

## 5.12 Show Trend Background

Type:

Boolean

Default:

Disabled

---

## 5.13 Show Transition Markers

Type:

Boolean

Default:

Enabled

---

# 6. Input Validation

The engine shall validate that:

- Fast Length is less than Medium Length.
- Medium Length is less than Slow Length.
- All lengths are positive.
- Slope Lookback is positive.
- Higher timeframe is not lower than the chart timeframe when higher-timeframe confirmation is enabled.

Where invalid combinations cannot be prevented through input constraints, the engine shall handle them safely.

The script shall not terminate due to an invalid user configuration.

---

# 7. Core Calculations

The Trend Engine shall calculate:

- Fast moving average
- Medium moving average
- Slow moving average
- Fast moving-average slope
- Medium moving-average slope
- Slow moving-average slope
- Price position relative to each moving average
- Moving-average ordering
- Separation between moving averages
- Higher-timeframe equivalents when enabled

Calculations shall be performed once per bar and reused by all dependent modules.

---

# 8. Moving-Average Alignment

The engine shall classify moving-average ordering.

## 8.1 Bullish Alignment

Bullish alignment exists when:

```text
Fast MA > Medium MA > Slow MA
```

## 8.2 Bearish Alignment

Bearish alignment exists when:

```text
Fast MA < Medium MA < Slow MA
```

## 8.3 Mixed Alignment

Mixed alignment exists when neither bullish nor bearish ordering is present.

Mixed alignment may indicate:

- Consolidation
- Trend transition
- Pullback
- Early reversal
- Insufficient directional agreement

---

# 9. Price Position

The engine shall classify the closing price relative to the trend averages.

Possible price-location states:

- Above all averages
- Above fast and medium, below slow
- Above fast only
- Between averages
- Below fast only
- Below fast and medium, above slow
- Below all averages

For scoring and dashboard output, these detailed states may be simplified into:

- Bullish location
- Bearish location
- Neutral location

---

# 10. Slope Classification

Each trend average shall be classified as:

- Rising
- Falling
- Flat

A moving average shall be considered rising when its normalised change exceeds the positive slope threshold.

A moving average shall be considered falling when its normalised change exceeds the negative slope threshold.

Otherwise, it shall be considered flat.

The slow moving-average slope shall carry greater trend-context importance than the fast moving-average slope.

---

# 11. Trend States

The Trend Engine shall produce one primary trend state.

Required states:

1. Strong Bullish
2. Bullish
3. Weak Bullish
4. Neutral
5. Weak Bearish
6. Bearish
7. Strong Bearish
8. Transition

---

# 12. Strong Bullish State

A Strong Bullish state should require most or all of the following:

- Bullish moving-average alignment
- Price above all trend averages
- Fast average rising
- Medium average rising
- Slow average rising or non-declining
- Positive average separation
- Higher-timeframe bullish agreement when enabled

The final number of mandatory conditions shall be confirmed during implementation planning.

---

# 13. Bullish State

A Bullish state may exist when:

- Price is above the medium and slow averages.
- Fast average is above the medium average.
- Medium average is above or approaching the slow average.
- Most relevant slopes are positive.
- No major bearish contradiction is present.

---

# 14. Weak Bullish State

A Weak Bullish state may exist when bullish evidence is present but incomplete.

Examples include:

- Bullish moving-average ordering with flat slope.
- Price above the slow average but below the fast average.
- Bullish chart trend with neutral higher-timeframe trend.
- Early bullish transition after a bearish phase.
- Bullish price location without full average alignment.

---

# 15. Strong Bearish State

A Strong Bearish state should require most or all of the following:

- Bearish moving-average alignment
- Price below all trend averages
- Fast average falling
- Medium average falling
- Slow average falling or non-rising
- Negative average separation
- Higher-timeframe bearish agreement when enabled

---

# 16. Bearish State

A Bearish state may exist when:

- Price is below the medium and slow averages.
- Fast average is below the medium average.
- Medium average is below or approaching the slow average.
- Most relevant slopes are negative.
- No major bullish contradiction is present.

---

# 17. Weak Bearish State

A Weak Bearish state may exist when bearish evidence is present but incomplete.

Examples include:

- Bearish moving-average ordering with flat slope.
- Price below the slow average but above the fast average.
- Bearish chart trend with neutral higher-timeframe trend.
- Early bearish transition after a bullish phase.
- Bearish price location without full average alignment.

---

# 18. Neutral State

A Neutral state may exist when:

- Moving averages are mixed.
- Slopes are predominantly flat.
- Price repeatedly crosses the averages.
- Average separation is low.
- Bullish and bearish evidence is balanced.
- The market lacks directional consistency.

Neutral shall not automatically mean low volatility.

---

# 19. Transition State

A Transition state shall identify conditions where the previous trend may be weakening or reversing.

Possible transition evidence includes:

- Fast average crossing the medium average.
- Price crossing the slow average.
- Trend slopes changing direction.
- Moving-average alignment becoming mixed.
- Chart trend opposing the higher-timeframe trend.
- Loss of average separation.
- Repeated closes through the trend averages.

Transition shall represent uncertainty and shall reduce confidence in directional signals.

---

# 20. Trend Strength

The Trend Engine shall produce a numerical strength score.

Suggested range:

```text
-100 to +100
```

Interpretation:

| Score Range | Classification |
|---|---|
| +76 to +100 | Strong Bullish |
| +41 to +75 | Bullish |
| +16 to +40 | Weak Bullish |
| -15 to +15 | Neutral |
| -16 to -40 | Weak Bearish |
| -41 to -75 | Bearish |
| -76 to -100 | Strong Bearish |

These thresholds are provisional and shall be validated during testing.

The score shall be derived from weighted trend evidence rather than a single condition.

---

# 21. Proposed Trend Score Components

The initial trend score may include:

| Component | Suggested Maximum Weight |
|---|---:|
| Moving-average alignment | 25 |
| Price location | 20 |
| Fast slope | 10 |
| Medium slope | 15 |
| Slow slope | 15 |
| Average separation | 5 |
| Higher-timeframe alignment | 10 |

Maximum directional magnitude:

```text
100
```

Bullish evidence shall contribute positive values.

Bearish evidence shall contribute negative values.

Conflicting evidence shall reduce the absolute score.

The final weighting model shall be approved before implementation.

---

# 22. Trend Separation

The engine may calculate moving-average separation to distinguish organised trends from compressed conditions.

Possible measurements include:

- Fast-to-medium percentage distance
- Medium-to-slow percentage distance
- Average True Range-normalised distance

Increasing directional separation may support trend strength.

Contracting separation may indicate:

- Momentum loss
- Consolidation
- Pullback
- Trend transition

Separation alone shall not determine the trend state.

---

# 23. Higher-Timeframe Trend

When enabled, the engine shall calculate the same core trend state on the selected higher timeframe.

Required higher-timeframe outputs:

- Higher-timeframe trend direction
- Higher-timeframe trend strength
- Higher-timeframe alignment status
- Higher-timeframe transition status

Possible alignment states:

- Bullish aligned
- Bearish aligned
- Conflicting
- Neutral
- Unavailable

---

# 24. Automatic Higher-Timeframe Selection

Automatic higher-timeframe selection may use the following provisional mapping:

| Chart Timeframe | Automatic Higher Timeframe |
|---|---|
| 1 minute | 5 minutes |
| 3 minutes | 15 minutes |
| 5 minutes | 30 minutes |
| 15 minutes | 1 hour |
| 30 minutes | 2 hours |
| 1 hour | 4 hours |
| 2 hours | Daily |
| 4 hours | Daily |
| Daily | Weekly |
| Weekly | Monthly |

This mapping shall be reviewed in the Settings specification.

The user shall be able to override the automatic selection.

---

# 25. Higher-Timeframe Data Policy

Higher-timeframe calculations shall:

- Use confirmed historical data.
- Never use lookahead.
- Avoid future leakage.
- Document behaviour on the currently forming higher-timeframe candle.
- Handle unavailable timeframe data safely.

The implementation shall distinguish between:

- Confirmed higher-timeframe state
- Developing higher-timeframe state

The production default shall prioritise confirmed higher-timeframe values.

---

# 26. Trend Change Detection

The engine shall detect trend-state changes.

Required transition events:

- Neutral to Bullish
- Neutral to Bearish
- Bearish to Bullish
- Bullish to Bearish
- Bullish to Transition
- Bearish to Transition
- Weak trend to Strong trend
- Strong trend to Weak trend

Trend changes shall only be confirmed at bar close unless a future real-time mode is explicitly added.

---

# 27. Outputs

The Trend Engine shall expose the following outputs internally:

- Trend enabled status
- Fast moving-average value
- Medium moving-average value
- Slow moving-average value
- Fast slope state
- Medium slope state
- Slow slope state
- Moving-average alignment state
- Price-location state
- Trend direction
- Trend classification
- Trend strength score
- Transition state
- Higher-timeframe trend direction
- Higher-timeframe trend strength
- Higher-timeframe alignment state
- Trend change event

---

# 28. Output Enumerations

Where practical, internal states shall use stable constants or enumerated integer values.

Suggested trend direction values:

```text
+1 = Bullish
 0 = Neutral
-1 = Bearish
```

Suggested transition value:

```text
true = Transition condition active
false = No transition condition
```

Suggested alignment values:

```text
+1 = Bullish alignment
 0 = Mixed alignment
-1 = Bearish alignment
```

Display text shall be produced separately from internal state values.

---

# 29. Dashboard Integration

The Dashboard shall be able to display:

- Current trend classification
- Trend strength score
- Moving-average alignment
- Higher-timeframe trend
- Higher-timeframe alignment
- Transition warning

Suggested display examples:

```text
Trend: Strong Bullish
Strength: +82
HTF: Bullish
Alignment: Confirmed
```

Dashboard colours shall use the central theme manager.

---

# 30. Chart Visualisation

Optional chart visuals may include:

- Fast trend average
- Medium trend average
- Slow trend average
- Trend-coloured background
- Trend transition markers
- Trend-strength label
- Higher-timeframe alignment marker

All visual elements shall be independently configurable where practical.

The default visual presentation shall avoid excessive chart clutter.

---

# 31. Alert Integration

The Trend Engine shall support alert conditions for:

- Trend turns bullish
- Trend turns bearish
- Strong bullish trend confirmed
- Strong bearish trend confirmed
- Trend enters transition
- Higher-timeframe bullish alignment
- Higher-timeframe bearish alignment
- Higher-timeframe conflict begins

Alert conditions shall be confirmed at bar close by default.

Alert messages shall include:

- Symbol
- Chart timeframe
- Event type
- Trend classification
- Trend strength
- Higher-timeframe trend
- Timestamp where supported

---

# 32. Scoring Integration

The Trend Engine shall provide directional evidence to the Signal Scoring Engine.

Possible scoring effects:

- Strong bullish trend supports long signals.
- Strong bearish trend supports short signals.
- Neutral trend reduces directional confidence.
- Transition state applies a confidence penalty.
- Higher-timeframe agreement increases confidence.
- Higher-timeframe conflict decreases confidence.

The Trend Engine shall not independently approve or reject a trade setup unless the user enables a strict trend filter.

---

# 33. Module Interactions

## 33.1 Market Structure

The Trend Engine shall provide directional context but shall not override confirmed structural events.

## 33.2 BOS and CHOCH

Break of Structure and Change of Character events may strengthen or challenge the current trend classification.

## 33.3 VPA

Volume Price Analysis events shall be interpreted differently depending on trend context.

## 33.4 Wyckoff

Wyckoff accumulation and distribution events may precede a Trend Engine transition.

## 33.5 Institutional Activity

Institutional evidence aligned with the trend may increase confidence.

Institutional evidence opposing the trend may indicate absorption, distribution, accumulation, or transition.

---

# 34. Repainting Requirements

The Trend Engine shall not intentionally repaint confirmed historical states.

Moving averages shall use standard historical values.

Higher-timeframe calculations shall use lookahead-disabled requests.

Trend-change markers shall be confirmed at bar close by default.

Any optional developing-bar behaviour shall be clearly labelled as provisional.

---

# 35. Performance Requirements

The Trend Engine shall:

- Calculate each moving average only once per timeframe.
- Reuse calculations across outputs.
- Minimise higher-timeframe requests.
- Avoid unnecessary loops.
- Avoid repeated string construction on every bar.
- Update dashboard text only when required by the dashboard implementation.
- Avoid unnecessary drawing-object creation.

---

# 36. Error Handling

The engine shall handle:

- Insufficient historical bars
- Missing volume-independent data
- Unsupported higher timeframes
- Invalid moving-average lengths
- Unavailable higher-timeframe values
- Early-chart null values

During insufficient-history periods, the engine shall return an unavailable or neutral state rather than producing misleading classifications.

---

# 37. Testing Requirements

The Trend Engine shall be tested across:

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

Required market-condition tests:

- Strong uptrend
- Weak uptrend
- Strong downtrend
- Weak downtrend
- Sideways market
- Volatile consolidation
- Trend reversal
- Pullback within trend
- Higher-timeframe conflict
- Limited historical data

---

# 38. Functional Test Cases

## Test TREND-001 — Bullish Alignment

Given:

- Fast MA above Medium MA
- Medium MA above Slow MA
- Price above all three averages

Expected:

- Bullish alignment is true.
- Trend score is positive.

---

## Test TREND-002 — Bearish Alignment

Given:

- Fast MA below Medium MA
- Medium MA below Slow MA
- Price below all three averages

Expected:

- Bearish alignment is true.
- Trend score is negative.

---

## Test TREND-003 — Neutral Compression

Given:

- Mixed moving-average ordering
- Flat slopes
- Price crossing averages repeatedly

Expected:

- Trend classification is Neutral or Transition.
- Absolute trend score remains low.

---

## Test TREND-004 — Higher-Timeframe Agreement

Given:

- Chart trend is Bullish
- Higher-timeframe trend is Bullish

Expected:

- Higher-timeframe alignment is Bullish Aligned.
- Trend confidence may increase.

---

## Test TREND-005 — Higher-Timeframe Conflict

Given:

- Chart trend is Bullish
- Higher-timeframe trend is Bearish

Expected:

- Alignment state is Conflicting.
- Trend confidence is reduced.

---

## Test TREND-006 — Trend Transition

Given:

- Previous trend is Bullish
- Alignment becomes mixed
- Slopes weaken or reverse
- Price moves through key averages

Expected:

- Transition state becomes active.
- A trend-transition event may be generated.

---

## Test TREND-007 — Insufficient Data

Given:

- Available chart history is less than the Slow Length

Expected:

- No runtime error.
- Trend output is unavailable or neutral.
- No misleading strong-trend signal is generated.

---

# 39. Acceptance Criteria

The Trend Engine shall be considered complete when:

- All three trend averages calculate correctly.
- All supported moving-average types operate correctly.
- Bullish, bearish, neutral, and transition states are produced.
- Trend strength remains within the defined range.
- Higher-timeframe calculations work without lookahead.
- Higher-timeframe alignment is correctly classified.
- Dashboard output is available.
- Alert conditions compile.
- No confirmed historical states repaint unexpectedly.
- Insufficient data is handled safely.
- TradingView compilation succeeds.
- Required test cases pass.

---

# 40. Open Design Decisions

The following decisions remain unresolved and shall be recorded in `DECISIONS.md` when approved:

1. Exact moving-average default periods.
2. Final slope normalisation method.
3. Final trend-score weights.
4. Final trend-state thresholds.
5. Whether the Transition state replaces or supplements directional classification.
6. Whether developing higher-timeframe values are visible.
7. Final automatic higher-timeframe mapping.
8. Whether HMA and RMA are included in the first alpha implementation.
9. Whether trend separation uses percentage or ATR normalisation.
10. Whether strict higher-timeframe alignment is available in Version 1.0.

---

# 41. Future Enhancements

Possible future enhancements include:

- Adaptive moving-average lengths
- Volatility-adjusted trend sensitivity
- Asset-class-specific presets
- Trend persistence measurement
- Trend maturity classification
- Trend exhaustion detection
- Multi-higher-timeframe alignment
- Session-aware trend analysis
- User-defined trend-score weighting