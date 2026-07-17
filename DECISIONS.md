---

## ADR-004 — Confirmed Pivot-Based Market Structure

**Date:** 2026-07-17

**Status:** Proposed

### Context

Market Structure requires stable swing references for HH, HL, LH, LL, BOS, CHOCH, and liquidity analysis.

Using unconfirmed developing highs and lows would create unstable historical states and could cause downstream structural events to repaint.

### Decision

The production Market Structure Engine shall use confirmed pivots as its authoritative structural source.

The engine may track provisional pivots internally, but provisional pivots shall not:

- Define confirmed HH, HL, LH, or LL states.
- Trigger confirmed BOS or CHOCH events.
- Trigger confirmed alerts.
- Alter final historical structure before confirmation.

Confirmed pivots may be plotted on their original pivot bars after the required right-side confirmation delay.

### Consequences

**Positive:**

- Stable historical structure
- Clear separation between developing and confirmed information
- Reliable downstream BOS and CHOCH analysis
- Reduced false structural events
- Easier regression testing

**Negative:**

- Structural information is delayed.
- Faster reversals may be recognised later.
- Users must understand that pivot labels are confirmed after the pivot bar.
- Conservative settings may reduce signal frequency.

### Approval

Pending specification review.
---

## ADR-005 — Unified Structural Event Pipeline

**Date:** 2026-07-17

**Status:** Proposed

### Context

Break of Structure (BOS), Change of Character (CHOCH), liquidity sweeps, failed breakouts, and acceptance events all originate from price interacting with confirmed structural levels. Implementing separate detection logic for each event would duplicate calculations, increase maintenance effort, and risk inconsistent classifications.

### Decision

The project shall use a unified structural event pipeline.

Each interaction with a confirmed structural level shall progress through a common sequence:

1. Reference level identified
2. Price approaches the level
3. Wick interaction evaluated
4. Close interaction evaluated
5. Confirmation rules applied
6. Event classified

The resulting classification may include:

- Bullish BOS
- Bearish BOS
- Bullish CHOCH
- Bearish CHOCH
- Liquidity Sweep
- Failed Break
- Acceptance
- Rejection

Contextual modules such as Liquidity, Wyckoff, and Institutional Activity will interpret these classified events rather than re-implementing detection logic.

### Consequences

**Positive:**

- Single source of truth for structural events
- Reduced duplicate logic
- Consistent behaviour across modules
- Easier regression testing
- Extensible architecture for future event types

**Negative:**

- Higher initial implementation complexity
- Requires careful event-state management
- Additional validation needed for edge cases

### Approval

Pending specification review
---

## ADR-006 — Close-Confirmed Structural Events with Immutable History

**Date:** 2026-07-17

**Status:** Proposed

### Context

Wick-only breaches frequently represent liquidity sweeps, stop runs, or temporary volatility rather than genuine structural breaks.

Reclassifying or deleting historical BOS and CHOCH events after later price action would also create unstable historical output.

### Decision

The production default shall require a confirmed close beyond an eligible structural level before publishing BOS or CHOCH.

Wick-only breaches shall remain distinct structural-interaction events.

Once a BOS or CHOCH event is confirmed, its historical record shall remain immutable.

If price subsequently returns through the broken level, the engine shall emit a separate Failed Break event rather than deleting or rewriting the original event.

### Consequences

**Positive:**

- More reliable structural-break classification
- Clear separation between sweeps and confirmed breaks
- Stable historical signals
- Better alert consistency
- Easier regression testing

**Negative:**

- Events occur later than wick-based systems
- Some fast breakouts may receive delayed confirmation
- Failed-break tracking requires additional state
- Users seeking aggressive intrabar signals may consider the default conservative

### Approval

Pending specification review
---

## ADR-007 — Liquidity Events as Contextual Interpretations of Structural Interactions

**Date:** 2026-07-17

**Status:** Proposed

### Context

Liquidity sweeps, accepted breakouts, failed auctions, and traps all arise from price interacting with structural levels.

Creating an independent liquidity pivot system would duplicate Market Structure logic and risk conflicting level definitions.

A wick breach must also remain distinct from a confirmed structural breakout.

### Decision

The Liquidity Engine shall use confirmed levels and events supplied by the Market Structure and BOS/CHOCH engines.

It shall interpret those interactions as:

- Buy-Side Liquidity Sweep
- Sell-Side Liquidity Sweep
- Liquidity Rejection
- Accepted Liquidity Break
- Bull Trap
- Bear Trap
- Cleared Liquidity

The Liquidity Engine shall not create a separate incompatible structural pivot model.

