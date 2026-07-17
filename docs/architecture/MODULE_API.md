# Module API Contracts

---

## Document Information

**Document ID:** ARCH-203

**Version:** 0.1 Draft

**Status:** Draft

**Purpose:** Define the public interface, ownership boundaries, inputs, outputs, update sequence, and integration contract for every module.

**Related Documents:**

- `IMPLEMENTATION_CONTRACTS.md`
- `MODULE_DEPENDENCY_MATRIX.md`
- `SHARED_TYPES_AND_ENUMS.md`
- `STATE_MACHINES.md`
- `DATA_FLOW.md`
- Module specifications 01 through 13

---

# 1. Purpose

This document defines the public application programming interface of every project module.

In this project, a module API describes:

- What a module may consume
- What a module must publish
- What state it owns
- Which functions it exposes
- Which objects it owns
- Which events it may create
- Which dependencies it requires
- Which behaviours are prohibited

The purpose is to prevent hidden coupling between Pine Script components.

---

# 2. API Design Principles

Every module shall:

- Have one clearly defined responsibility.
- Consume resolved effective settings.
- Consume approved upstream outputs only.
- Publish a stable current-bar snapshot.
- Own its persistent state.
- Own its visual objects.
- Avoid mutating upstream state.
- Avoid performing downstream interpretation.
- Distinguish candidate and confirmed values.
- Preserve confirmed historical events.
- Use bounded persistent storage.
- Publish availability and diagnostic status.

---

# 3. Standard Module API Pattern

Each module should conceptually expose:

```text
validate()
update()
resolveState()
publishOutput()
registerVisualIntent()
cleanup()
```

Pine Script may implement these as several helper functions rather than one object-oriented interface.

Recommended processing pattern:

```text
validate
→ update persistent state
→ detect candidate
→ confirm event
→ resolve primary state
→ publish output
→ register visuals
```

---

# 4. Base Module Output

Every analytical module shall publish the equivalent of:

```text
ModuleOutput
{
    enabled
    available
    sufficientHistory
    configurationValid
    dependencyValid
    state
    direction
    lifecycle
    confidence
    eventId
    newEvent
    creationBar
    confirmationBar
}
```

Module-specific output contracts extend this base record.

---

# 5. Framework API

## Responsibility

The Framework owns shared runtime and chart context.

## Consumes

- Raw OHLCV series
- Symbol metadata
- Timeframe metadata
- Resolved general settings
- Resolved theme settings
- Resolved performance settings

## Publishes

```text
FrameworkOutput
{
    barIndex
    barConfirmed
    barRealtime
    barHistorical
    symbol
    timeframe
    tickSize
    pricePrecision
    volumeAvailable
    volumeUsable
    volumeQuality
    atr
    minimumHistoryMet
    theme
    debugEnabled
    schemaVersion
    objectBudget
}
```

## Owns

- Shared ATR
- Shared bar-state interpretation
- Volume availability state
- Theme semantic colours
- Global object-budget metadata
- Shared conversion functions
- Common safety helpers

## Must Not

- Detect market events
- Calculate Trend
- Calculate RVOL
- Score signals
- Route alerts

---

# 6. Settings API

## Responsibility

The Settings module resolves user inputs into validated effective settings.

## Consumes

- Raw TradingView inputs
- Profile selection
- Configuration level
- Module enablement
- User overrides

## Publishes

```text
EffectiveSettings
{
    general
    trend
    rvol
    structure
    bosChoch
    liquidity
    vpa
    wyckoff
    institutional
    scoring
    dashboard
    alerts
    visuals
    debug
}
```

It shall also publish:

```text
ConfigurationStatus
{
    valid
    warningCount
    highestWarningSeverity
    highestWarningCode
}
```

## Owns

- Profile resolution
- Input validation
- Cross-setting validation
- Dependency validation
- Safety clamps
- Effective configuration publication
- Configuration-warning generation

## Must Not

