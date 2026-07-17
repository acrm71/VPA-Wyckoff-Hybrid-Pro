# Implementation Contracts

---

## Document Information

**Document ID:** ARCH-201

**Version:** 0.1 Draft

**Status:** Draft

**Purpose:** Architecture consolidation and implementation contract definition

**Applies To:**

- Framework
- Settings
- Trend
- RVOL
- Market Structure
- BOS and CHOCH
- Liquidity
- VPA
- Wyckoff
- Institutional Activity
- Signal Scoring
- Dashboard
- Alerts

---

# 1. Purpose

This document consolidates the module specifications into implementation-ready contracts.

It defines the shared rules that every module shall follow, including:

- Processing order
- Module boundaries
- State semantics
- Event semantics
- Enumeration governance
- Evidence transport
- Dependency handling
- Confidence handling
- Identity generation
- Lifecycle behaviour
- Historical stability
- Same-bar processing
- Missing-data handling
- Object ownership
- Alert publication
- Dashboard publication
- Performance constraints
- Testing interfaces

This document shall resolve cross-module ambiguity before Pine Script implementation begins.

Module specifications remain authoritative for module-specific analytical behaviour.

This document is authoritative for shared interfaces and integration behaviour.

---

# 2. Architectural Objective

The indicator shall operate as a deterministic analytical pipeline.

```text
Inputs
  │
  ▼
Effective Settings
  │
  ▼
Framework Context
  │
  ├── Trend
  ├── RVOL
  ├── Market Structure
  │      ▼
  │   BOS / CHOCH
  │      ▼
  │   Liquidity
  │
  ├── VPA
  └── Wyckoff
          ▼
   Institutional Activity
          ▼
     Signal Scoring
          ▼
   Dashboard and Alerts
```

Each module shall:

- Own one analytical responsibility.
- Consume approved upstream outputs.
- Publish stable outputs.
- Avoid recomputing upstream logic.
- Avoid mutating another module’s state.
- Preserve confirmed historical events.
- Handle unavailable dependencies safely.

---

# 3. Mandatory Processing Order

The production processing order shall be:

```text
1. Read user inputs
2. Resolve effective settings
3. Initialise framework context
4. Validate minimum history
5. Process Trend
6. Process RVOL
7. Process Market Structure
8. Process BOS and CHOCH
9. Process Liquidity
10. Process VPA
11. Process Wyckoff
12. Process Institutional Activity
13. Process Signal Scoring
14. Resolve visual events
15. Update Dashboard
16. Process Alerts
17. Perform bounded cleanup
18. Publish debug diagnostics
```

No downstream module shall run before its required upstream dependencies have completed for the current bar.

---

# 4. Same-Bar Data Contract

Every module shall consume upstream values resolved on the same script execution.

Example:

```text
Current confirmed Bullish CHOCH
```

may be consumed by:

- Liquidity
- Wyckoff
- Institutional Activity
- Signal Scoring
- Dashboard
- Alerts

on the same confirmed bar.

No module shall require a one-bar delay unless its own specification explicitly defines delayed confirmation.

---

# 5. Bar-State Contract

All modules shall distinguish:

```text
Historical confirmed bar
Real-time confirmed bar
Real-time developing bar
```

Production event confirmation shall use:

```text
barstate.isconfirmed
```

or an approved equivalent.

Developing-bar values may be calculated for:

- Dashboard preview
- Debugging
- Candidate visualisation
- Provisional alerts

Developing values shall never overwrite confirmed historical events.

---

# 6. State Semantics

Every module state shall use one of the following semantic classes.

## 6.1 Disabled

The user has disabled the module.

Required behaviour:

- Enabled flag = false
- State = Unavailable
- No analytical events
- No evidence publication
- No module alerts
- No module visuals

## 6.2 Unavailable

The module is enabled but cannot calculate a valid result.

Possible causes:

- Missing hard dependency
- Insufficient history
- Invalid configuration
- Required source data unavailable
- Unsupported context

Unavailable is not Neutral.

## 6.3 Neutral

The module calculated successfully but found no directional or classified state.

## 6.4 Candidate

A provisional event or state exists but is not confirmed.

## 6.5 Confirmed

An event or state has satisfied its confirmation requirements.

## 6.6 Reinforced

A confirmed event or state has gained additional supporting evidence.

## 6.7 Weakening

The state remains active but supporting evidence is deteriorating.

## 6.8 Failed

The event failed its expected confirmation or follow-through.

## 6.9 Invalidated

A persistent state or setup has been structurally invalidated.

## 6.10 Expired

An event or setup aged beyond its valid lifecycle without structural failure.

## 6.11 Completed

A setup or sequence reached its defined terminal state.

---

# 7. Common Availability Contract

Each module shall publish:

```text
enabled
available
sufficientHistory
configurationValid
dependencyValid
```

Suggested semantics:

```text
enabled = user intent
available = valid module result can be calculated
sufficientHistory = minimum data requirement met
configurationValid = settings are internally valid
dependencyValid = hard dependencies are available
```

A module is available only when all required conditions are satisfied.

---

# 8. Common Direction Contract

Directional outputs shall use:

```text
-1 = Bearish or Short
 0 = Neutral or Non-Directional
+1 = Bullish or Long
```

Unavailable direction shall not be represented by a misleading directional value alone.

It shall be accompanied by:

```text
available = false
```

---

# 9. Common Confidence Contract

All published confidence values shall use:

```text
0 to 100
```

Required rules:

- Confidence shall be clamped to the valid range.
- Confidence shall describe classification quality.
- Confidence shall not imply guaranteed outcome probability.
- Unavailable modules shall publish unavailable status.
- A numeric zero shall not be presented as valid confidence when unavailable.
- Candidate confidence may change intrabar.
- Confirmed event confidence shall be snapshotted.

Recommended semantic bands:

```text
0        = Invalid or unavailable numeric fallback
1–39     = Weak
40–59    = Developing
60–74    = Moderate
75–89    = Strong
90–100   = Exceptional
```

---

# 10. Common Score Contract

Normalised public scores shall use:

```text
0 to 100
```

Raw internal scores may use any safe numeric range.

Every normalised score shall define:

- Input evidence
- Maximum theoretical contribution
- Normalisation method
- Missing-module handling
- Conflict handling
- Clamp behaviour

---

# 11. Common Event Contract

Each confirmed event shall conceptually publish:

```text
Event
{
    eventId
    sourceModule
    eventType
    direction
    lifecycle
    confidence
    creationBar
    confirmationBar
    sourcePrice
    eventLevel
    invalidationLevel
    sourceObjectId
    correlationGroup
    contextFlags
}
```

Not every field must exist physically in one Pine Script object.

The semantic contract shall remain intact.

---

# 12. Event Immutability

Once an event is confirmed, the following values shall remain immutable:

- Event ID
- Event type
- Direction
- Creation bar
- Confirmation bar
- Confirmation confidence
- Source price
- Event level
- Source object ID
- Historical label location
- Historical alert snapshot

Later state changes shall create new events.

They shall not rewrite the original event.

---

# 13. Current State Versus Historical Event

Every module shall distinguish:

```text
Current State
```

from:

```text
Historical State-Transition Event
```

Example:

A Wyckoff state may currently be:

```text
Phase D
```

while historical events include:

```text
Spring confirmed at bar 1020
Test confirmed at bar 1028
SOS confirmed at bar 1044
Phase D entered at bar 1044
```

The current state may evolve.

Historical transition events shall remain unchanged.

---

# 14. Event Identity Contract

Event identifiers shall be deterministic within one script calculation.

Conceptual format:

```text
Module
+
Event Type
+
Direction
+
Source Object ID
+
Confirmation Bar
```

Example conceptual IDs:

```text
MS_SWING_HIGH_1040
BOS_BULL_1048_43120
LIQ_SSL_SWEEP_1051_42980
VPA_STOPPING_VOLUME_1051
WYC_SPRING_RANGE_12_1051
SIG_LONG_READY_SETUP_44_1060
```

Pine Script implementation may store numeric hashes or composite integers rather than full strings.

---

# 15. Identifier Hierarchy

The implementation shall support distinct identifiers for:

- Swing ID
- Structure Level ID
- Structural Event ID
- Liquidity Level ID
- Liquidity Interaction ID
- VPA Event ID
- Range ID
- Wyckoff Event ID
- Evidence ID
- Institutional State Event ID
- Setup ID
- Signal Event ID
- Alert ID

Identifiers shall not be reused while historical records remain active.

---

# 16. Source Object Contract

An event derived from a persistent object shall retain the source object identifier.

Examples:

- BOS references a Swing or Structure Level ID.
- Liquidity Sweep references a Liquidity Level ID.
- Spring references a Range ID and boundary interaction.
- Signal references a Setup ID.
- Alert references a Signal Event ID or upstream Event ID.

This allows traceability across the pipeline.

---

# 17. Shared Evidence Contract

Evidence shall be treated as a standard downstream transport abstraction.

Conceptual evidence fields:

```text
Evidence
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

---

# 18. Evidence Consumer Contract

The approved evidence consumers are:

- Institutional Activity
- Signal Scoring
- Debug diagnostics
- Evidence summary presentation
- Alert message enrichment

Upstream modules shall not consume Institutional or Signal evidence to alter their original analytical detections.

This prevents circular dependencies.

---

# 19. Evidence Direction

Evidence direction values:

```text
-1 = Bearish
 0 = Non-Directional