Wick breaches shall not be classified as structural breaks unless the BOS/CHOCH confirmation rules are satisfied.

### Consequences

**Positive:**

- One structural source of truth
- Consistent sweep and breakout classification
- Reduced duplicate calculations
- Clear separation between structural detection and liquidity interpretation
- Easier integration with Wyckoff and Institutional analysis

**Negative:**

- Liquidity analysis depends on confirmed structural pivots
- Some short-lived micro-liquidity events may be detected late
- Level lifecycle management adds implementation complexity
- Session and range liquidity require future source extensions

### Approval

Pending specification review
---

## ADR-008 — Unified Volume Price Analysis Classification Pipeline

**Date:** 2026-07-17

**Status:** Proposed

### Context

Traditional VPA implementations often consist of numerous independent pattern detectors that may produce conflicting or overlapping outputs for the same candle.

Examples include:

- No Demand
- No Supply
- Stopping Volume
- Churn
- Absorption
- Climactic Volume

Running separate detection logic for each increases maintenance complexity and can produce inconsistent classifications.

### Decision

The project shall use a unified VPA classification pipeline.

Each confirmed bar shall be evaluated through a fixed sequence:

1. Price analysis
2. Volume analysis
3. Spread analysis
4. Close-location analysis
5. Wick analysis
6. Context analysis
7. Event classification
8. Confidence calculation

Each analysed bar shall publish at most one primary VPA classification and optional supporting metadata.

Interpretation remains descriptive rather than predictive.

Directional significance shall be determined later by the Signal Scoring Engine using trend, market structure, liquidity, and Wyckoff context.

### Consequences

**Positive:**

- Consistent candle interpretation
- One VPA event per analysed bar
- Easier regression testing
- Reduced duplicate logic
- Clear separation between observation and trading decisions

**Negative:**

- Some composite bars may fit multiple textbook VPA definitions
- Event-priority rules must be carefully defined
- Confidence scoring requires calibration across many market conditions

### Approval

Pending specification review
---

## ADR-009 — Contextual VPA with Single Primary Classification

**Date:** 2026-07-17

**Status:** Proposed

### Context

A single candle may satisfy several traditional Volume Price Analysis definitions simultaneously.

For example, a high-volume narrow-spread bar at a liquidity level may resemble:

- Absorption
- Churn
- Stopping Volume
- Effort Versus Result

Publishing every matching label would create conflicting output, chart clutter, and unreliable downstream scoring.

VPA meaning also depends heavily on trend, structure, liquidity, and structural-event context.

### Decision

The VPA Engine shall evaluate every confirmed eligible bar through one contextual classification pipeline.

Each bar shall publish:

- At most one primary VPA event
- One event direction
- One event-confidence value
- Optional supporting evidence

Candidate events shall be resolved using deterministic priority rules.

The classification shall use:

- Relative Volume
- Spread
- Body
- Close location
- Wick structure
- Trend
- Market Structure
- BOS and CHOCH
- Liquidity context

VPA classifications shall remain descriptive and shall not independently generate trade decisions.

### Consequences

**Positive:**

- Clear chart output
- Deterministic event classification
- Reduced duplicate logic
- Reliable downstream scoring
- Better contextual interpretation
- Easier regression testing

**Negative:**

- Some valid secondary textbook interpretations will not appear as primary labels
- Event-priority rules require careful calibration
- Context dependencies increase testing scope
- Confidence values require validation across markets and timeframes

### Approval

Pending specification review
---

## ADR-010 — Sequence-Based Probabilistic Wyckoff State Engine

**Date:** 2026-07-17

**Status:** Proposed

### Context

Wyckoff events derive meaning from their position within a trading range and from the sequence of preceding and following events.

Isolated detection of Springs, Upthrusts, Signs of Strength, or Last Points of Support can produce misleading classifications when no valid range or schematic context exists.

Real markets also rarely follow a complete textbook schematic, and Phase C may be absent.

### Decision

The project shall implement Wyckoff analysis as a persistent, probabilistic state engine.

The engine shall:

- Detect and maintain trading-range state
- Track ordered Wyckoff events
- Support Phase A through Phase E
- Allow valid phase transitions that omit Phase C
- Classify Accumulation, Distribution, Reaccumulation, and Redistribution using confidence values
- Use confirmed outputs from Trend, Market Structure, BOS/CHOCH, Liquidity, RVOL, and VPA
- Publish candidate and confirmed states
- Preserve confirmed historical events
- Represent ambiguous ranges as Undetermined rather than forcing a schematic

Springs, Upthrusts, SOS, SOW, LPS, and LPSY shall require appropriate range and sequence context.

### Consequences

**Positive:**

