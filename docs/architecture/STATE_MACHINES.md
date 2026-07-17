# State Machine Architecture

---

## Document Information

**Document ID:** ARCH-204

**Version:** 0.1 Draft

**Status:** Draft

**Purpose:** Define formal state transitions, entry conditions, exit conditions, invalidation, expiry, and historical event behaviour.

**Primary State Machines:**

- Trend
- Structural regime
- Liquidity interactions
- Wyckoff range and phase
- Institutional Activity
- Signal setup lifecycle
- Alert lifecycle

---

# 1. Purpose

This document defines the persistent state machines used by the indicator.

A state machine shall define:

- Valid states
- Permitted transitions
- Entry requirements
- Persistence requirements
- Exit requirements
- Invalidation rules
- Expiry rules
- Transition event publication
- Historical immutability

No persistent module shall transition between states through undocumented logic.

---

# 2. General State-Machine Rules

All persistent state machines shall:

- Separate candidate state from confirmed state.
- Use confirmed-bar transitions by default.
- Publish a transition event only when confirmed state changes.
- Retain previous confirmed state for diagnostics.
- Use hysteresis where appropriate.
- Avoid direct reversal through weak evidence.
- Give hard invalidation priority over persistence.
- Preserve historical transition events.
- Use deterministic transition precedence.

---

# 3. Generic State Transition Pattern

```text
Current Confirmed State
        │
        ▼
Evaluate Hard Invalidation
        │
        ├── Yes → Invalidated / Neutral / Unavailable
        │
        ▼
Evaluate Candidate Transition
        │
        ▼
Apply Threshold and Persistence
        │
        ▼
Confirm New State
        │
        ▼
Publish Transition Event
        │
        ▼
Persist State
```

---

# 4. State Transition Metadata

Every confirmed transition should capture:

```text
Transition
{
    transitionId
    previousState
    newState
    creationBar
    confirmationBar
    confidence
    primaryEvidenceId
    reasonCode
}
```

---

# 5. Trend State Machine

## States

```text
Unavailable
Neutral
Bullish
Bearish
Strong Bullish
Strong Bearish
```

## Primary Transitions

```text
Unavailable → Neutral
Neutral → Bullish
Neutral → Bearish
Bullish → Strong Bullish
Bearish → Strong Bearish
Strong Bullish → Bullish
Strong Bearish → Bearish
Bullish → Neutral
Bearish → Neutral
Bullish → Bearish
Bearish → Bullish
```

## Direct Reversal Rule

Direct Bullish-to-Bearish or Bearish-to-Bullish transitions may occur only when the opposing classification exceeds the approved reversal threshold.

Otherwise:

```text
Bullish → Neutral → Bearish
```

or:

```text
Bearish → Neutral → Bullish
```

## Entry Conditions

Trend entry may consider:

- Fast and slow directional relationship
- Slope
- Price location
- Persistence
- Strength threshold
- Higher-timeframe alignment where enabled

## Exit Conditions

Trend exits when:

- Directional score falls below exit threshold.
- Neutral conditions persist.
- Opposing reversal threshold confirms.
- Required calculation becomes unavailable.

## Hysteresis

Use:

- Higher entry threshold
- Lower exit threshold
- Optional persistence bars
- Reversal margin

---

# 6. Structural Regime State Machine

## States

```text
Unavailable
Undetermined
Bullish Structure
Bearish Structure
Range Structure
```

## Events Affecting State

- Higher High
- Higher Low
- Lower High
- Lower Low
- Bullish BOS
- Bearish BOS
- Bullish CHOCH
- Bearish CHOCH

## Example Transitions

```text
Undetermined → Bullish Structure
Undetermined → Bearish Structure
Bullish Structure → Bearish Structure through confirmed Bearish CHOCH
Bearish Structure → Bullish Structure through confirmed Bullish CHOCH
Bullish Structure → Range Structure
Bearish Structure → Range Structure
Range Structure → Bullish Structure through accepted Bullish BOS
Range Structure → Bearish Structure through accepted Bearish BOS
```

