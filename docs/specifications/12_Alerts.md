# Alerts Specification

---

## Document Information

**Document ID:** SPEC-112

**Version:** 0.1 Draft

**Status:** Draft

**Module:** Alerts

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
- Signal Scoring Engine
- Shared Evidence Model
- Dashboard

**Dependent Modules:**

- External TradingView alert configuration
- Future webhook consumers
- Future strategy and automation integrations

---

# 1. Purpose

The Alerts module publishes confirmed analytical and signal events to TradingView alert conditions and formatted runtime messages.

Its primary responsibilities are to:

- Receive approved events from upstream modules.
- Determine alert eligibility.
- Apply global and event-specific filters.
- Suppress duplicates.
- Enforce cooldown and rate limits.
- Format clear and consistent messages.
- Publish static alert conditions where required.
- Publish dynamic alert messages where supported.
- Preserve event identity and traceability.
- Prevent provisional or invalid events from reaching users.
- Expose alert diagnostics.
- Operate independently from analytical classification.

The Alerts module shall not create analytical events.

It shall only route, filter, format, and publish events already produced by upstream engines.

---

# 2. Objectives

The Alerts module shall:

- Centralise all alert publication logic.
- Avoid alert logic duplication across analytical modules.
- Support module-level alert enablement.
- Support event-level alert enablement.
- Support state-transition-only alerts.
- Support confirmed-bar publication by default.
- Suppress duplicate alerts from the same source event.
- Apply confidence, score, quality, and readiness thresholds.
- Support alert cooldowns.
- Support global alert-rate limits.
- Distinguish candidate, confirmed, reinforced, failed, invalidated, expired, and completed events.
- Provide compact and detailed message formats.
- Support TradingView `alertcondition()` and `alert()` models where appropriate.
- Produce deterministic messages.
- Include sufficient metadata for webhook processing.
- Avoid unsupported promises of guaranteed delivery.
- Handle missing values safely.
- Preserve stable event identifiers.
- Remain performant under high event frequency.
- Support later external integrations without changing analytical engines.

---

# 3. Design Principles

## 3.1 Alerts Are a Publication Layer

The Alerts module shall not:

- Detect Trend
- Detect BOS
- Detect Liquidity Sweeps
- Classify VPA
- Classify Wyckoff
- Infer Institutional Activity
- Calculate Signal Scores
- Generate setup states independently

It shall consume published events only.

## 3.2 Confirmed Events by Default

Production alerts shall use confirmed-bar events unless a specific developing-alert mode is explicitly enabled.

Default policy:

```text
Confirmed bar only
```

## 3.3 One Event, One Alert Identity

Each alertable source event shall have a stable unique identifier.

Repeated evaluation of the same source event shall not create repeated alerts unless the alert policy explicitly permits reinforcement alerts.

## 3.4 State Changes Over Repeated States

For persistent states such as:

- Bullish Trend
- Phase D
- Accumulation Bias
- Long Ready

the default policy shall alert on state entry or transition rather than on every bar while the state remains active.

## 3.5 Conservative Alerting

When evidence is incomplete, conflicting, stale, invalid, or below threshold, the module shall withhold the alert.

## 3.6 Explainable Messages

Alert messages shall identify:

- What happened
- Direction
- Confidence
- Relevant context
- Trigger or level
- Invalidation where available

## 3.7 Centralised Governance

Global alert behaviour shall be controlled in one module.

Upstream modules may publish event metadata, but they shall not independently implement duplicate suppression, cooldown, or message formatting.

## 3.8 Stable Historical Events

Once an alert event is confirmed and recorded internally, later data shall not rewrite its original:

- Event bar
- Event type
- Direction
- Confidence
- Source identifier
- Message context snapshot

---

# 4. Core Definitions

## 4.1 Alert Event

An Alert Event is a confirmed, alert-eligible published event received from an upstream module.

## 4.2 Alert Candidate

An Alert Candidate is an upstream event that has not yet passed all alert filters.

## 4.3 Alert Publication

Alert Publication is the act of exposing an event through:

- TradingView alert condition
- Runtime `alert()` message
- Internal alert state
- Future webhook output

## 4.4 Alert Identity

Alert Identity is a stable identifier representing one source event.

Conceptually:

```text
Alert ID =
Source Module
+
Event Type
+
Direction
+
Source Event ID
+
Confirmation Bar
```

## 4.5 Duplicate Alert

A Duplicate Alert is a repeated publication attempt for the same alert identity.

## 4.6 Reinforcement Alert

A Reinforcement Alert is a later event that materially strengthens an existing active state.

Example:

```text
Bullish Qualified
→ Long Ready
```

This is not a duplicate because the state changed.

## 4.7 Cooldown

Cooldown is the minimum number of bars between eligible alerts of a specified type, direction, module, or setup.

## 4.8 Rate Limit

A Rate Limit restricts the total number of alerts that may be published over a defined period or bar window.

## 4.9 Alert Severity

Alert Severity indicates the practical importance of an event.

Suggested levels:

- Informational
- Watch
- Qualified
- Actionable
- Critical State Change

## 4.10 Alert Snapshot

An Alert Snapshot is the set of values captured when an alert is confirmed.

Later current-state changes shall not alter the original snapshot.

---

# 5. Alert Processing Pipeline

Required processing order:

```text
Receive Upstream Event
        │
        ▼
Validate Event
        │
        ▼
Check Module Enablement
        │
        ▼
Check Event Enablement
        │
        ▼
Check Lifecycle Eligibility
        │
        ▼
Check Confirmed-Bar Policy
        │
        ▼
Check Confidence and Score Filters
        │
        ▼
Check Context Requirements
        │
        ▼
Check Duplicate Identity
        │
        ▼
Check Cooldown
        │
        ▼
Check Global Rate Limit
        │
        ▼
Resolve Priority
        │
        ▼
Format Message
        │
        ▼
Publish Alert
        │
        ▼
Store Alert Record
```

---

# 6. Alert Sources

The Alerts module shall support events from:

- Trend
- RVOL
- Market Structure
- BOS and CHOCH
- Liquidity
- VPA
- Wyckoff
- Institutional Activity
- Signal Scoring
- Framework health and diagnostics where approved

---

# 7. Inputs

## 7.1 Enable Alerts

Type:

Boolean

Default:

Enabled

When disabled:

- No runtime alerts shall be published.
- No event shall pass alert eligibility.
- Analytical engines shall continue operating.
- Internal diagnostics may continue if Debug mode is enabled.

---

## 7.2 Alert Mode

Type:

Selection

Options:

- Signal Only
- Major Events
- All Confirmed Events
- Custom

Suggested production default:

Signal Only

### Signal Only

Includes:

- Long Ready
- Short Ready
- Long Triggered
- Short Triggered
- Signal Invalidated
- Signal Expired where enabled

### Major Events

Includes Signal Only plus:

- BOS and CHOCH
- Liquidity Sweeps and Traps
- Spring and Upthrust
- SOS and SOW
- Institutional state transitions
- Confirmed range breakout or breakdown

### All Confirmed Events

Includes all eligible confirmed upstream events.

### Custom

Uses individual event settings.

---

## 7.3 Confirmation Mode

Type:

Selection

Options:

- Confirmed Bar Only
- Developing and Confirmed
- Developing Only

Suggested production default:

Confirmed Bar Only

Developing alerts shall be clearly marked provisional.

---

## 7.4 Alert Frequency Policy

Type:

Selection

Options:

- Once Per Event
- Once Per Bar
- Once Per Bar Close
- State Change Only

Suggested production default:

Once Per Event

---

## 7.5 Alert Message Detail

Type:

Selection

Options:

- Compact
- Standard
- Detailed
- Webhook JSON

Suggested production default:

Standard

---

## 7.6 Minimum Alert Confidence

Type:

Integer

Suggested default:

70

Range:

```text
0 to 100
```

Applies globally unless overridden by a stricter event-specific threshold.

---

## 7.7 Minimum Signal Score

Type:

Integer

Suggested default:

65

Purpose:

Minimum directional score for signal-related alerts.

---

## 7.8 Minimum Setup Quality

Type:

Integer

Suggested default:

65

---

## 7.9 Minimum Entry Readiness

Type:

Integer

Suggested default:

70

---

## 7.10 Minimum Institutional Confidence

Type:

Integer

Suggested default:

70

---

## 7.11 Minimum Wyckoff Confidence

Type:

Integer

Suggested default:

70

---

## 7.12 Require Confirmed Structure for Directional Alerts

Type:

Boolean

Suggested default:

Enabled

Purpose:

Directional alerts such as Long Ready or Bullish Institutional Bias may require valid structure where specified.

---

## 7.13 Require Multi-Module Confirmation

Type:

Boolean

Suggested default:

Enabled for Signal alerts

---

## 7.14 Minimum Confirming Modules

Type:

Integer

Suggested default:

3

Minimum:

1

---

## 7.15 Alert Cooldown Bars

Type:

Integer

Suggested default:

3

Minimum:

0

Purpose:

Global minimum bars between repeated similar alerts.

---

## 7.16 Directional Cooldown Bars

Type:

Integer

Suggested default:

5

Purpose:

Minimum bars between repeated same-direction setup or signal alerts.

---

## 7.17 Opposing Signal Override

Type:

Boolean

Suggested default:

Enabled

Purpose:

Allows a valid opposite-direction alert to bypass same-direction cooldown after a confirmed invalidation or reversal.

---

## 7.18 Maximum Alerts Per Bar

Type:

Integer

Suggested default:

1

Minimum:

1

Purpose:

Limits alert bursts when multiple events confirm on one bar.

---

## 7.19 Same-Bar Alert Policy

Type:

Selection

Options:

- Highest Priority Only
- Combine Compatible Events
- Publish All Within Limit

Suggested production default:

Highest Priority Only

---

## 7.20 Combine Supporting Evidence

Type:

Boolean

Suggested default:

Enabled

Purpose:

Allows one primary alert message to include supporting upstream events confirmed on the same bar.

---

## 7.21 State Change Only

Type:

Boolean

Suggested default:

Enabled for persistent states

Purpose:

Prevents repeated alerts while a state remains unchanged.

---

## 7.22 Alert on Reinforcement

Type:

Boolean

Suggested default:

Enabled

Examples:

- Watch to Qualified
- Qualified to Ready
- Ready to Triggered
- Accumulation Bias to Markup Confirmation
- Phase C to Phase D

---

## 7.23 Alert on Weakening

Type:

Boolean

Suggested default:

Disabled

---

## 7.24 Alert on Invalidation

Type:

Boolean

Suggested default:

Enabled

---

## 7.25 Alert on Expiry

Type:

Boolean