- More faithful Wyckoff interpretation
- Reduced false pattern labelling
- Supports incomplete real-market schematics
- Clear separation between candidate and confirmed states
- Stronger downstream Institutional and Scoring inputs
- Stable and testable phase transitions

**Negative:**

- Considerably more state-management complexity
- Longer confirmation delays
- Wider test matrix
- Confidence calibration required across instruments and timeframes
- Some valid discretionary interpretations may remain unclassified

### Approval

Pending specification review
---

## ADR-011 — Institutional Activity as Evidence Aggregation Rather Than Direct Detection

**Date:** 2026-07-17

**Status:** Proposed

### Context

No indicator can directly observe institutional orders or identify the actions of professional participants with certainty.

Traditional "Smart Money" indicators often infer institutional activity from isolated events, producing conflicting or overstated conclusions.

The project already contains specialised engines for:

- Trend
- Relative Volume
- Market Structure
- BOS and CHOCH
- Liquidity
- Volume Price Analysis
- Wyckoff

These engines collectively provide a rich set of observable evidence.

### Decision

The Institutional Activity Engine shall not attempt to detect institutional trading directly.

Instead, it shall aggregate evidence from upstream engines into:

- Bullish evidence
- Bearish evidence
- Accumulation evidence
- Distribution evidence
- Continuation evidence
- Reversal evidence

The engine shall resolve conflicting evidence using deterministic weighting and publish:

- One primary institutional state
- One confidence value
- Supporting evidence metadata

Institutional states shall remain probabilistic interpretations rather than factual assertions.

### Consequences

**Positive:**

- Scientifically defensible architecture
- Fully explainable outputs
- Consistent downstream scoring
- Easy regression testing
- Flexible weighting without changing upstream modules
- Clear separation between observation and interpretation

**Negative:**

- Weight calibration requires extensive testing
- Confidence values depend on multiple upstream engines
- Users seeking simple binary "smart money" signals may perceive the output as more conservative
- Evidence conflicts require carefully defined resolution rules

### Approval

Pending specification review
---

## ADR-012 — Standardised Shared Evidence Model for Downstream Aggregation

**Date:** 2026-07-17

**Status:** Proposed

### Context

The Institutional Activity and Signal Scoring engines require information from multiple upstream analytical modules.

Passing large numbers of unrelated booleans, numeric values, and module-specific enumerations would tightly couple downstream logic to every upstream implementation.

It would also make future analytical modules difficult to integrate.

### Decision

The project shall define a standardised conceptual evidence model for downstream aggregation.

Each qualifying upstream event shall expose compatible evidence metadata including:

- Source module
- Source event type
- Direction
- Evidence category
- Confidence
- Base or suggested weight
- Creation bar
- Confirmation bar
- Price level
- Structural context
- Liquidity context
- Lifecycle state
- Unique source-event identifier

The Pine Script implementation may use user-defined types, parallel arrays, scalar outputs, or bounded queues according to language and performance constraints.

The conceptual semantics shall remain consistent regardless of the implementation representation.

Institutional Activity and Signal Scoring shall consume this shared evidence model rather than duplicating module-specific interpretation logic.

### Consequences

**Positive:**

- Reduced coupling between modules
- Consistent evidence handling
- Easier future module integration
- Centralised duplication control
- More explainable institutional and scoring outputs
- Improved testing and traceability

**Negative:**

- Additional abstraction and metadata
- More bounded state management
- Pine Script implementation may require parallel arrays or simplified records
- Evidence identifiers and lifecycle rules require strict governance

### Approval

Pending specification review
---

## ADR-013 — Separation of Setup Quality, Entry Readiness, and Signal State

**Date:** 2026-07-17

**Status:** Proposed

### Context

A market may present strong directional and structural evidence without offering a timely or sufficiently confirmed entry.

Combining contextual quality and entry timing into one score would create several problems:

- High-quality setups may trigger prematurely.
- Late entries may retain artificially high scores.
- Users cannot distinguish developing opportunities from actionable conditions.
- Alert logic becomes difficult to govern.
- Future strategy execution would lack a clear trigger boundary.

### Decision

The Signal Scoring Engine shall maintain separate calculations for:

- Bullish and Bearish Composite Scores
- Continuation and Reversal Scores
- Setup Quality
- Entry Readiness
- Signal Confidence
- Conflict Score

Setup Quality shall measure the strength of the broader market context.

Entry Readiness shall measure the presence, quality, freshness, and proximity of an approved trigger.

The engine shall resolve these values into a persistent signal lifecycle:

```text
Watch
→ Qualified
→ Ready
→ Triggered
→ Invalidated, Expired, or Completed
```