Confirmed pivots remain immutable even when the current structural regime changes.

---

# 7. Structural Break Lifecycle

## States

```text
Candidate
Confirmed
Accepted
Failed
Invalidated
Expired
```

## Flow

```text
Candidate Break
      │
      ├── Close confirmation fails → Expired
      │
      ▼
Confirmed Break
      │
      ├── Acceptance succeeds → Accepted
      │
      ├── Price rejects within failure window → Failed
      │
      └── Source structure invalid → Invalidated
```

A confirmed break event shall not be rewritten as Failed.

Failure shall be a separate lifecycle event referencing the original break ID.

---

# 8. Liquidity Interaction State Machine

## States

```text
Active Level
Approached
Probed
Swept
Accepted Break
Trap Confirmed
Cleared
Invalidated
Expired
```

## Flow

```text
Active Level
    │
    ▼
Approached
    │
    ▼
Probed
    ├── Rejection and return → Swept
    ├── Accepted continuation → Accepted Break
    └── No resolution → Active Level or Expired
```

After a Sweep:

```text
Swept
   ├── Opposing follow-through → Trap Confirmed
   ├── Later acceptance through level → Cleared
   └── Context invalidation → Invalidated
```

A Liquidity Level ID remains stable through all interactions.

Each interaction receives a distinct Liquidity Interaction ID.

---

# 9. VPA Event Lifecycle

Most VPA events are event classifications rather than long-lived states.

## Lifecycle

```text
Candidate
Confirmed
Reinforced
Failed
Expired
```

## Rules

- One primary VPA classification per confirmed bar.
- Candidate classifications may change intrabar.
- Confirmed classification is immutable.
- Reinforcement is a later separate event.
- A failed expectation creates a new failure event rather than rewriting the original classification.

---

# 10. Wyckoff Range State Machine

## States

```text
No Range
Range Candidate
Range Confirmed
Range Mature
Range Breaking
Range Resolved
Range Invalidated
Range Expired
```

## Flow

```text
No Range
   │
   ▼
Range Candidate
   ├── Insufficient confirmation → No Range
   ▼
Range Confirmed
   ▼
Range Mature
   ├── Boundary interaction → Range Breaking
   ├── Structural invalidation → Range Invalidated
   └── Excessive age → Range Expired
   ▼
Range Breaking
   ├── Accepted breakout → Range Resolved
   ├── Failed breakout → Range Mature
   └── Hard invalidation → Range Invalidated
```

A resolved or invalidated Range ID shall never become active again.

---

# 11. Wyckoff Schematic State Machine

## States

```text
Unknown
Accumulation Candidate
Accumulation
Reaccumulation Candidate
Reaccumulation
Distribution Candidate
Distribution
Redistribution Candidate
Redistribution
Resolved
Invalidated
```

## Rules

- Schematic classification is probabilistic.
- Candidate state precedes confirmed classification.
- Reaccumulation requires broader bullish context.
- Redistribution requires broader bearish context.
- Opposing evidence reduces confidence before causing reclassification.
- Hard range invalidation terminates the schematic.

---

# 12. Wyckoff Phase State Machine

## States

```text
Phase Unknown
Phase A
Phase B
Phase C
Phase D
Phase E
```

## Typical Accumulation Flow

```text
Unknown → A → B → C → D → E
```

## Typical Distribution Flow

```text
Unknown → A → B → C → D → E
```

The semantic interpretation differs by schematic.

## Transition Constraints

- Phase transitions normally move forward.
- Regression requires explicit failure or reclassification logic.
- Phase C shall not be confirmed solely by elapsed time.
- Phase D requires approved evidence of directional emergence.
- Phase E requires accepted departure from the range.

## Invalid Transitions

The following should not occur without reinitialisation:

```text
Phase E → Phase B
Phase D → Phase A
Resolved → Phase C
```

A new Range ID shall be created instead.

---

# 13. Wyckoff Event Sequence Examples

## Accumulation Sequence