Suggested default:

Disabled

---

## 7.26 Alert on No Trade

Type:

Boolean

Suggested default:

Disabled

Possible use:

- High Conflict
- Mid Range
- Extended Price
- Trigger Invalidated

---

## 7.27 Include Price

Type:

Boolean

Suggested default:

Enabled

---

## 7.28 Include Event Level

Type:

Boolean

Suggested default:

Enabled

---

## 7.29 Include Invalidation Level

Type:

Boolean

Suggested default:

Enabled for signal alerts

---

## 7.30 Include Confidence

Type:

Boolean

Suggested default:

Enabled

---

## 7.31 Include Scores

Type:

Boolean

Suggested default:

Enabled for Standard and Detailed modes

---

## 7.32 Include Evidence Summary

Type:

Boolean

Suggested default:

Enabled

---

## 7.33 Maximum Evidence Items in Message

Type:

Integer

Suggested default:

3

Range:

```text
0 to 5
```

---

## 7.34 Include Wyckoff Context

Type:

Boolean

Suggested default:

Enabled where available

---

## 7.35 Include Institutional Context

Type:

Boolean

Suggested default:

Enabled where available

---

## 7.36 Include Timestamp

Type:

Boolean

Suggested default:

Enabled

---

## 7.37 Include Ticker and Timeframe

Type:

Boolean

Suggested default:

Enabled

---

## 7.38 Enable Webhook Format

Type:

Boolean

Suggested default:

Disabled

Purpose:

Enables machine-readable message formatting.

This does not send a webhook by itself.

The user must configure a TradingView alert webhook externally.

---

## 7.39 Enable Trend Alerts

Type:

Boolean

Suggested default:

Disabled

---

## 7.40 Enable RVOL Alerts

Type:

Boolean

Suggested default:

Disabled

---

## 7.41 Enable Structure Alerts

Type:

Boolean

Suggested default:

Disabled

---

## 7.42 Enable BOS and CHOCH Alerts

Type:

Boolean

Suggested default:

Enabled in Major Events mode

---

## 7.43 Enable Liquidity Alerts

Type:

Boolean

Suggested default:

Enabled in Major Events mode

---

## 7.44 Enable VPA Alerts

Type:

Boolean

Suggested default:

Disabled

---

## 7.45 Enable Wyckoff Alerts

Type:

Boolean

Suggested default:

Enabled in Major Events mode

---

## 7.46 Enable Institutional Alerts

Type:

Boolean

Suggested default:

Enabled in Major Events mode

---

## 7.47 Enable Signal Alerts

Type:

Boolean

Suggested default:

Enabled

---

## 7.48 Enable Diagnostic Alerts

Type:

Boolean

Suggested default:

Disabled

---

# 8. Input Validation

The Alerts module shall validate that:

- Confidence and score thresholds are between 0 and 100.
- Minimum Confirming Modules is positive.
- Cooldown values are non-negative.
- Maximum Alerts Per Bar is positive.
- Maximum Evidence Items is within approved limits.
- Webhook mode uses a supported message template.
- Developing-only mode is not silently treated as confirmed.
- Event-specific thresholds do not fall outside valid ranges.

Invalid inputs shall not cause runtime failure.

---

# 9. Alert Event Model

Conceptual alert event:

```text
AlertEvent
{
    Alert ID
    Source Module
    Event Type
    Event Direction
    Event Lifecycle
    Event Severity
    Source Event ID
    Setup ID
    Confirmation Bar
    Timestamp
    Price
    Event Level
    Invalidation Level
    Confidence
    Bullish Score
    Bearish Score
    Setup Quality
    Entry Readiness
    Confirming Modules
    Primary Context
    Supporting Evidence
    Message
}
```

Implementation may use:

- Scalar event channels
- Enumerations
- Parallel arrays
- User-defined types where supported
- Bounded queues

---

# 10. Alert Lifecycle Eligibility

Suggested lifecycle handling:

| Lifecycle | Default Alert Eligibility |
|---|---|
| Candidate | No |
| Developing | Optional |
| Confirmed | Yes |
| Reinforced | Optional |
| Weakening | No |
| Failed | Optional |
| Invalidated | Yes |
| Expired | Optional |
| Completed | Optional |

Signal alerts shall prioritise:

- Ready
- Triggered
- Invalidated

---

# 11. Alert Severity

Suggested severity mapping:

## Informational

Examples:

- Trend change
- RVOL expansion
- Range confirmed
- Phase transition

## Watch

Examples:

- Bullish Watch
- Bearish Watch
- Spring candidate where developing mode is enabled
- Institutional Reversal Evidence

## Qualified

Examples:

- Bullish Qualified
- Bearish Qualified
- Confirmed Spring
- Confirmed Upthrust
- Accumulation Bias
- Distribution Bias

## Actionable

Examples:

- Long Ready
- Short Ready
- Long Triggered
- Short Triggered
- Markup Confirmation
- Markdown Confirmation

## Critical State Change

Examples:

- Signal Invalidated
- Opposing Triggered signal
- Failed breakout after Ready state
- Hard range invalidation

Severity shall support priority resolution.

---

# 12. Event Priority

Suggested same-bar event priority:

1. Signal Invalidated
2. Opposite-direction Triggered signal
3. Long Triggered or Short Triggered
4. Long Ready or Short Ready
5. Failed range breakout or breakdown
6. Markup or Markdown Confirmation
7. Spring or Upthrust
8. Sign of Strength or Sign of Weakness
9. Bullish or Bearish Qualified
10. Institutional state transition
11. BOS or CHOCH
12. Liquidity Sweep or Trap
13. VPA event
14. Trend transition
15. RVOL event
16. Diagnostic event

This priority is provisional.

---

# 13. Same-Bar Event Resolution

When multiple events qualify on the same bar:

## Highest Priority Only

Publish only the highest-priority event.

Supporting compatible events may appear in the message.

## Combine Compatible Events

Example:

```text
Long Ready
+
Sell-Side Sweep
+
Bullish CHOCH
+
Bullish Absorption
```

publishes one Long Ready alert containing the other events as evidence.

## Publish All Within Limit

Publish multiple events up to Maximum Alerts Per Bar.

This mode may produce excessive alerts and shall not be the default.

---

# 14. Compatibility Rules

Events may be combined when they:

- Share direction.
- Share setup ID or source interaction.
- Occur on the same confirmation bar.
- Support the same primary interpretation.
- Do not represent separate critical state transitions.

Events shall not be combined when:

- Directions conflict.
- One event invalidates another.
- Different setup IDs are involved.
- Both events require separate operational attention.
- A critical invalidation would be obscured.

---

# 15. Duplicate Suppression

Duplicate suppression shall use:

- Alert ID
- Source Event ID
- Setup ID
- Event type
- Direction
- Confirmation bar
- Source level
- Lifecycle transition

An event shall not alert repeatedly merely because it remains active.

---

# 16. Duplicate Suppression Examples

## Persistent Trend

```text
Bullish Trend
```

shall alert on transition into Bullish, not every bullish bar.

## Persistent Signal State

```text
Long Ready
```

shall alert once on entry into Ready.

## Repeated BOS Reference

The same BOS source event shall not produce repeated alerts.

## Repeated Sweep Evaluation

One Liquidity Sweep source ID shall alert once.

---

# 17. Reinforcement Events

A reinforcement event may alert when the lifecycle materially advances.

Examples:

```text
Bullish Watch
→ Bullish Qualified
```

```text
Bullish Qualified
→ Long Ready
```

```text
Long Ready
→ Long Triggered
```

```text
Accumulation Bias
→ Markup Confirmation
```

These shall use distinct alert identities.

---

# 18. Cooldown Policy

Cooldown shall not suppress:

- Signal Invalidated
- Opposing hard reversal
- Critical failure event
- New setup with a different Setup ID
- Event explicitly exempted by severity

Cooldown may suppress:

- Repeated same-direction BOS alerts
- Repeated liquidity sweeps of similar priority
- Repeated VPA labels
- Repeated Watch states
- Repeated same-direction Qualified states

---

# 19. Cooldown Scope

Possible cooldown scopes:

- Global
- Per module
- Per event type
- Per direction
- Per setup ID
- Per source level

Suggested production policy:

```text
Per event type and direction
```

with additional Setup ID checks for signals.

---

# 20. Alert Rate Limiting

Rate limiting may consider:

- Maximum Alerts Per Bar
- Maximum alerts in rolling bar window
- Maximum same-module alerts
- Maximum same-direction alerts

The first implementation shall require only:

- Maximum Alerts Per Bar
- Cooldown Bars
- Duplicate suppression

More complex rolling limits may be added later.

---

# 21. Trend Alerts

Supported Trend alerts may include:

- Trend changed Bullish
- Trend changed Bearish
- Trend changed Neutral
- Strong Bullish Trend
- Strong Bearish Trend
- Higher-timeframe alignment gained
- Higher-timeframe alignment lost

Default:

Disabled

Trend alerts shall use state-change-only logic.

---

# 22. RVOL Alerts

Supported RVOL alerts may include:

- High RVOL
- Extreme RVOL
- Volume availability restored
- Volume unavailable

Default:

Disabled

Because RVOL is non-directional, messages shall not imply bullish or bearish intent.

Example:

```text
High relative volume detected — RVOL 2.4
```

---

# 23. Market Structure Alerts

Supported events may include:

- Higher High
- Higher Low
- Lower High
- Lower Low
- Equal High
- Equal Low
- Structural direction changed
- Protection level failed

Default:

Disabled

---

# 24. BOS and CHOCH Alerts

Supported events:

- Bullish BOS
- Bearish BOS
- Bullish CHOCH
- Bearish CHOCH
- Failed Bullish Break
- Failed Bearish Break
- Accepted Structural Break
- Structural Rejection

Recommended:

Enabled in Major Events mode.

---

# 25. Liquidity Alerts

Supported events:

- Sell-Side Sweep
- Buy-Side Sweep
- Bear Trap
- Bull Trap
- Accepted Buy-Side Break
- Accepted Sell-Side Break
- High-priority liquidity cleared
- Liquidity rejection

Messages shall distinguish:

```text
Sweep
```

from:

```text
Accepted Break
```

---

# 26. VPA Alerts

Supported events:

- No Demand
- No Supply
- Stopping Volume
- Buying Climax
- Selling Climax
- Bullish Absorption
- Bearish Absorption
- Churn
- Successful Test
- Failed Test
- Strength Confirmation
- Weakness Confirmation

Default:

Disabled

VPA messages shall remain descriptive.

They shall not claim guaranteed future movement.

---

# 27. Wyckoff Alerts

Supported events:

- Trading Range Confirmed
- Accumulation Candidate
- Accumulation Confirmed
- Distribution Candidate
- Distribution Confirmed
- Reaccumulation
- Redistribution
- Phase Transition
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
- Range Breakout
- Range Breakdown
- Failed Breakout
- Failed Breakdown
- Range Invalidated

Wyckoff alerts shall include phase and schematic confidence where available.

---

# 28. Institutional Alerts

Supported events:

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
- State Invalidated

Default policy:

State Change Only

---

# 29. Signal Alerts

Supported events:

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
- Signal Completed
- No Trade where explicitly enabled

Recommended production alerts:

- Long Ready
- Short Ready
- Long Triggered
- Short Triggered
- Invalidated

---

# 30. Signal Alert Eligibility

Long Ready or Short Ready may require:

- Confirmed signal state
- Minimum Alert Confidence
- Minimum Setup Quality
- Minimum Entry Readiness
- Minimum directional score
- Minimum confirming modules
- No hard No-Trade filter
- Valid Setup ID
- Not previously alerted
- Cooldown satisfied

---

# 31. Triggered Signal Eligibility

Long Triggered or Short Triggered shall require:

- Approved trigger state
- Confirmed bar
- Valid setup lifecycle
- Stable Setup ID
- Trigger not previously consumed
- No invalidation on the same event
- Alert filters satisfied

Triggered alerts shall have higher priority than Ready alerts on the same bar where same-bar triggering is permitted.

---

# 32. Invalidation Alerts

Invalidation alerts shall generally bypass normal directional cooldown.

Message shall include:

- Setup direction
- Setup type
- Invalidation reason
- Invalidation level
- Current price
- Original Setup ID where practical
- Opposing event where applicable

Example:

```text
EURUSD 15m — Long setup invalidated — accepted close below 1.0824
```

---

# 33. Expiry Alerts

Expiry alerts may be enabled for users who track developing setups.

Default:

Disabled

Example:

```text
BTCUSD 1H — Bullish Reversal setup expired — no valid trigger within 10 bars
```

---

# 34. Compact Message Format

Compact format shall prioritise brevity.

Example:

```text
BTCUSD 4H | Long Ready | Bullish Reversal | Conf 82
```

Example invalidation:

```text
BTCUSD 4H | Long Invalidated | Below 64200
```

---

# 35. Standard Message Format

Standard format shall include:

- Symbol
- Timeframe
- Event
- Setup or context
- Confidence
- Key evidence

Example:

```text
BTCUSD 4H — Long Ready — Bullish Reversal — Confidence 82 — Spring + Bullish CHOCH + Bullish Absorption
```

---

# 36. Detailed Message Format

Detailed format may include:

- Symbol
- Timeframe
- Timestamp
- Event
- Direction
- Setup type
- Confidence
- Bullish score
- Bearish score
- Setup Quality
- Entry Readiness
- Trigger price
- Invalidation
- Wyckoff phase
- Institutional state
- Supporting evidence
- Opposing evidence
- Setup ID

Example:

```text
BTCUSD 4H — Long Ready
Setup: Bullish Reversal
Confidence: 82
Bullish Score: 86
Bearish Score: 31
Quality: 80
Readiness: 76
Trigger: 65240
Invalidation: 64180
Wyckoff: Accumulation Phase D
Institutional: Accumulation Bias
Drivers: Spring, Bullish CHOCH, Bullish Absorption
```

---

# 37. Webhook JSON Format

Webhook format should be machine-readable and deterministic.

Conceptual example:

```json
{
  "version": "1.0",
  "indicator": "VPA Wyckoff Hybrid Pro Professional",
  "symbol": "BTCUSD",
  "timeframe": "240",
  "timestamp": 0,
  "event_id": "SIGNAL_LONG_READY_12345",
  "source": "SignalScoring",
  "event": "LongReady",
  "direction": "Long",
  "setup_type": "BullishReversal",
  "confidence": 82,
  "bullish_score": 86,
  "bearish_score": 31,
  "setup_quality": 80,
  "entry_readiness": 76,
  "price": 65240,
  "trigger_level": 65240,
  "invalidation_level": 64180,
  "wyckoff_state": "Accumulation",
  "institutional_state": "AccumulationBias",
  "setup_id": "BR_12040_64180"
}
```

The implementation shall ensure valid escaping and supported string length.

---

# 38. Webhook Schema Stability

Webhook field names shall remain stable after Version 1.0.

Future fields may be added.

Existing fields shall not be renamed without:

- Version increment
- Changelog entry
- Migration note

---

# 39. Message Value Formatting

Price formatting shall use symbol tick precision where possible.

Percentages and scores shall use consistent decimals.

Suggested:

- Scores: integer
- Confidence: integer
- RVOL: two decimals
- Price: symbol format
- ATR distance: two decimals
- Timeframe: TradingView timeframe string

Unavailable values shall be omitted or shown as:

```text
N/A
```

They shall not appear as misleading zero values.

---

# 40. Alert Conditions

Where practical, static `alertcondition()` entries shall be provided for major user-selectable events.

Possible conditions:

- Long Ready
- Short Ready
- Long Triggered
- Short Triggered
- Signal Invalidated
- Bullish BOS
- Bearish BOS
- Bullish CHOCH
- Bearish CHOCH
- Sell-Side Sweep
- Buy-Side Sweep
- Spring
- Upthrust
- SOS
- SOW
- Markup Confirmation
- Markdown Confirmation