- Modify TradingView input controls
- Perform market analysis
- Override explicit Custom settings silently
- Enable user-disabled modules silently

---

# 7. Trend API

## Responsibility

Classify current directional trend and trend strength.

## Consumes

- Framework context
- Effective Trend settings
- Price series
- Optional higher-timeframe data where approved

## Publishes

```text
TrendOutput
{
    base
    trendState
    trendDirection
    trendStrength
    trendConfidence
    slopeState
    alignmentState
    newTrendEvent
    trendEventId
    previousTrendState
}
```

## Owns

- Trend calculations
- Trend hysteresis
- Trend state transitions
- Trend event history
- Trend visuals

## Must Not

- Detect BOS
- Interpret liquidity
- Classify Wyckoff
- Generate signal states

---

# 8. RVOL API

## Responsibility

Measure participation and relative volume without assigning directional intent.

## Consumes

- Framework volume context
- Effective RVOL settings
- Volume series

## Publishes

```text
RvolOutput
{
    base
    rvolValue
    rvolState
    volumeClass
    participationConfidence
    volumeAvailable
    volumeUsable
    newRvolEvent
    rvolEventId
}
```

## Owns

- Volume baseline
- RVOL calculation
- Participation classification
- Volume-quality diagnostics

## Must Not

- Infer bullish or bearish direction independently
- Classify VPA
- Detect institutional activity

---

# 9. Market Structure API

## Responsibility

Detect confirmed pivots and maintain structural context.

## Consumes

- Framework context
- Effective Market Structure settings
- Price series

## Publishes

```text
StructureOutput
{
    base
    structuralDirection
    structureConfidence

    lastSwingHighPrice
    lastSwingHighBar
    lastSwingHighId

    lastSwingLowPrice
    lastSwingLowBar
    lastSwingLowId

    bullishProtectionLevel
    bearishProtectionLevel

    equalHighActive
    equalLowActive

    newSwingHigh
    newSwingLow
    newStructureEvent
}
```

## Owns

- Pivot confirmation
- Swing IDs
- Swing history
- Structural direction
- Protection levels
- Equal-high and equal-low context
- Structure visuals

## Must Not

- Confirm BOS or CHOCH
- Interpret sweeps
- Rewrite confirmed pivots

---

# 10. BOS and CHOCH API

## Responsibility

Interpret confirmed structural breaks.

## Consumes

- Framework context
- Effective BOS/CHOCH settings
- Market Structure output
- Current price bar

## Publishes

```text
BosChochOutput
{
    base
    primaryEventType
    eventDirection
    eventConfidence
    brokenLevel
    sourceSwingId
    structuralEventId
    acceptanceStatus
    failureStatus
    structuralRegime
    newPrimaryEvent
}
```

## Owns

- Structural-break candidates
- Break confirmation
- Acceptance state
- Failure state
- BOS/CHOCH event IDs
- BOS/CHOCH visuals

## Must Not

- Create pivots
- Reclassify Market Structure history
- Interpret a break as a liquidity event

---

# 11. Liquidity API

## Responsibility

Interpret price interactions with structural liquidity levels.

## Consumes

- Framework context
- Effective Liquidity settings
- Market Structure output
- BOS/CHOCH output
- Price series

## Publishes

```text
LiquidityOutput
{
    base
    primaryEventType
    eventDirection
    eventConfidence

    liquidityLevelId
    liquidityInteractionId
    levelPrice
    levelType
    levelPriority

    sweepStatus
    acceptanceStatus
    trapStatus
    clearedStatus

    newPrimaryEvent
}
```

## Owns

- Liquidity-level records
- Liquidity interaction records
- Sweep detection
- Accepted-break classification
- Trap classification
- Liquidity visuals

## Must Not

- Confirm Market Structure
- Rewrite BOS/CHOCH events
- Produce Signal Ready states

---

# 12. VPA API

## Responsibility