```text
Selling Climax
→ Automatic Rally
→ Secondary Test
→ Spring
→ Test
→ Sign of Strength
→ Last Point of Support
→ Markup
```

## Distribution Sequence

```text
Buying Climax
→ Automatic Reaction
→ Secondary Test
→ Upthrust
→ Test
→ Sign of Weakness
→ Last Point of Supply
→ Markdown
```

Not every event is mandatory.

Sequence confidence depends on:

- Event order
- Range location
- Volume behaviour
- Structural context
- Liquidity behaviour
- Follow-through

---

# 14. Institutional Activity State Machine

## States

```text
Unavailable
Neutral
Bullish Bias
Strong Bullish Bias
Bearish Bias
Strong Bearish Bias
Accumulation Bias
Distribution Bias
Bullish Absorption Bias
Bearish Absorption Bias
Bullish Reversal Evidence
Bearish Reversal Evidence
Markup Confirmation
Markdown Confirmation
Balanced Conflict
```

## Transition Principles

- Evidence score must pass entry threshold.
- Directional dominance must exceed minimum margin.
- Conflict must remain below maximum threshold.
- State transitions may require persistence.
- Strong states use stricter thresholds.
- Hard opposing structural evidence may invalidate a state.
- Missing soft dependencies may reduce confidence without forcing Neutral.

## Example Bullish Flow

```text
Neutral
→ Bullish Reversal Evidence
→ Accumulation Bias
→ Bullish Bias
→ Strong Bullish Bias
→ Markup Confirmation
```

## Example Bearish Flow

```text
Neutral
→ Bearish Reversal Evidence
→ Distribution Bias
→ Bearish Bias
→ Strong Bearish Bias
→ Markdown Confirmation
```

## Conflict Flow

```text
Bullish Bias
→ Balanced Conflict
→ Neutral
→ Bearish Bias
```

Direct reversal requires the approved reversal margin and confirmed opposing evidence.

---

# 15. Signal Setup State Machine

One bullish and one bearish setup may exist internally.

## States

```text
Watching
Qualified
Ready
Triggered
Invalidated
Expired
Completed
```

## Normal Flow

```text
Watching
   │
   ▼
Qualified
   │
   ▼
Ready
   │
   ▼
Triggered
   ├── Success condition → Completed
   └── Invalidation → Invalidated
```

## Alternate Terminal Flows

```text
Watching → Expired
Qualified → Expired
Ready → Expired
Watching → Invalidated
Qualified → Invalidated
Ready → Invalidated
```

A terminal setup shall never return to an active state.

A new setup requires a new Setup ID.

---

# 16. Watching State

## Entry

A setup enters Watching when:

- Directional evidence exists.
- Minimum preliminary score is met.
- A valid setup hypothesis can be assigned.
- No hard invalidation exists.

## Exit

Watching exits to:

- Qualified
- Expired
- Invalidated

---

# 17. Qualified State

## Entry

A setup enters Qualified when:

- Directional score meets qualification threshold.
- Setup Quality meets minimum threshold.
- Minimum evidence diversity is present.
- Conflict remains acceptable.
- Setup remains structurally valid.

## Exit

Qualified exits to:

- Ready
- Expired
- Invalidated

It shall not transition directly to Completed.

---

# 18. Ready State

## Entry

A setup enters Ready when:

- Setup is Qualified.
- Entry Readiness meets threshold.
- Required trigger context exists.
- Trigger level is valid.
- Invalidation level is valid.
- Price is not excessively extended.
- Setup age is within limit.

## Exit

Ready exits to:

- Triggered
- Invalidated
- Expired

---

# 19. Triggered State

## Entry

A setup enters Triggered when:

- Approved trigger confirms.
- Setup ID is active.
- Trigger has not already been consumed.
- No hard invalidation occurs.
- Same-bar triggering rules are satisfied.

## Behaviour

Each Setup ID may publish one primary Triggered event.

## Exit

Triggered exits to:

- Completed
- Invalidated

---

# 20. Same-Bar Ready and Triggered

The initial implementation permits:

```text
Qualified → Ready → Triggered
```