+1 = Bullish
```

Non-directional evidence shall not become directional merely because price closed up or down.

Example:

```text
High RVOL
```

remains non-directional until interpreted with approved contextual evidence.

---

# 20. Evidence Category Flags

Evidence may belong to multiple categories.

Approved conceptual categories:

```text
Directional
Structure
Participation
Liquidity
Behaviour
Sequence
Institutional
Accumulation
Distribution
Continuation
Reversal
Absorption
Expansion
Defence
Exhaustion
Trigger
Risk
Conflict
```

Implementation should use bit flags or equivalent compact representation where practical.

---

# 21. Evidence Lifecycle Contract

Approved evidence lifecycle values:

```text
Unavailable
Candidate
Confirmed
Reinforced
Weakening
Failed
Expired
```

Rules:

- Candidate evidence receives reduced or zero production contribution.
- Confirmed evidence may contribute fully.
- Reinforced evidence may receive a bounded multiplier.
- Weakening evidence receives reduced contribution.
- Failed evidence contributes zero or approved opposing evidence.
- Expired evidence contributes zero.

---

# 22. Evidence Persistence Classes

Approved persistence classes:

```text
Bar
Short
Medium
Structural
Range
State
Persistent
```

Suggested interpretation:

| Class | Typical Duration |
|---|---|
| Bar | Current bar only |
| Short | Several bars |
| Medium | Event lookback window |
| Structural | Until structure supersedes or invalidates |
| Range | While range remains active |
| State | While state remains active |
| Persistent | Until explicit invalidation |

Each event type shall map to one persistence class.

---

# 23. Evidence Correlation Contract

Correlated evidence shall be grouped by underlying market interaction.

Approved initial groups:

```text
Bullish Reversal
Bearish Reversal
Bullish Continuation
Bearish Continuation
Bullish Accumulation
Bearish Distribution
Bullish Expansion
Bearish Expansion
Bullish Defence
Bearish Defence
```

Correlation control shall occur in:

- Institutional Activity
- Signal Scoring

Upstream modules shall publish evidence independently.

---

# 24. Evidence Duplication Contract

Evidence is duplicate when it represents the same:

- Source module
- Source event
- Direction
- Confirmation bar
- Source object
- Lifecycle transition

Persistent evidence shall not be inserted again on every bar.

It shall remain active through lifecycle updates.

---

# 25. Dependency Contract

Dependencies shall be classified as:

```text
None
Optional
Soft
Hard
```

## Hard Dependency

The dependent module becomes Unavailable when the dependency is unavailable.

## Soft Dependency

The module continues with reduced capability or confidence.

## Optional Dependency

The module operates normally but omits optional enhancement.

---

# 26. Approved Dependency Matrix

| Module | Dependency | Type |
|---|---|---|
| Trend | Framework | Hard |
| RVOL | Framework | Hard |
| Market Structure | Framework | Hard |
| BOS/CHOCH | Market Structure | Hard |
| Liquidity | Market Structure | Hard |
| Liquidity | BOS/CHOCH | Hard |
| VPA | RVOL | Soft |
| VPA | Framework price data | Hard |
| Wyckoff | Market Structure | Hard |
| Wyckoff | Liquidity | Soft |
| Wyckoff | VPA | Soft |
| Wyckoff | RVOL | Soft |
| Institutional | Trend | Optional |
| Institutional | Market Structure | Soft |
| Institutional | BOS/CHOCH | Soft |
| Institutional | Liquidity | Soft |
| Institutional | VPA | Soft |
| Institutional | Wyckoff | Soft |
| Scoring | Market Structure | Soft |
| Scoring | BOS/CHOCH | Soft |
| Scoring | Liquidity | Optional |
| Scoring | VPA | Optional |
| Scoring | Wyckoff | Optional |
| Scoring | Institutional | Optional |
| Dashboard | Published outputs | Soft |
| Alerts | Published events | Soft |

This matrix is the initial implementation contract and may be amended only through an ADR.

---

# 27. Hard Dependency Failure

When a hard dependency fails:

```text
enabled = true
available = false
state = Unavailable
confidence = unavailable
events = none
```

A configuration or dependency warning shall be published.

---

# 28. Soft Dependency Failure

When a soft dependency fails:

- The module remains available if its minimum evidence basis remains valid.
- Missing contribution is omitted.
- Confidence may be reduced.
- Missing dependency shall be exposed in diagnostics.
- No artificial negative evidence shall be created.

---

# 29. Settings Contract

Every module shall consume resolved effective settings.

Modules shall not:

- Interpret profile names independently.
- Apply conflicting global defaults.
- Revalidate unrelated modules.
- Modify user inputs.
- Silently enable disabled dependencies.

Settings resolution shall occur before analytical processing.

---

# 30. Effective Settings Contract

Each module shall receive:

```text
effectiveEnabled
effectiveConfirmationMode
effectiveThresholds
effectiveHistoryLimits
effectiveVisualSettings
effectiveAlertSettings
dependencyAvailability
configurationWarnings
```

Module-specific settings may use a dedicated user-defined type or grouped variables.

---

# 31. Configuration Warning Contract

Warnings shall publish:

```text
warningId
sourceModule
severity
messageCode
active
```

Approved severity levels:

```text
Information
Degraded
Invalid
Critical
```

Warnings shall not be treated as market events.

They shall not affect scores except through the actual unavailable or degraded conditions they represent.

---

# 32. Framework Context Contract

The Framework shall publish shared bar and instrument context.

Required values:

- Current bar index
- Confirmed-bar status
- Real-time status
- Historical status
- Symbol
- Timeframe
- Tick size
- Price precision
- Volume availability
- ATR value
- Minimum-history status
- Theme colours
- Global object budget
- Debug mode
- Current schema version

---

# 33. Shared ATR Contract

A single Framework-owned ATR calculation should be used for shared normalisation.

Suggested default:

```text
ATR Length = 14
```

Modules may use specialised volatility calculations only when required by their specification.

Shared ATR shall support:

- Tolerance normalisation
- Range-width normalisation
- Sweep distance
- Entry proximity
- Extension filters
- Risk location
- Gap distortion

---

# 34. Volume Availability Contract

The Framework or RVOL Engine shall publish:

```text
volumeAvailable
volumeUsable
volumeQuality
```

Suggested semantics:

```text
volumeAvailable = volume series exists
volumeUsable = volume is valid for current calculation
volumeQuality = normal, degraded, unavailable
```

Missing volume shall not automatically make the entire indicator unavailable.

---

# 35. Market Structure Publication Contract

Market Structure shall publish at minimum:

- Enabled
- Available
- Current structural direction
- Last confirmed swing high
- Last confirmed swing low
- Last higher high
- Last higher low
- Last lower high
- Last lower low
- Current bullish protection level
- Current bearish protection level
- Equal High context
- Equal Low context
- Current Swing ID values
- New swing event flags
- Structure confidence
- Sufficient-history status

---

# 36. BOS and CHOCH Publication Contract

BOS and CHOCH shall publish at minimum:

- Enabled
- Available
- Current structural event
- Event direction
- Event lifecycle
- Event confidence
- Broken level
- Source Swing ID
- Structural Event ID
- Acceptance status
- Failure status
- Confirmation bar
- New-event flag
- Current structural regime
- Sufficient-history status

Only one primary structural event shall be published per confirmed bar.

Secondary metadata may describe compatible events.

---

# 37. Liquidity Publication Contract

Liquidity shall publish at minimum:

- Enabled
- Available
- Primary liquidity event
- Direction
- Lifecycle
- Confidence
- Liquidity Level ID
- Liquidity Interaction ID
- Level price
- Level type
- Level priority
- Sweep status
- Acceptance status
- Trap status
- Cleared status
- New-event flag
- Confirmation bar
- Sufficient-history status

Only one primary liquidity event shall be published per confirmed bar.

---

# 38. VPA Publication Contract

VPA shall publish at minimum:

- Enabled
- Available
- Primary VPA classification
- Direction
- Lifecycle
- Confidence
- Spread classification
- Volume classification
- Close-location classification
- Effort-versus-result state
- Event ID
- New-event flag
- Confirmation bar
- Volume availability status
- Sufficient-history status

Only one primary VPA classification shall be published per confirmed bar.

---

# 39. Wyckoff Publication Contract

Wyckoff shall publish at minimum:

- Enabled
- Available
- Current Range ID
- Range active status
- Range high
- Range low
- Range midpoint
- Current schematic
- Current phase
- Primary Wyckoff event
- Event direction
- Event lifecycle
- Range confidence
- Phase confidence
- Event confidence
- Sequence validity
- New-event flag
- Confirmation bar
- Invalidation status
- Sufficient-history status

Only one primary Wyckoff event shall be published per confirmed bar.

---

# 40. Institutional Publication Contract

Institutional Activity shall publish at minimum:

- Enabled
- Available
- Primary institutional state
- Direction
- Confidence
- Bullish score
- Bearish score
- Accumulation score
- Distribution score
- Bullish continuation score
- Bearish continuation score
- Bullish reversal score
- Bearish reversal score
- Bullish absorption score
- Bearish absorption score
- Conflict score
- Confirming-module count
- Strongest supporting evidence
- Strongest opposing evidence
- State Event ID
- New-state-event flag
- State-change bar
- Sufficient-history status

---

# 41. Signal Scoring Publication Contract

Signal Scoring shall publish at minimum:

- Enabled
- Available
- Signal state
- Signal direction
- Signal confidence
- Bullish composite score
- Bearish composite score
- Bullish continuation score
- Bearish continuation score
- Bullish reversal score
- Bearish reversal score
- Setup Quality
- Entry Readiness
- Conflict score
- Directional dominance
- Setup type
- Setup ID
- Setup lifecycle
- Setup creation bar
- Setup qualification bar
- Setup ready bar
- Setup trigger bar
- Trigger level
- Invalidation level
- Setup age
- No-Trade reason
- Strongest supporting evidence
- Strongest opposing evidence
- New-state-event flag
- Sufficient-history status

---

# 42. Primary Event Per Bar Contract

The following modules shall publish at most one primary event per confirmed bar:

- BOS and CHOCH
- Liquidity
- VPA
- Wyckoff
- Institutional Activity
- Signal Scoring

This simplifies:

- Dashboard display
- Alerts
- Historical testing
- Event priority
- Explainability

Secondary compatible evidence may remain available as metadata.

---

# 43. Event Priority Ownership

Each module owns priority resolution among its own event types.

Examples:

- VPA resolves one primary VPA event.
- Wyckoff resolves one primary Wyckoff event.
- Signal Scoring resolves one primary signal state.

The Alerts module owns priority resolution across modules.

---

# 44. Same-Bar Structural Pipeline

When one confirmed bar produces multiple structural interactions, processing shall occur as follows:

```text
1. Confirm pivots
2. Update structure
3. Detect BOS or CHOCH
4. Resolve structural acceptance or failure
5. Interpret liquidity interaction
6. Classify VPA
7. Update Wyckoff sequence
8. Aggregate institutional evidence
9. Score signal
10. Publish state events
11. Route alert
```

No downstream event shall be backdated to an earlier bar.

---

# 45. Candidate and Confirmed Separation

Candidate values shall use separate variables from confirmed values.

Example:

```text
candidateSpring
confirmedSpring
```

or:

```text
springLifecycle = Candidate
springLifecycle = Confirmed
```

Confirmed historical values shall never be overwritten by a later candidate calculation.

---

# 46. Hysteresis Contract

Persistent state modules shall implement bounded hysteresis.

Applicable modules:

- Trend
- Wyckoff phase or schematic where appropriate
- Institutional Activity
- Signal Scoring

Hysteresis may use:

- Entry threshold
- Exit threshold
- Confirmation bars
- Dominance margin
- Opposing-event requirement
- Explicit invalidation

Hysteresis shall not suppress hard invalidation events.

---

# 47. State Transition Contract

A state transition event shall publish only when:

```text
newState != previousConfirmedState
```

and all confirmation requirements are satisfied.

Required state-transition metadata:

- Previous state
- New state
- Transition bar
- Confidence
- Source evidence
- State Event ID

---

# 48. Historical Storage Contract

All persistent collections shall be bounded.

Required bounded collections may include:

- Swing history
- Structural events
- Liquidity levels
- Liquidity interactions
- VPA events
- Wyckoff ranges
- Wyckoff events
- Evidence records
- Institutional transitions
- Signal setups
- Alert records
- Visual object references

No unbounded array growth is permitted.

---

# 49. Cleanup Contract

Cleanup shall occur after current-bar analytical and publication processing.

Deletion priority:

```text
1. Expired debug objects
2. Expired candidate objects
3. Old informational visuals
4. Completed setup visuals
5. Old confirmed event visuals
6. Active analytical objects only as a last resort
```

The Dashboard table shall be preserved.

---

# 50. Object Ownership Contract

Each visual object shall have one owning module.

Examples:

- Swing labels → Market Structure
- BOS labels → BOS/CHOCH
- Liquidity lines → Liquidity
- VPA labels → VPA
- Range boxes → Wyckoff
- Institutional markers → Institutional
- Signal markers → Signal Scoring
- Dashboard table → Dashboard

No module shall delete another module’s objects directly.

The Framework may coordinate global budget pressure through published cleanup instructions.

---

# 51. Dashboard Contract

The Dashboard shall:

- Consume published states.
- Perform no market analysis.
- Use one persistent table.
- Reuse table cells.
- Display N/A for unavailable states.
- Distinguish Neutral from Unavailable.
- Display only resolved confirmed production states unless preview mode is enabled.
- Display at most three evidence summary items.
- Avoid stale values when a module becomes unavailable.

---

# 52. Alert Contract

The Alerts module shall:

- Consume confirmed event channels.
- Perform no event detection.
- Apply central eligibility rules.
- Suppress duplicates.
- Apply cooldown.
- Resolve same-bar priority.
- Snapshot event data.
- Publish at most the configured alert count.
- Avoid repeated alerts for unchanged persistent states.

Alert processing shall occur after Signal Scoring and Dashboard state resolution.

---

# 53. No Circular Dependency Contract

The following circular dependencies are prohibited:

- Dashboard influencing analytics
- Alerts influencing analytics
- Signal Scoring changing Institutional classification
- Institutional Activity changing Wyckoff classification
- Wyckoff changing Market Structure history
- Liquidity changing BOS confirmation history
- VPA changing RVOL calculation
- Settings being mutated by analytical modules

Downstream interpretation may not rewrite upstream observation.

---

# 54. Missing-Data Contract

Missing data shall produce one of:

```text
Omitted optional contribution
Reduced confidence
Unavailable module
Configuration warning
```

Missing data shall not produce:

- Fabricated evidence
- Automatic opposing evidence
- Misleading zero state
- Runtime error

---

# 55. Gap Contract

Modules shall expose gap distortion where relevant.

A gap may affect:

- BOS acceptance
- Liquidity sweep interpretation
- VPA spread interpretation
- Wyckoff event confidence
- Signal Entry Readiness
- Risk location
- Alerts

A gap alone shall not confirm a Sweep, Spring, Upthrust, or Triggered signal without approved acceptance logic.

---

# 56. Zero-Range Contract

For bars where:

```text
high == low
```

modules shall:

- Avoid division by zero.
- Mark spread-derived metrics unavailable.
- Avoid new VPA classification requiring valid spread.
- Preserve existing persistent state where valid.
- Avoid false trigger publication.

---

# 57. Score Conflict Contract

Institutional and Signal Scoring shall retain:

- Bullish score
- Bearish score
- Score difference
- Conflict score

They shall not publish only a net score.

This preserves contradictory information.

---

# 58. Normalisation Contract

Enabled-module normalisation is the preferred initial approach.

Required safeguards:

- Disabled modules are excluded from the maximum basis.
- Unavailable soft modules are excluded from available capacity.
- Minimum module diversity still applies.
- One available module cannot produce misleading maximum confidence.
- Contribution caps remain enforced before normalisation.

---

# 59. Institutional Correlation Contract

Institutional Activity shall aggregate upstream evidence.

Signal Scoring may consume both:

- Direct upstream evidence
- Institutional consensus

To prevent double counting:

- Institutional contribution shall be capped.
- Institutional contribution shall act as a consensus bonus.
- Institutional contribution shall be discounted when direct evidence from the same correlation cluster is already dominant.

---

# 60. Signal Setup Contract

The initial implementation shall support:

```text
One active bullish setup
One active bearish setup
```

Signal Scoring shall resolve them into one primary published signal state.

Each setup shall have:

- Setup ID
- Direction
- Setup type
- Creation bar
- Lifecycle
- Quality
- Readiness
- Confidence
- Trigger level
- Invalidation level
- Age
- Primary evidence

---

# 61. Setup Lifecycle Contract

Approved lifecycle:

```text
Watching
Qualified
Ready
Triggered
Invalidated
Expired
Completed
```

Required transition rules:

```text
Watching → Qualified
Qualified → Ready
Ready → Triggered
Watching → Expired
Qualified → Expired
Ready → Invalidated
Triggered → Invalidated or Completed
```

A setup shall not return from Invalidated or Completed to an active state.

A new Setup ID shall be created instead.

---

# 62. Same-Bar Ready and Triggered Contract

Initial implementation decision:

```text
Same-bar Ready and Triggered is allowed.
```

Conditions:

- All Ready requirements are satisfied.
- The approved trigger confirms on the same bar.
- The Setup ID is valid.
- No hard invalidation occurs.
- Trigger has not already been consumed.

Publication policy:

- Primary signal state = Triggered.
- Ready metadata shall be retained.
- Alerts shall publish Triggered only by default.
- Debug mode may expose same-bar Ready transition.

---

# 63. Direct Direction Reversal Contract

A direct transition from:

```text
Long Ready
```

to:

```text
Short Ready
```

shall not occur under normal soft evidence changes.

Required default path:

```text
Long state
→ Invalidated, Conflict, No Trade, or Neutral
→ Bearish Qualified
→ Short Ready
```

A direct same-bar reversal may occur only when:

- Current setup hard-invalidates.
- Strong opposing structural event confirms.
- Opposing score exceeds the reversal margin.
- Opposing setup has a new Setup ID.

---

# 64. Trigger Consumption Contract

Each Setup ID may publish:

```text
one primary Triggered event
```

Repeated same-direction trigger events require:

- New Setup ID
- New structure
- New liquidity interaction
- Prior setup completion or invalidation

---

# 65. No-Trade Contract

No Trade shall include a primary reason.

Approved initial reasons:

```text
Insufficient Score
High Conflict
Weak Structure
Mid Range
Extended Price
Opposing Liquidity
Missing Trigger
Missing Required Module
Signal Too Old
Duplicate Signal
Gap Distortion
Poor Risk Location
Insufficient History
State Instability
```

Only one primary No-Trade reason shall be published.

Secondary reasons may remain available in debug metadata.

---

# 66. Alert Same-Bar Priority Contract

Initial production priority:

```text
1. Signal Invalidated
2. Opposing Triggered signal
3. Long or Short Triggered
4. Long or Short Ready
5. Markup or Markdown Confirmation
6. Spring or Upthrust
7. SOS or SOW
8. Qualified signal
9. Institutional state transition
10. BOS or CHOCH
11. Liquidity Sweep or Trap
12. VPA event
13. Trend transition
14. RVOL event
15. Diagnostic event
```

Default same-bar policy:

```text
Highest Priority Only
```

Supporting compatible events may be included in the message.

---

# 67. Dashboard Status Priority Contract

The Dashboard shall display:

1. Signal state
2. Setup type
3. Signal confidence
4. Institutional state
5. Wyckoff state or event
6. Liquidity event
7. Structural event
8. Trend
9. RVOL and VPA context
10. Configuration warning

Expanded and Debug modes may show additional detail.

---

# 68. Enumeration Governance

All shared enumerations shall be defined centrally.

Required central enumerations:

- Direction
- Availability
- Lifecycle
- Module ID
- Dependency type
- Warning severity
- Evidence category
- Evidence source
- Persistence class
- Signal state
- Setup type
- Setup lifecycle
- No-Trade reason
- Alert severity
- Alert rejection reason

Enumeration numeric values shall remain stable after implementation begins.

---

# 69. String Conversion Contract

User-facing strings shall be created through shared conversion functions.

Examples:

```text
directionToString()
signalStateToString()
wyckoffEventToString()
warningToString()
```

Analytical logic shall use enums.

It shall not repeatedly compare user-facing strings.

---

# 70. Naming Contract

Recommended implementation prefixes:

```text
fw_      Framework
cfg_     Effective settings
tr_      Trend
rv_      RVOL
ms_      Market Structure
bc_      BOS and CHOCH
lq_      Liquidity
vp_      VPA
wy_      Wyckoff
in_      Institutional
sc_      Signal Scoring
db_      Dashboard
al_      Alerts
dbg_     Debug
```

Final naming shall remain consistent with `CODING_STANDARD.md`.

---

# 71. Function Contract

Functions shall:

- Have one primary responsibility.
- Avoid hidden state mutation where practical.
- Use descriptive names.
- Return deterministic values.
- Avoid repeated expensive work.
- Accept resolved settings rather than raw input strings.
- Avoid constructing display strings inside analytical functions.
- Avoid accessing unrelated module state.

---

# 72. Update Function Pattern

Each module should follow a consistent pattern:

```text
validate
→ update persistent state
→ detect candidate
→ confirm event
→ resolve primary state
→ publish outputs
→ register visual intent
```

Suggested conceptual function:

```text
moduleUpdate(context, settings, upstreamOutputs)
```

Pine Script implementation may use grouped functions rather than one large function.

---

# 73. Module Output Snapshot Contract

Each module should publish one consolidated current-bar output snapshot.

Conceptual example:

```text
ModuleOutput
{
    enabled
    available
    state
    direction
    lifecycle
    confidence
    eventId
    newEvent
    confirmationBar
}
```

Module-specific metadata shall extend this base contract.

---

# 74. Error-Safety Contract

No module shall fail because of:

- Empty array
- Invalid array index
- Deleted object reference
- Division by zero
- Missing volume
- Insufficient history
- Disabled dependency
- Invalid level
- `na` source value
- Object limit pressure
- Symbol change
- Timeframe change

Every array access and deletion path shall be guarded.

---

# 75. Performance Contract

Implementation shall:

- Avoid full-history rescanning.
- Use persistent incremental state.
- Use bounded arrays.
- Use one Dashboard table.
- Avoid unnecessary string construction.
- Avoid object creation when visuals are disabled.
- Process only active objects and new events.
- Use cached shared calculations.
- Apply cleanup incrementally.
- Avoid deeply nested loops.

---

# 76. Initial Hard Limits

Provisional implementation limits:

```text
Swing records: 100
Structural events: 100
Liquidity levels: 75
Liquidity interactions: 100
VPA events: 75
Wyckoff ranges: 20
Wyckoff events: 100
Active evidence records: 75
Institutional transitions: 50
Signal setup history: 50
Alert records: 50
Visible signal labels: 25
Visible Wyckoff labels: 25
```

These limits shall be reviewed during performance testing.

---

# 77. Debug Contract

Debug mode may expose:

- Current enum values
- Current IDs
- Candidate states
- Confirmed states
- Active array sizes
- Dependency status
- Configuration warnings
- Correlation groups
- Effective weights
- Score components
- Lifecycle transitions
- Alert rejection reasons
- Object counts

Debug mode shall not alter production calculations.

---

# 78. Testability Contract

Every module shall expose enough deterministic state to test:

- Availability
- State
- Event
- Direction
- Confidence
- Event ID
- Confirmation bar
- Source object ID
- Lifecycle
- Key levels
- New-event flag

Testing shall not rely solely on visual inspection.

---

# 79. Regression Snapshot Contract

Regression tests should record:

- Symbol
- Timeframe
- Settings profile
- Bar timestamp
- Module state
- Event IDs
- Score values
- Signal state
- Setup ID
- Alert eligibility

A future value change shall require:

- Expected calibration change
- Bug fix explanation
- Specification change
- Regression review

---

# 80. Implementation Phase Order

Implementation shall proceed in the following order:

```text
Phase 1 — Shared types, enums, settings, framework
Phase 2 — Trend and RVOL
Phase 3 — Market Structure
Phase 4 — BOS and CHOCH
Phase 5 — Liquidity
Phase 6 — VPA
Phase 7 — Wyckoff
Phase 8 — Shared Evidence
Phase 9 — Institutional Activity
Phase 10 — Signal Scoring
Phase 11 — Dashboard
Phase 12 — Alerts
Phase 13 — Integration testing
Phase 14 — Performance optimisation
Phase 15 — Release validation
```

A downstream phase shall not begin until required upstream contracts compile and pass their module tests.

---

# 81. Definition of Module Completion

A module is implementation-complete only when:

- Specification behaviour is implemented.
- Shared contracts are satisfied.
- Pine Script compiles.
- Outputs are published.
- Dependencies are handled.
- Disabled and unavailable states work.
- Candidate and confirmed states are separated.
- Historical event stability is verified.
- Arrays remain bounded.
- Visual objects remain bounded.
- Functional tests pass.
- Integration tests pass.
- Debug outputs are available.
- Documentation is updated.

---

# 82. Consolidated Decisions

The following initial implementation decisions are adopted:

1. Confirmed-bar production behaviour is the default.
2. Same-bar downstream event consumption is allowed.
3. One primary event per module per confirmed bar.
4. Historical confirmed events are immutable.
5. Current state is separate from historical event history.
6. Direction uses `-1`, `0`, and `+1`.
7. Confidence and public scores use `0–100`.
8. Evidence uses shared semantic fields.
9. Evidence duplication is controlled downstream.
10. Enabled-module normalisation is the initial scoring method.
11. Institutional output acts as a capped consensus contribution.
12. One bullish and one bearish setup may coexist internally.
13. One primary signal state is published.
14. Same-bar Ready and Triggered is allowed.
15. Triggered has alert priority over Ready.
16. Persistent states alert on transition only by default.
17. Dashboard is presentation-only.
18. Alerts are publication-only.
19. Modules consume resolved effective settings.
20. No unbounded arrays or visual collections are permitted.

---

# 83. Remaining Architecture Decisions

The following decisions remain open before implementation of affected modules:

1. Exact Pine Script representation of shared evidence.
2. Exact numeric event ID encoding.
3. Exact bit-flag implementation for categories and context.
4. Final module contribution caps.
5. Final evidence weights.
6. Final normalisation formulas.
7. Final confidence formulas.
8. Final decay formulas.
9. Final state hysteresis thresholds.
10. Final range hierarchy and nested-range support.
11. Final institutional correlation discount.
12. Final score conflict formula.
13. Final object budgets.
14. Final static versus dynamic alert strategy.
15. Final webhook schema.
16. Final public input count.
17. Final profile values.
18. Final configuration-warning display.
19. Final higher-timeframe architecture.
20. Final Pine Script version and supported language features.

Open decisions shall be resolved before implementation reaches the affected module.

---

# 84. Acceptance Criteria

Architecture consolidation is complete when:

- Every module has a defined processing position.
- Every module has a defined availability contract.
- Every shared enum is identified.
- Every module has a publication contract.
- Event identity rules are defined.
- Event immutability rules are defined.
- Evidence semantics are defined.
- Dependencies are classified.
- Settings resolution is centralised.
- Same-bar processing rules are defined.
- Setup lifecycle is defined.
- Alert priority is defined.
- Dashboard priority is defined.
- Object ownership is defined.
- Bounded storage is required.
- Performance rules are defined.
- Testing outputs are defined.
- Remaining decisions are explicitly listed.
- Implementation can begin without inventing integration behaviour ad hoc.k{
  "default": false,
  "MD009": true,
  "MD012": true,
  "MD047": true
}