No Trade shall remain a valid primary state when evidence exists but execution conditions are unsuitable.

Confirmed signal-state transitions shall remain immutable.

### Consequences

**Positive:**

- Clear distinction between analysis and timing
- Fewer premature signals
- Better alert quality
- Stronger explainability
- Easier future strategy integration
- Explicit setup lifecycle
- Conservative late-entry handling

**Negative:**

- Additional scoring dimensions
- More state-management complexity
- More thresholds require calibration
- Users may need dashboard guidance to understand setup versus readiness
- Same-bar trigger policy requires careful testing

### Approval

Pending specification review
---

## ADR-014 — Dashboard as a Pure Presentation Layer

**Date:** 2026-07-17

**Status:** Proposed

### Context

As analytical complexity increases, there is a temptation to duplicate calculations within the Dashboard to simplify display logic.

Doing so would create multiple sources of truth, increase maintenance cost, and risk inconsistencies between displayed values and analytical outputs.

### Decision

The Dashboard shall function exclusively as a presentation layer.

It shall:

- Consume published outputs from analytical modules.
- Render those outputs according to the active layout and theme.
- Perform no analytical calculations.
- Maintain no independent market state.

Every displayed value shall originate from exactly one upstream engine.

### Consequences

**Positive:**

- Single source of truth
- Simpler testing
- Easier maintenance
- Consistent user interface
- Reduced coupling
- Lower computational overhead

**Negative:**

- Dashboard layout depends on stable upstream interfaces
- Presentation flexibility must operate within published outputs
- Additional display-specific metadata may occasionally be required

### Approval

Pending specification review
---

## ADR-015 — Centralised Confirmed-Event Alert Routing

**Date:** 2026-07-17

**Status:** Proposed

### Context

Every analytical module produces events that may be useful to users.

Allowing each module to implement its own alert logic would create inconsistent:

- Confirmation rules
- Confidence thresholds
- Duplicate suppression
- Cooldowns
- Message formats
- Same-bar priority
- State-transition behaviour

It would also make it difficult to prevent multiple alerts describing the same underlying market interaction.

### Decision

The project shall implement one central Alerts module.

Upstream modules shall publish event metadata but shall not independently govern alert delivery.

The Alerts module shall:

- Consume confirmed upstream events.
- Apply module and event enablement.
- Apply confidence and contextual filters.
- Suppress duplicate event identities.
- Apply cooldown and rate limits.
- Resolve same-bar event priority.
- Combine compatible supporting events where configured.
- Format compact, standard, detailed, or webhook messages.
- Preserve immutable alert snapshots.
- Publish state-transition alerts rather than repeated persistent-state alerts by default.

Confirmed-bar publication shall be the production default.

Developing alerts shall remain optional and shall be clearly identified as provisional.

### Consequences

**Positive:**

- Consistent alert behaviour
- Reduced duplicate notifications
- Centralised message governance
- Easier testing
- Stable webhook integration
- Clear state-transition semantics
- Lower coupling between analytics and delivery

**Negative:**

- Alerts depend on stable upstream event interfaces
- Central event prioritisation adds configuration complexity
- Alert identifiers require strict governance
- Dynamic messages may increase Pine Script string-management cost
- Users may need guidance when selecting between static and dynamic alert mechanisms

### Approval

Pending specification review
---

## ADR-016 — Centralised Effective Settings Resolution

**Date:** 2026-07-17

**Status:** Proposed

### Context

The indicator contains multiple analytical, presentation, and alert modules with shared configuration concerns.

If each module independently interprets profiles, validates dependencies, applies fallbacks, and resolves user overrides, configuration behaviour may become inconsistent.

This would create risks including:

- Different modules interpreting the same profile differently
- Invalid threshold combinations
- Hidden dependency failures
- Misleading disabled or unavailable states
- Duplicated settings
- Difficult settings migration
- Unstable user presets
- Increased implementation complexity

### Decision

The project shall implement a central effective-settings resolution stage before analytical processing.

The Settings layer shall:

- Read all user inputs.
- Resolve the selected global profile.
- Apply approved module overrides.
- Validate numeric ranges.
- Validate cross-setting relationships.
- Resolve hard, soft, and optional dependencies.
- Apply safety clamps and deterministic fallbacks.
- Publish effective module settings.
- Publish configuration warnings.
- Distinguish Disabled, Unavailable, Neutral, and Valid states.
- Preserve stable user-facing input names and option strings after Version 1.0.

Modules shall consume resolved effective settings rather than independently interpreting profile names or global configuration.

The precedence order shall be:

```text
Hard Safety Limits
→ Explicit Custom Settings
→ Approved Module Overrides
→ Global Profile Defaults
→ System Fallbacks
```

Confirmed-bar behaviour shall remain the production default.

### Consequences

**Positive:**

- Consistent configuration across modules
- Deterministic profile behaviour
- Safer dependency handling
- Easier testing
- Reduced duplicated validation
- Clearer migration and versioning
- Better user experience
- More reliable disabled and unavailable states

**Negative:**

- Central settings resolution adds architectural complexity
- Effective settings require disciplined naming and interfaces
- Profile changes affect many modules and require regression testing
- Pine Script input controls cannot be programmatically reset
- User presets may still require manual migration after breaking input changes

### Approval

Pending specification review
---

## ADR-018 — Frozen Shared Enumerations and Data Contracts

**Date:** 2026-07-17

**Status:** Proposed

### Context

Large Pine Script projects frequently fail because modules evolve their own local state representations.

This causes incompatible interfaces, duplicated conversion logic, enum drift, and difficult regression testing.

### Decision

The project shall define one shared set of enumerations and conceptual data contracts before implementation begins.

All analytical, presentation, and alert modules shall use these shared definitions.

User-facing strings shall be produced only through shared conversion functions.

Enumeration numeric values shall be treated as stable after implementation begins.

New values shall be appended rather than inserted or reordered.

### Consequences

**Positive**

- Stable interfaces
- Easier regression testing
- Consistent debugging
- Lower maintenance cost
- Simpler module integration
- Better future extensibility

**Negative**

- Enum planning requires discipline
- Later structural changes become more expensive
- Conversion utilities become a shared dependency

### Approval

Pending implementation review
---

## ADR-019 — Explicit Public Module APIs

**Date:** 2026-07-17

**Status:** Proposed

### Context

The module specifications describe analytical responsibilities, but implementation also requires explicit public boundaries.

Without defined APIs, downstream modules may access private state, duplicate calculations, or become dependent on implementation details.

### Decision

Every module shall expose a documented public output contract.

Modules shall:

- Consume only approved upstream outputs.
- Publish stable current-bar snapshots.
- Own their persistent state and visual objects.
- Prevent downstream mutation of private state.
- Expose sufficient metadata for testing.
- Separate analytical outputs from display strings.
- Treat public interface changes as architectural changes.

### Consequences

**Positive:**

- Reduced coupling
- Clear ownership
- Easier testing
- Easier incremental implementation
- Lower risk of duplicated calculations
- Safer refactoring

**Negative:**

- Interfaces require upfront design
- API changes require dependent-module review
- Some Pine Script implementations may require verbose grouped variables

### Approval

Pending implementation review
---

## ADR-020 — Formal Persistent State Machines

**Date:** 2026-07-17

**Status:** Proposed

### Context

Trend, Wyckoff, Institutional Activity, Signal Scoring, and Alerts all maintain persistent state.

Informal transition logic would increase the risk of invalid state jumps, repaint-like behaviour, duplicate events, and inconsistent historical records.

### Decision

Every persistent module shall implement a documented state machine.

State machines shall define:

- Valid states
- Permitted transitions
- Entry criteria
- Exit criteria
- Invalidation
- Expiry
- Transition precedence
- Candidate and confirmed separation
- Historical transition immutability

Terminal states shall not return to active states.

New active sequences shall receive new identifiers.

### Consequences

**Positive:**

- Deterministic lifecycle behaviour
- Easier testing
- Reduced signal flicker
- Clear invalidation semantics
- Stable historical transitions

**Negative:**

- State implementation becomes more explicit and verbose
- Transition changes require architecture review
- Same-bar transitions require careful precedence handling

### Approval

Pending implementation review
---

## ADR-021 — Read-Only Downstream Data Flow and Explicit Ownership

**Date:** 2026-07-17

**Status:** Proposed

### Context

The indicator contains several dependent analytical stages.

Allowing downstream modules to modify upstream records would create circular dependencies, hidden coupling, and unstable historical behaviour.

### Decision

Data shall flow downstream through read-only published contracts.

Each persistent record and visual object shall have one owner.

Only the owning module may mutate or delete its records.

Downstream modules may interpret outputs but shall not rewrite upstream observations.

Dashboard, Alerts, and Debug components shall remain terminal consumers.

### Consequences

**Positive:**

- No circular dependencies
- Stable historical records
- Clear cleanup ownership
- Easier debugging
- Safer module replacement
- Predictable same-bar processing

**Negative:**

- Some information must be duplicated in output snapshots
- Output interfaces may become larger
- Global cleanup requires coordination rather than direct deletion

### Approval

Pending implementation review.
