# Data Flow Architecture

---

## Document Information

**Document ID:** ARCH-205

**Version:** 0.1 Draft

**Status:** Draft

**Purpose:** Define end-to-end data movement, execution timing, ownership, event propagation, and publication flow.

**Related Documents:**

- `IMPLEMENTATION_CONTRACTS.md`
- `MODULE_DEPENDENCY_MATRIX.md`
- `SHARED_TYPES_AND_ENUMS.md`
- `MODULE_API.md`
- `STATE_MACHINES.md`

---

# 1. Purpose

This document defines how information moves through the indicator.

It covers:

- Raw market data
- Settings resolution
- Shared framework context
- Analytical outputs
- Structural events
- Evidence creation
- Institutional aggregation
- Signal scoring
- Dashboard rendering
- Alert routing
- Historical storage
- Cleanup
- Debug publication

The purpose is to prevent:

- Circular dependencies
- Same-bar timing errors
- Hidden state access
- Duplicate calculations
- Stale output consumption
- Event backdating
- Unbounded data propagation

---

# 2. End-to-End Flow

```text
TradingView Inputs
        │
        ▼
Settings Resolution
        │
        ▼
Effective Settings
        │
        ▼
Framework Context
        │
        ├───────────────┬───────────────┐
        ▼               ▼               ▼
      Trend            RVOL       Market Structure
                                           │
                                           ▼
                                      BOS / CHOCH
                                           │
                                           ▼
                                        Liquidity
        │               │                   │
        └───────────────┴──────────┬────────┘
                                   ▼
                                  VPA
                                   │
                                   ▼
                                Wyckoff
                                   │
             Confirmed Events ─────┴─────
                                   │
                                   ▼
                           Shared Evidence
                                   │
                                   ▼
                       Institutional Activity
                                   │
                                   ▼
                          Signal Scoring
                              │       │
                              ▼       ▼
                         Dashboard   Alerts
```

---

# 3. Input Layer

The Input layer consists of TradingView input controls.

It publishes raw user choices only.

Examples:

- Profile
- Module enablement
- Thresholds
- Display settings
- Alert settings
- Debug settings

Raw inputs shall not be consumed directly by analytical modules.

They shall first pass through Settings resolution.

---

# 4. Settings Resolution Flow

```text
Raw Inputs
   │
   ▼
Profile Defaults
   │
   ▼
Module Overrides
   │
   ▼
Numeric Validation
   │
   ▼
Cross-Setting Validation
   │
   ▼
Dependency Validation
   │
   ▼
Safety Clamps
   │
   ▼
Effective Settings
```

The Effective Settings output is immutable for the duration of one script execution.

---

# 5. Framework Context Flow

The Framework receives:

- OHLCV data
- Chart metadata
- Effective general settings
- Effective theme settings
- Effective performance settings

The Framework publishes:

- Bar state
- Symbol and timeframe context
- ATR
- Volume availability
- Tick size
- Price precision
- Minimum-history state
- Theme colours
- Object-budget metadata
- Debug state

All modules may consume Framework context.

---

# 6. Trend Data Flow

```text
Price
+
Framework Context
+
Trend Settings
        │
        ▼
Trend Classification
        │
        ▼
Trend Output
```

Trend output may be consumed by:

- Wyckoff
- Institutional Activity
- Signal Scoring
- Dashboard
- Alerts

Trend does not consume downstream analytical outputs.

---

# 7. RVOL Data Flow

```text
Volume
+
Framework Volume State
+
RVOL Settings
        │
        ▼
Relative Volume Calculation
        │
        ▼
Participation Classification
        │
        ▼
RVOL Output
```

RVOL output may be consumed by:

- VPA
- Wyckoff
- Institutional Activity
- Signal Scoring
- Dashboard
- Alerts

RVOL remains non-directional.

---

# 8. Market Structure Data Flow

```text
Price
+
Framework Context
+
Structure Settings
        │
        ▼
Pivot Detection
        │
        ▼
Pivot Confirmation
        │
        ▼
Swing History
        │
        ▼
Structural Context
        │
        ▼
Structure Output
```