The number of static conditions shall remain manageable.

---

# 41. Runtime Alerts

Dynamic `alert()` messages may be used for:

- Detailed messages
- Combined events
- Webhook JSON
- Evidence lists
- Dynamic levels
- Setup identifiers

Runtime alerts shall follow the configured confirmation and frequency policies.

---

# 42. Alertcondition and Runtime Coexistence

The architecture may support both:

- Static `alertcondition()` for simple TradingView selection
- Dynamic `alert()` for rich messages

The implementation shall avoid accidental double-publication where both are enabled.

A clear user setting shall govern the publication method where required.

---

# 43. Alert Publication Method

Suggested options:

- Alert Conditions Only
- Dynamic Alerts Only
- Both

Suggested default:

Dynamic Alerts Only

The final production decision remains open.

---

# 44. Bar Confirmation

Confirmed-bar alert eligibility shall use the approved bar confirmation state.

No alert shall be backdated to an earlier bar.

A developing event that becomes confirmed shall publish on its confirmation bar.

---

# 45. Intrabar Alerts

Intrabar alerts are inherently provisional.

When enabled:

- Message shall contain `PROVISIONAL`.
- They shall use a separate Alert ID lifecycle.
- Confirmed alert may still publish later.
- Intrabar cancellation does not rewrite an already delivered external notification.
- Default profiles shall keep intrabar alerts disabled.

---

# 46. Historical Versus Real-Time Behaviour

Historical bars may calculate alert events for testing and plots.

Actual TradingView alert delivery occurs only under platform alert execution conditions.

The module shall maintain consistent event logic across:

- Historical calculation
- Real-time confirmed bars
- Real-time developing bars where enabled

---

# 47. Alert Snapshot Fields

For each published alert, store where practical:

- Alert ID
- Event type
- Direction
- Source
- Confirmation bar
- Price
- Event level
- Confidence
- Scores
- Setup ID
- Primary context
- Alert severity
- Message-format version

Storage shall be bounded.

---

# 48. Alert History

The indicator may retain a bounded internal history of recent alert events for:

- Duplicate suppression
- Debug display
- Regression testing
- State traceability

Suggested initial maximum:

```text
50 alert records
```

This is not an external audit log.

---

# 49. Alert Record Expiry

Old alert records may be removed when:

- Maximum history is exceeded.
- Associated setup is completed and cooldown has passed.
- Event cannot recur with the same ID.
- Internal memory limits require cleanup.

Current active setup records shall be preserved.

---

# 50. Alert Diagnostics

Debug outputs may include:

- Last candidate event
- Last rejected event
- Rejection reason
- Last published alert
- Last Alert ID
- Cooldown remaining
- Alerts published this bar
- Duplicate suppression count
- Event-priority result
- Message length
- Webhook-format validity state

---

# 51. Alert Rejection Reasons

Suggested rejection reasons:

```text
0 = None
1 = Alerts Disabled
2 = Module Disabled
3 = Event Disabled
4 = Candidate Not Confirmed
5 = Confidence Too Low
6 = Score Too Low
7 = Setup Quality Too Low
8 = Entry Readiness Too Low
9 = Insufficient Modules
10 = Missing Required Context
11 = Duplicate Event
12 = Cooldown Active
13 = Rate Limit Reached
14 = Lower Priority Same-Bar Event
15 = State Unchanged
16 = Event Expired
17 = Event Invalid
18 = Message Construction Failed
```

---

# 52. Error Handling

The Alerts module shall safely handle:

- Missing source event
- Invalid event enumeration
- Missing direction
- Missing price level
- Missing invalidation level
- Missing Setup ID
- Missing confidence
- Oversized evidence list
- Message-length pressure
- Duplicate Alert ID
- Alert history overflow
- Conflicting same-bar events
- Disabled source module
- Developing event cancellation
- Gap bars
- Symbol changes
- Timeframe changes
- Indicator reload
- Settings changes

No error shall cause runtime failure.

---

# 53. Symbol and Timeframe Changes

When symbol or timeframe changes:

- Alert deduplication state shall reset safely where required.
- Active Setup IDs shall be reinitialised by upstream modules.
- No stale event shall alert on the new context.
- Alert history may be cleared or namespaced by symbol and timeframe.

---

# 54. Settings Changes

When alert settings change:

- Existing confirmed analytical events shall not be regenerated automatically.
- New publication behaviour applies from the current calculation context.
- Duplicate suppression shall remain safe.
- Enabling an event shall not necessarily alert an old persistent state unless an explicit current-state alert option exists.

---

# 55. Current-State Alert Option

Optional input:

```text
Alert Current State on Enable
```

Suggested default:

Disabled

When enabled, a current persistent state may alert after settings reload.

This may cause unexpected alerts and shall be clearly documented.

---

# 56. Accessibility and Clarity

Alert messages shall:

- Avoid ambiguous abbreviations in Standard mode.
- Use direction words explicitly.
- Avoid colour-dependent meaning.
- Avoid promotional language.
- Avoid claims of certainty.
- Avoid language such as:
  - Guaranteed
  - Certain
  - Risk-free
  - Institutions are definitely buying

Preferred language:

```text
Bullish Institutional Bias
```

```text
Long Ready
```

```text
Spring confirmed
```

---

# 57. Performance Requirements

The Alerts module shall:

- Process only eligible current-bar events.
- Avoid full-history rescanning.
- Use bounded alert history.
- Avoid building messages for rejected events.
- Build only the selected message format.
- Limit evidence-item loops.
- Use stable enumerations and helper functions.
- Reuse formatting utilities.
- Skip disabled modules early.
- Avoid multiple publications of the same event.
- Maintain deterministic execution order.

---

# 58. Testing Requirements

The Alerts module shall be tested across:

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

- One event on one bar
- Multiple compatible events on one bar
- Multiple conflicting events on one bar
- Duplicate source event
- State transition
- Persistent unchanged state
- Cooldown active
- Cooldown override
- Maximum alerts per bar reached
- Confidence below threshold
- Setup Quality below threshold
- Entry Readiness below threshold
- Missing context
- Signal invalidation
- Signal expiry
- Same-bar Ready and Triggered
- Developing event
- Confirmed event
- Webhook message
- Missing field
- Symbol change
- Timeframe change
- Settings change

---

# 59. Functional Test Cases

## Test ALERT-001 — Confirmed Signal Alert

Given:

- Long Ready confirms at bar close.
- Confidence exceeds threshold.
- Setup Quality and Entry Readiness exceed thresholds.
- Event has not previously alerted.

Expected:

- One Long Ready alert is published.
- Alert ID is stored.
- Message contains required fields.

---

## Test ALERT-002 — Candidate Suppression

Given:

- Long Ready is provisional intrabar.
- Confirmation Mode is Confirmed Bar Only.

Expected:

- No alert is published.

---

## Test ALERT-003 — Duplicate Suppression

Given:

- The same Long Ready event remains active for five bars.
- State Change Only is enabled.

Expected:

- One alert is published on state entry.
- No repeated alerts occur.

---

## Test ALERT-004 — Reinforcement Alert

Given:

- Bullish Qualified alerted previously.
- State advances to Long Ready.
- Alert on Reinforcement is enabled.

Expected:

- A new Long Ready alert is published.
- It uses a distinct Alert ID.

---

## Test ALERT-005 — Same-Bar Highest Priority

Given:

- Sell-Side Sweep confirms.
- Bullish CHOCH confirms.
- Long Ready confirms.
- Maximum Alerts Per Bar equals 1.
- Same-Bar Policy is Highest Priority Only.

Expected:

- Long Ready alert is published.
- Sweep and CHOCH appear as supporting evidence where enabled.
- Separate lower-priority alerts are suppressed.

---

## Test ALERT-006 — Conflicting Same-Bar Events

Given:

- Bullish event and Bearish event both qualify.
- One is Signal Invalidated.
- One is a lower-priority VPA event.

Expected:

- Signal Invalidated is published.
- Lower-priority event is suppressed.

---

## Test ALERT-007 — Cooldown Suppression

Given:

- Bullish BOS alerted two bars ago.
- Cooldown equals three bars.
- Another eligible Bullish BOS occurs.

Expected:

- Second alert is suppressed if policy treats it as similar and no override applies.
- Rejection reason is Cooldown Active.

---

## Test ALERT-008 — New Setup Bypasses Duplicate Rule

Given:

- One Long Ready setup alerted previously.
- A new setup with a different Setup ID qualifies.

Expected:

- New alert is eligible.
- It is not treated as a duplicate.

---

## Test ALERT-009 — Invalidation Bypasses Cooldown

Given:

- Long Ready alerted one bar ago.
- Setup invalidates.
- Alert on Invalidation is enabled.

Expected:

- Invalidation alert publishes despite directional cooldown.

---

## Test ALERT-010 — Confidence Filter

Given:

- Spring confirms with confidence 66.
- Minimum Wyckoff Confidence equals 70.

Expected:

- No Spring alert is published.
- Event remains stored upstream.

---

## Test ALERT-011 — Signal Quality Filter

Given:

- Long Ready state exists.
- Setup Quality equals 62.
- Minimum Setup Quality equals 65.

Expected:

- Alert is withheld.
- Rejection reason identifies Setup Quality.

---

## Test ALERT-012 — Entry Readiness Filter

Given:

- Bullish Qualified state exists.
- Entry Readiness equals 68.
- Minimum Entry Readiness equals 70.

Expected:

- No Long Ready alert is published.

---

## Test ALERT-013 — State Change Only

Given:

- Institutional state remains Accumulation Bias for ten bars.

Expected:

- One alert occurs on entry.
- No repeat alerts occur while unchanged.

---

## Test ALERT-014 — Opposing Signal Override

Given:

- Long signal alerted recently.
- Long setup invalidates.
- Strong Short Triggered event confirms.
- Opposing Signal Override is enabled.

Expected:

- Short Triggered alert bypasses same-direction or global cooldown as approved.

---

## Test ALERT-015 — Combined Compatible Events

Given:

- Long Ready, Spring Test, and Bullish BOS confirm on one bar.
- Combine Compatible Events is enabled.

Expected:

- One Long Ready alert is published.
- Supporting evidence includes Spring Test and Bullish BOS.

---

## Test ALERT-016 — Webhook JSON

Given:

- Webhook JSON format is enabled.
- Long Triggered qualifies.

Expected:

- Message uses stable field names.
- Numeric and string fields are valid.
- Missing optional values are safely omitted or represented.
- Message is deterministic.

---

## Test ALERT-017 — Missing Event Level

Given:

- Trend state changes.
- No event price level applies.
- Include Event Level is enabled.