Classify the relationship between price spread, close location, volume, effort, and result.

## Consumes

- Framework context
- Effective VPA settings
- Price series
- RVOL output
- Optional Liquidity context

## Publishes

```text
VpaOutput
{
    base
    primaryClassification
    eventDirection
    eventConfidence

    spreadClass
    volumeClass
    closeLocationClass
    effortResultClass

    vpaEventId
    newPrimaryEvent
}
```

## Owns

- Spread classification
- Close-location classification
- Effort-versus-result classification
- Primary VPA event priority
- VPA event history
- VPA visuals

## Must Not

- Modify RVOL
- Detect structural breaks
- Publish multiple primary classifications for one confirmed bar

---

# 13. Wyckoff API

## Responsibility

Maintain trading ranges, schematic hypotheses, phases, and sequence-based Wyckoff events.

## Consumes

- Framework context
- Effective Wyckoff settings
- Market Structure output
- BOS/CHOCH output
- Liquidity output
- VPA output
- RVOL output
- Optional Trend output

## Publishes

```text
WyckoffOutput
{
    base
    rangeActive
    rangeId
    rangeHigh
    rangeLow
    rangeMidpoint

    schematic
    phase
    primaryEvent
    eventDirection
    eventConfidence

    rangeConfidence
    phaseConfidence
    sequenceValid
    rangeInvalidated

    wyckoffEventId
    newPrimaryEvent
}
```

## Owns

- Trading-range records
- Range IDs
- Range boundaries
- Schematic hypotheses
- Phase state
- Sequence state
- Wyckoff event history
- Wyckoff visuals

## Must Not

- Rewrite structural pivots
- Treat a single bar as sufficient for schematic confirmation
- Publish institutional certainty

---

# 14. Shared Evidence API

## Responsibility

Convert approved upstream events into standard downstream evidence records.

## Consumes

- Confirmed upstream module events
- Event metadata
- Lifecycle metadata
- Confidence values
- Correlation classifications

## Publishes

```text
EvidenceRecord
{
    evidenceId
    sourceModule
    sourceEventId
    eventType
    direction
    categoryFlags
    lifecycle
    confidence
    baseWeight
    effectiveWeight
    creationBar
    confirmationBar
    eventLevel
    correlationGroup
    persistenceClass
    contextFlags
}
```

## Owns

- Evidence ID allocation
- Evidence record insertion
- Evidence lifecycle updates
- Evidence expiry
- Duplicate suppression
- Evidence history bounds

## Must Not

- Reinterpret upstream event identity
- Change upstream confirmation bars
- Assign unsupported direction to non-directional events

---

# 15. Institutional Activity API

## Responsibility

Aggregate observable evidence into an institutional activity interpretation.

## Consumes

- Framework context
- Effective Institutional settings
- Shared evidence records
- Optional direct module summaries

## Publishes

```text
InstitutionalOutput
{
    base
    primaryState
    stateDirection
    stateConfidence

    bullishScore
    bearishScore
    accumulationScore
    distributionScore
    bullishContinuationScore
    bearishContinuationScore
    bullishReversalScore
    bearishReversalScore
    bullishAbsorptionScore
    bearishAbsorptionScore
    conflictScore

    confirmingModuleCount
    strongestSupportingEvidenceId
    strongestOpposingEvidenceId

    stateEventId
    newStateEvent
}
```

## Owns

- Evidence aggregation
- Institutional correlation control
- Institutional confidence
- Institutional hysteresis
- Institutional state transitions

## Must Not

- Claim direct institutional detection
- Rewrite evidence records
- Change Wyckoff classifications
- Publish trade-entry states

---

# 16. Signal Scoring API

## Responsibility

Resolve evidence into setup quality, entry readiness, confidence, and signal lifecycle.

## Consumes

- Framework context
- Effective Scoring settings
- Shared evidence records
- Institutional output
- Structural and trigger metadata
- Current price location

## Publishes