Structure output is consumed by:

- BOS/CHOCH
- Liquidity
- Wyckoff
- Institutional Activity
- Signal Scoring
- Dashboard
- Alerts

Confirmed pivots shall not be rewritten downstream.

---

# 9. BOS and CHOCH Data Flow

```text
Structure Output
+
Current Price
+
BOS/CHOCH Settings
        │
        ▼
Break Candidate
        │
        ▼
Close Confirmation
        │
        ▼
Acceptance or Failure
        │
        ▼
Primary Structural Event
```

The structural event contains:

- Broken level
- Source Swing ID
- Direction
- Event type
- Confidence
- Confirmation bar
- Lifecycle

It is consumed by:

- Liquidity
- Wyckoff
- Institutional Activity
- Signal Scoring
- Dashboard
- Alerts
- Shared Evidence

---

# 10. Liquidity Data Flow

```text
Structure Levels
+
BOS/CHOCH Events
+
Current Price Interaction
+
Liquidity Settings
        │
        ▼
Level Registration
        │
        ▼
Interaction Detection
        │
        ▼
Sweep / Acceptance / Trap Resolution
        │
        ▼
Primary Liquidity Event
```

Liquidity output is consumed by:

- VPA as optional context
- Wyckoff
- Institutional Activity
- Signal Scoring
- Dashboard
- Alerts
- Shared Evidence

---

# 11. VPA Data Flow

```text
Price Spread
+
Close Location
+
Raw Volume
+
RVOL Output
+
Optional Liquidity Context
+
VPA Settings
        │
        ▼
Spread Classification
        │
        ▼
Volume Classification
        │
        ▼
Effort-versus-Result
        │
        ▼
Contextual Classification
        │
        ▼
Primary VPA Event
```

Only one primary VPA event is published per confirmed bar.

---

# 12. Wyckoff Data Flow

```text
Structure
+
BOS/CHOCH
+
Liquidity
+
VPA
+
RVOL
+
Optional Trend
+
Wyckoff Settings
        │
        ▼
Range Detection
        │
        ▼
Range Lifecycle
        │
        ▼
Schematic Hypothesis
        │
        ▼
Phase State
        │
        ▼
Sequence Validation
        │
        ▼
Primary Wyckoff Event
```

Wyckoff output is consumed by:

- Institutional Activity
- Signal Scoring
- Dashboard
- Alerts
- Shared Evidence

---

# 13. Event-to-Evidence Flow

Confirmed upstream events are transformed into evidence.

```text
Confirmed Module Event
        │
        ▼
Validate Event Metadata
        │
        ▼
Assign Evidence Categories
        │
        ▼
Assign Correlation Group
        │
        ▼
Assign Persistence Class
        │
        ▼
Create Evidence ID
        │
        ▼
Insert or Update Evidence Record
```

Only approved events become evidence.

Persistent states shall not insert duplicate evidence every bar.

---

# 14. Evidence Source Flow

Possible evidence sources:

```text
Trend transition
RVOL expansion
New pivot or structural context
BOS or CHOCH
Liquidity Sweep
Liquidity Trap
Accepted Break
VPA classification
Wyckoff event
Wyckoff phase transition
Institutional state transition
Signal transition
```

Signal evidence shall not flow back into Institutional Activity.

This prevents circular aggregation.

---

# 15. Evidence Lifecycle Flow

```text
Candidate
   │
   ▼
Confirmed
   │
   ├── Additional support → Reinforced
   ├── Deterioration → Weakening
   ├── Failure → Failed
   └── Age limit → Expired
```

Evidence lifecycle changes shall not modify the original upstream event.

---

# 16. Institutional Aggregation Flow