on one confirmed bar.

Required behaviour:

- Final published state is Triggered.
- Ready bar metadata is retained.
- One Triggered alert is published by default.
- Debug mode may expose the intermediate Ready transition.

---

# 21. Signal Invalidation

Hard invalidation has priority over:

- Score persistence
- Setup Quality
- Entry Readiness
- Cooldown
- Signal hysteresis

Possible invalidation causes:

- Structural level failure
- Accepted opposing break
- Trigger level failure
- Range invalidation
- Opposing setup dominance
- Setup context no longer valid

Invalidation reason shall be recorded.

---

# 22. Signal Expiry

Expiry occurs when:

- Maximum setup age is exceeded.
- Required trigger does not occur in time.
- Evidence decays below minimum without structural invalidation.
- Setup context becomes stale.

Expiry is different from invalidation.

---

# 23. Published Signal State Machine

The primary published signal state is resolved from internal bullish and bearish setups.

## States

```text
Unavailable
Neutral
No Trade
Bullish Watch
Bearish Watch
Bullish Qualified
Bearish Qualified
Long Ready
Short Ready
Long Triggered
Short Triggered
Bullish Conflict
Bearish Conflict
Invalidated
Expired
Completed
```

## Resolution Precedence

Suggested precedence:

```text
Invalidated
Triggered
Ready
Conflict
Qualified
Watch
No Trade
Neutral
Unavailable
```

Direction and setup priority must also be applied.

---

# 24. Direct Signal Reversal

Default transition:

```text
Long Ready
→ Invalidated or No Trade
→ Bearish Qualified
→ Short Ready
```

Direct:

```text
Long Ready → Short Ready
```

is prohibited unless:

- Long setup hard-invalidates.
- Opposing structural event confirms.
- Opposing reversal margin is exceeded.
- New bearish Setup ID is created.

---

# 25. Alert Lifecycle State Machine

## States

```text
Candidate
Rejected
Eligible
Published
Suppressed
Expired
```

## Flow

```text
Candidate
   ├── Validation failure → Rejected
   ▼
Eligible
   ├── Duplicate → Suppressed
   ├── Cooldown → Suppressed
   ├── Priority loss → Suppressed
   ▼
Published
```

An externally delivered alert cannot be withdrawn.

A provisional alert and confirmed alert shall use distinct identities.

---

# 26. Dashboard State Behaviour

The Dashboard is not an analytical state machine.

It reflects the latest resolved output snapshots.

Required behaviour:

- Clear stale values when availability changes.
- Show N/A for Unavailable.
- Show Neutral only for valid neutral states.
- Display confirmed values by default.
- Display developing values only in approved preview mode.

---

# 27. Transition Precedence

Where several conditions occur on one bar:

```text
1. Hard dependency failure
2. Hard invalidation
3. Completion
4. Trigger
5. Ready transition
6. Qualification
7. Reinforcement
8. Weakening
9. Expiry
10. Persistence
```

Module specifications may define stricter local precedence.

---

# 28. Historical Stability

Once a state transition is confirmed, the transition record shall retain:

- Transition ID
- Previous state
- New state
- Confirmation bar
- Confidence
- Source IDs
- Relevant levels

Future states shall not move the historical transition to another bar.

---

# 29. Testing Requirements

Every state machine shall test:

- Initialisation
- Normal forward transition
- Persistence
- Weakening
- Reinforcement
- Hard invalidation
- Expiry
- Direct reversal prevention
- Same-bar multiple conditions
- Missing dependency
- Settings recalculation
- Historical stability
- Terminal-state immutability

---

# 30. Acceptance Criteria

State-machine architecture is complete when:

- Every persistent module has explicit states.
- Every permitted transition is documented.
- Candidate and confirmed states are separated.
- Invalid transitions are identified.
- Hard invalidation precedence is defined.
- Expiry differs from invalidation.
- Transition metadata is defined.
- Historical transition events are immutable.
- Same-bar transition behaviour is defined.
- State-machine tests can be written directly from this document.