Expected:

- Alert still publishes.
- Event level is omitted or shown as N/A.
- No runtime error occurs.

---

## Test ALERT-018 — Developing Alert

Given:

- Developing and Confirmed mode is enabled.
- A provisional Spring candidate appears intrabar.

Expected:

- Provisional alert may publish according to policy.
- Message is marked PROVISIONAL.
- A later confirmed Spring receives a separate confirmation identity.

---

## Test ALERT-019 — Event Cancellation

Given:

- Provisional intrabar event alerts.
- Event disappears before bar close.

Expected:

- No confirmed alert is published.
- Previously delivered provisional external alert cannot be retracted.
- Internal confirmed history does not record the event as confirmed.

---

## Test ALERT-020 — Rate Limit

Given:

- Three alertable events confirm on one bar.
- Maximum Alerts Per Bar equals one.

Expected:

- Highest-priority event publishes.
- Remaining events are rejected with Rate Limit or Priority reason.

---

## Test ALERT-021 — Settings Enablement

Given:

- Liquidity alerts were disabled.
- A Sweep occurred previously.
- Liquidity alerts are enabled later.
- Current-State-on-Enable is disabled.

Expected:

- No historical Sweep alert is generated.

---

## Test ALERT-022 — Symbol Change

Given:

- Alert state exists on one symbol.
- Chart symbol changes.

Expected:

- No stale event alerts on the new symbol.
- Deduplication state resets safely.

---

## Test ALERT-023 — Timeframe Change

Given:

- Active setup exists on one timeframe.
- Timeframe changes.

Expected:

- Old Setup ID is not reused incorrectly.
- No stale signal alert is published.

---

## Test ALERT-024 — Historical Stability

Given:

- Long Triggered alert was confirmed.
- Additional future bars are loaded.

Expected:

- Stored alert event identity, bar, message snapshot, and confidence remain unchanged.

---

# 60. Acceptance Criteria

The Alerts module shall be complete when:

- All alertable upstream modules can publish events through one central pipeline.
- No analytical classification occurs inside Alerts.
- Confirmed-bar publication works.
- Developing alerts remain optional and visibly provisional.
- Module and event enablement work.
- Confidence, score, quality, and readiness filters work.
- Duplicate suppression works.
- Cooldown works.
- Same-bar priority resolution works.
- Compatible event combination works.
- Maximum Alerts Per Bar is enforced.
- Persistent states alert only on transition by default.
- Reinforcement alerts use distinct identities.
- Invalidation alerts operate correctly.
- Signal Ready and Triggered alerts operate correctly.
- Message formats are consistent.
- Webhook JSON is deterministic.
- Unavailable values are handled safely.
- Alert history remains bounded.
- Confirmed alert snapshots remain historically stable.
- No runtime errors occur.
- TradingView compilation succeeds.
- Required tests pass.

---

# 61. Open Design Decisions

The following items remain unresolved:

1. Final default Alert Mode.
2. Final alert publication method.
3. Number of static `alertcondition()` entries.
4. Whether dynamic alerts are enabled by default.
5. Final event-priority order.
6. Final severity mapping.
7. Final same-bar policy.
8. Whether compatible events are combined by default.
9. Final Maximum Alerts Per Bar.
10. Final global cooldown.
11. Final directional cooldown.
12. Final cooldown scope.
13. Final cooldown exemptions.
14. Final rate-limiting policy.
15. Final duplicate Alert ID format.
16. Final alert-history size.
17. Final current-state-on-enable policy.
18. Whether Watch alerts are exposed in production.
19. Whether Qualified alerts are enabled by default.
20. Whether Ready and Triggered both alert by default.
21. Whether same-bar Ready and Triggered produces one or two alerts.
22. Final signal invalidation priority.
23. Final signal expiry policy.
24. Whether No-Trade alerts are supported in Version 1.0.
25. Final Trend alert set.
26. Final RVOL alert set.
27. Final Market Structure alert set.
28. Final BOS and CHOCH alert set.
29. Final Liquidity alert set.
30. Final VPA alert set.
31. Final Wyckoff alert set.
32. Final Institutional alert set.
33. Final Signal alert set.
34. Final Compact message fields.
35. Final Standard message fields.
36. Final Detailed message fields.
37. Final webhook field schema.
38. Final webhook schema version.
39. Whether null fields are omitted or represented as strings.
40. Final maximum message length policy.
41. Final evidence-item count in messages.
42. Final score decimal precision.
43. Final price formatting policy.
44. Final provisional-alert identity policy.
45. Whether a confirmed event follows a provisional alert automatically.
46. Whether failed events may alert.
47. Whether weakened states may alert.
48. Whether Completed setup alerts are supported.
49. Final symbol and timeframe reset policy.
50. Final diagnostic alert support.

---

# 62. Future Enhancements

Possible future enhancements include:

- External webhook schema documentation
- Alert templates for popular automation platforms
- Broker-routing integrations
- Discord message templates
- Telegram message templates
- Email-specific templates
- Per-event custom message overrides
- User-defined webhook fields
- Alert batching
- Alert acknowledgement tracking
- Persistent external event ledger
- Strategy execution integration
- Portfolio alert aggregation
- Multi-symbol alert orchestration
- Multi-timeframe alert confirmation
- Session-aware alert suppression
- News-event alert filters
- Adaptive rate limiting
- Alert analytics
- False-alert review tools
- Historical alert replay