```text
Active Evidence
        │
        ▼
Eligibility Filtering
        │
        ▼
Confidence Scaling
        │
        ▼
Lifecycle Scaling
        │
        ▼
Recency Decay
        │
        ▼
Correlation Control
        │
        ▼
Module and Category Caps
        │
        ▼
Bullish and Bearish Scores
        │
        ▼
Accumulation / Distribution Scores
        │
        ▼
Continuation / Reversal Scores
        │
        ▼
Conflict Resolution
        │
        ▼
Institutional State
```

Institutional output becomes a capped consensus input to Signal Scoring.

---

# 17. Signal Scoring Flow

```text
Active Evidence
+
Institutional Consensus
+
Structural Context
+
Trigger Context
+
Price Location
+
Scoring Settings
        │
        ▼
Evidence Eligibility
        │
        ▼
Duplicate and Correlation Control
        │
        ▼
Directional Scores
        │
        ▼
Continuation and Reversal Scores
        │
        ▼
Conflict
        │
        ▼
Setup Type Selection
        │
        ▼
Setup Quality
        │
        ▼
Entry Readiness
        │
        ▼
Signal Confidence
        │
        ▼
Setup Lifecycle
        │
        ▼
Primary Signal State
```

---

# 18. Bullish and Bearish Setup Flow

The scoring engine may maintain:

```text
Bullish Setup
Bearish Setup
```

in parallel.

Each setup receives:

- Setup ID
- Setup type
- Lifecycle
- Quality
- Readiness
- Confidence
- Trigger level
- Invalidation level
- Age

The engine publishes one primary state based on precedence, strength, conflict, and lifecycle.

---

# 19. Same-Bar Event Flow

One confirmed bar may produce:

```text
Bullish CHOCH
Sell-Side Sweep
Bullish Absorption
Spring Test
Long Ready
```

Required sequence:

```text
Structure confirms CHOCH
        │
        ▼
Liquidity interprets Sweep
        │
        ▼
VPA classifies Absorption
        │
        ▼
Wyckoff updates Spring sequence
        │
        ▼
Evidence records become available
        │
        ▼
Institutional state updates
        │
        ▼
Signal Scoring resolves Long Ready
        │
        ▼
Dashboard displays resolved state
        │
        ▼
Alerts publish highest-priority event
```

All events use the current confirmation bar.

No event is backdated.

---

# 20. Dashboard Data Flow

```text
Module Output Snapshots
+
Configuration Warnings
+
Dashboard Settings
+
Theme
        │
        ▼
Row Visibility Resolution
        │
        ▼
State-to-Text Conversion
        │
        ▼
Semantic Formatting
        │
        ▼
Persistent Table Update
```

The Dashboard does not read private module arrays.

It consumes published snapshots only.

---

# 21. Alert Data Flow

```text
New Event Channels
+
Signal Transition
+
Supporting Evidence
+
Alert Settings
        │
        ▼
Eligibility
        │
        ▼
Threshold Filters
        │
        ▼
Duplicate Suppression
        │
        ▼
Cooldown
        │
        ▼
Same-Bar Priority
        │
        ▼
Message Formatting
        │
        ▼
Publication
        │
        ▼
Alert History Record
```

Alert routing occurs after the complete analytical pipeline.

---

# 22. Historical Storage Flow

Persistent records may include:

- Swings
- Structural events
- Liquidity levels
- Liquidity interactions
- VPA events
- Wyckoff ranges
- Wyckoff events
- Evidence
- Institutional transitions
- Signal setups
- Alert records
- Visual object references

Each collection shall define:

- Owner
- Maximum size
- Insertion rule
- Update rule
- Expiry rule
- Deletion rule

---

# 23. Cleanup Flow

Cleanup occurs after current-bar output publication.

```text
Current-Bar Analysis Complete
        │
        ▼
Dashboard Updated
        │
        ▼
Alerts Processed
        │
        ▼
Expire Old Records
        │
        ▼
Delete Expired Visuals
        │
        ▼
Enforce Array Limits
        │
        ▼
Enforce Object Budget
```

Active records required by current state shall not be deleted.

---

# 24. Debug Data Flow

Debug mode may consume:

- Effective settings
- Module availability
- Internal enum states
- Candidate events
- Confirmed events
- Active IDs
- Evidence weights
- Scores
- Array counts
- Alert rejection reasons