```text
ScoringOutput
{
    base
    signalState
    signalDirection
    signalConfidence

    bullishCompositeScore
    bearishCompositeScore
    bullishContinuationScore
    bearishContinuationScore
    bullishReversalScore
    bearishReversalScore

    setupQuality
    entryReadiness
    conflictScore
    directionalDominance

    setupType
    setupId
    setupLifecycle

    setupCreationBar
    qualificationBar
    readyBar
    triggerBar

    triggerLevel
    invalidationLevel
    setupAge

    noTradeReason
    strongestSupportingEvidenceId
    strongestOpposingEvidenceId

    newStateEvent
}
```

## Owns

- Bullish setup state
- Bearish setup state
- Setup IDs
- Setup lifecycle
- Setup Quality
- Entry Readiness
- Signal Confidence
- No-Trade resolution
- Signal event history
- Signal visuals

## Must Not

- Execute trades
- Rewrite Institutional state
- Reclassify upstream evidence
- Publish repeated Triggered events for one Setup ID

---

# 17. Dashboard API

## Responsibility

Render the current published analytical state.

## Consumes

- Framework theme
- Effective Dashboard settings
- All module output snapshots
- Configuration warnings

## Publishes

The Dashboard publishes no analytical events.

Optional diagnostics may publish:

```text
DashboardStatus
{
    enabled
    tableCreated
    lastUpdateBar
    visibleRowCount
}
```

## Owns

- One table object
- Cell values
- Cell formatting
- Dashboard layout
- Dashboard evidence summary

## Must Not

- Perform analytical calculations
- Modify module state
- Create market events
- Route alerts

---

# 18. Alerts API

## Responsibility

Filter, prioritise, format, and publish approved events.

## Consumes

- Framework context
- Effective Alert settings
- Upstream new-event channels
- Signal state transitions
- Supporting evidence metadata

## Publishes

```text
AlertOutput
{
    candidatePresent
    published
    alertId
    sourceModule
    eventType
    severity
    rejectionReason
    publicationBar
}
```

## Owns

- Alert IDs
- Duplicate suppression
- Cooldowns
- Same-bar priority
- Rate limits
- Message construction
- Alert history

## Must Not

- Detect analytical events
- Modify upstream state
- Reclassify evidence
- Generate historical alerts retroactively

---

# 19. Visual Ownership API

Each module may expose visual intent rather than immediately creating objects.

Conceptual intent:

```text
VisualIntent
{
    owner
    visualType
    priority
    creationBar
    expiryBar
    price1
    price2
    text
    lifecycle
}
```

The initial implementation may create module-owned objects directly, provided ownership and limits remain explicit.

---

# 20. Cleanup API

Each persistent module shall expose an internal cleanup stage.

Cleanup may remove:

- Expired records
- Invalidated inactive records
- Old historical records beyond limits
- Deleted visual references
- Expired candidates

Cleanup shall not remove active records required by downstream modules.

---

# 21. Error and Diagnostic API

Every module should make available:

```text
lastErrorCode
lastWarningCode
activeRecordCount
historyRecordCount
candidatePresent
```

These values may remain debug-only.

No module shall rely on display strings as diagnostic state.

---

# 22. API Change Policy

A public module API changes when:

- A required output is added or removed.
- An enum meaning changes.
- An identifier relationship changes.
- A dependency changes.
- A lifecycle transition changes.
- A field changes semantic meaning.

Public API changes require:

- Architecture review
- ADR where material
- Changelog entry
- Regression-test update
- Dependent-module review

---

# 23. Acceptance Criteria

The Module API architecture is complete when:

- Every module has a defined responsibility.
- Every module has documented inputs and outputs.
- Persistent-state ownership is clear.
- Visual ownership is clear.
- Dependencies are explicit.
- Prohibited behaviours are documented.
- Downstream modules can be implemented without reading private upstream state.
- Testing can inspect deterministic published outputs.