Debug outputs shall not feed back into analytics.

---

# 25. Ownership Boundaries

| Data | Owner |
|---|---|
| Effective settings | Settings |
| Bar and symbol context | Framework |
| Trend state | Trend |
| RVOL state | RVOL |
| Swing records | Market Structure |
| Structural-break records | BOS/CHOCH |
| Liquidity levels and interactions | Liquidity |
| VPA classifications | VPA |
| Ranges and phases | Wyckoff |
| Shared evidence records | Evidence layer |
| Institutional state | Institutional |
| Setup records | Signal Scoring |
| Dashboard table | Dashboard |
| Alert records | Alerts |

Only the owner may mutate the data.

Downstream modules receive read-only semantic outputs.

---

# 26. Prohibited Data Flows

The following are prohibited:

```text
Dashboard → Analytical module
Alerts → Analytical module
Signal Scoring → Institutional classification
Institutional → Wyckoff classification
Wyckoff → Structure history
Liquidity → BOS confirmation history
VPA → RVOL baseline
Analytical module → Raw input mutation
```

---

# 27. Availability Propagation

## Hard Dependency Failure

```text
Upstream unavailable
        │
        ▼
Dependent module unavailable
```

## Soft Dependency Failure

```text
Upstream unavailable
        │
        ▼
Contribution omitted
        │
        ▼
Confidence reduced where specified
```

## Optional Dependency Failure

```text
Upstream unavailable
        │
        ▼
Enhancement omitted
        │
        ▼
Core calculation continues
```

Missing data shall never be converted automatically into opposing evidence.

---

# 28. Confirmation Timing

Production flow:

```text
Developing calculation
        │
        ▼
Bar closes
        │
        ▼
Upstream event confirms
        │
        ▼
Downstream modules consume event
        │
        ▼
Final state confirms
        │
        ▼
Dashboard and Alerts publish
```

Developing values shall use separate lifecycle state or variables.

---

# 29. Symbol and Timeframe Change Flow

On chart-context change:

```text
TradingView recalculation
        │
        ▼
Settings re-resolve
        │
        ▼
Framework context reinitialises
        │
        ▼
Persistent module state rebuilds
        │
        ▼
IDs regenerate within new context
        │
        ▼
Dashboard clears stale cells
        │
        ▼
Alert deduplication resets safely
```

No state from the prior chart context shall remain active.

---

# 30. Settings Change Flow

On input change:

```text
Historical recalculation
        │
        ▼
Effective settings change
        │
        ▼
All dependent states rebuild
        │
        ▼
Historical results may differ
```

This is configuration recalculation, not repainting under unchanged settings.

Old external alerts cannot be withdrawn.

---

# 31. Performance Flow Rules

The implementation shall:

- Calculate shared metrics once.
- Publish reusable outputs.
- Process only new events.
- Update persistent state incrementally.
- Avoid full-array rescans where possible.
- Avoid building strings before presentation stages.
- Skip disabled modules early.
- Avoid visual-object creation when hidden.
- Clean collections incrementally.
- Keep loop bounds explicit.

---

# 32. Testing Data Flow

Tests should observe published outputs rather than private implementation details.

Recommended test snapshot:

```text
symbol
timeframe
profile
barTime
moduleAvailability
moduleState
eventType
eventId
confirmationBar
confidence
institutionalState
signalState
setupId
alertEligibility
```

---

# 33. Data Flow Acceptance Criteria

Data-flow architecture is complete when:

- Every source and destination is documented.
- Processing order is deterministic.
- Same-bar consumption is defined.
- Ownership boundaries are explicit.
- Circular flows are prohibited.
- Event-to-evidence conversion is defined.
- Institutional and Scoring flows are distinct.
- Dashboard and Alerts remain downstream-only.
- Cleanup occurs after publication.
- Availability propagation is defined.
- Historical and real-time timing are defined.
- Testing can observe stable published outputs.![alt text](image.png)
