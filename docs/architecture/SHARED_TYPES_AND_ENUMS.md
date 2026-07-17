# Shared Types and Enumerations

---

## Document Information

**Document ID:** ARCH-202

**Version:** 0.1 Draft

**Status:** Draft

**Purpose:** Define all shared enums, identifiers, record contracts and Pine representation strategy.

This document is authoritative for every shared type used by the project.

---

# 1. Purpose

Every module shall communicate using a common vocabulary.

This document standardises:

- Enumerations
- Identifier strategy
- Record contracts
- Lifecycle semantics
- Direction values
- Module identifiers
- Event identifiers
- Evidence categories
- Confidence representation
- Shared constants

No module may invent its own versions of these shared concepts.

---

# 2. Pine Script Design Constraints

Pine Script does not provide:

- Traditional enums
- Struct inheritance
- Interfaces
- Namespaces
- Dynamic objects

Therefore the project shall implement shared types using:

- `const int`
- `const string`
- User-defined types (`type`) where beneficial
- Parallel arrays where performance requires
- Helper conversion functions

---

# 3. Enumeration Rules

All enums shall satisfy:

- Stable numeric values
- Never reused
- Never reordered after implementation
- New values appended only
- User-facing strings separated from numeric values

Example:

```text
STATE_LONG_READY = 7
```

not

```text
"Long Ready"
```

inside analytical logic.

---

# 4. Direction Enumeration

```text
DIR_BEARISH = -1
DIR_NEUTRAL = 0
DIR_BULLISH = 1
```

Used by:

- Trend
- Structure
- BOS
- Liquidity
- VPA
- Wyckoff
- Institutional
- Scoring
- Alerts

---

# 5. Availability Enumeration

```text
AVAIL_DISABLED = 0
AVAIL_UNAVAILABLE = 1
AVAIL_AVAILABLE = 2
```

Neutral is analytical state.

Unavailable is computational state.

---

# 6. Lifecycle Enumeration

```text
LIFE_DISABLED = 0
LIFE_UNAVAILABLE = 1
LIFE_CANDIDATE = 2
LIFE_CONFIRMED = 3
LIFE_REINFORCED = 4
LIFE_WEAKENING = 5
LIFE_FAILED = 6
LIFE_INVALIDATED = 7
LIFE_EXPIRED = 8
LIFE_COMPLETED = 9
```

Every persistent event uses this lifecycle.

---

# 7. Module Enumeration

```text
MODULE_FRAMEWORK = 0
MODULE_TREND = 1
MODULE_RVOL = 2
MODULE_STRUCTURE = 3
MODULE_BOS = 4
MODULE_LIQUIDITY = 5
MODULE_VPA = 6
MODULE_WYCKOFF = 7
MODULE_INSTITUTIONAL = 8
MODULE_SCORING = 9
MODULE_DASHBOARD = 10
MODULE_ALERTS = 11
```

---

# 8. Dependency Enumeration

```text
DEP_NONE = 0
DEP_OPTIONAL = 1
DEP_SOFT = 2
DEP_HARD = 3
```

---

# 9. Warning Severity

```text
WARN_INFO = 0
WARN_DEGRADED = 1
WARN_INVALID = 2
WARN_CRITICAL = 3
```

---

# 10. Confidence Bands

```text
CONF_INVALID = 0

CONF_WEAK_MIN = 1
CONF_WEAK_MAX = 39

CONF_DEVELOPING_MIN = 40
CONF_DEVELOPING_MAX = 59

CONF_MODERATE_MIN = 60
CONF_MODERATE_MAX = 74

CONF_STRONG_MIN = 75
CONF_STRONG_MAX = 89

CONF_EXCEPTIONAL_MIN = 90
CONF_EXCEPTIONAL_MAX = 100
```

---

# 11. Signal State Enumeration

```text
SIG_UNAVAILABLE = 0
SIG_NEUTRAL = 1
SIG_NO_TRADE = 2

SIG_BULLISH_WATCH = 3
SIG_BEARISH_WATCH = 4

SIG_BULLISH_QUALIFIED = 5
SIG_BEARISH_QUALIFIED = 6

SIG_LONG_READY = 7
SIG_SHORT_READY = 8

SIG_LONG_TRIGGERED = 9
SIG_SHORT_TRIGGERED = 10

SIG_BULLISH_CONFLICT = 11
SIG_BEARISH_CONFLICT = 12

SIG_INVALIDATED = 13
SIG_EXPIRED = 14
SIG_COMPLETED = 15
```

---

# 12. Setup Lifecycle

```text
SETUP_WATCHING = 0
SETUP_QUALIFIED = 1
SETUP_READY = 2
SETUP_TRIGGERED = 3
SETUP_INVALIDATED = 4
SETUP_EXPIRED = 5
SETUP_COMPLETED = 6
```

---

# 13. Setup Type

```text
SETUP_NONE = 0

SETUP_BULL_CONT = 1
SETUP_BEAR_CONT = 2

SETUP_BULL_REV = 3
SETUP_BEAR_REV = 4

SETUP_BULL_BREAK = 5
SETUP_BEAR_BREAK = 6

SETUP_BULL_RETEST = 7
SETUP_BEAR_RETEST = 8
```

---

# 14. No Trade Reasons

```text
NT_NONE = 0
NT_SCORE = 1
NT_CONFLICT = 2
NT_STRUCTURE = 3
NT_MID_RANGE = 4
NT_EXTENSION = 5
NT_LIQUIDITY = 6
NT_TRIGGER = 7
NT_MODULE = 8
NT_AGE = 9
NT_DUPLICATE = 10
NT_GAP = 11
NT_RISK = 12
NT_HISTORY = 13
NT_INSTABILITY = 14
```

---

# 15. Alert Severity

```text
ALERT_INFO = 0
ALERT_WATCH = 1
ALERT_QUALIFIED = 2
ALERT_ACTIONABLE = 3
ALERT_CRITICAL = 4
```

---

# 16. Evidence Categories

Evidence shall use bit flags.

Suggested layout:

```text
CATEGORY_DIRECTION
CATEGORY_STRUCTURE
CATEGORY_VOLUME
CATEGORY_LIQUIDITY
CATEGORY_VPA
CATEGORY_SEQUENCE
CATEGORY_ACCUMULATION
CATEGORY_DISTRIBUTION
CATEGORY_CONTINUATION
CATEGORY_REVERSAL
CATEGORY_ABSORPTION
CATEGORY_EXPANSION
CATEGORY_TRIGGER
CATEGORY_RISK
CATEGORY_CONFLICT
```

Bit flags minimise storage while allowing multiple categories.

---

# 17. Persistence Class

```text
PERSIST_BAR = 0
PERSIST_SHORT = 1
PERSIST_MEDIUM = 2
PERSIST_STRUCTURE = 3
PERSIST_RANGE = 4
PERSIST_STATE = 5
PERSIST_PERMANENT = 6
```

---

# 18. Object Owner

Every visual object stores owner:

```text
OWNER_STRUCTURE
OWNER_BOS
OWNER_LIQUIDITY
OWNER_VPA
OWNER_WYCKOFF
OWNER_INSTITUTIONAL
OWNER_SCORING
OWNER_DASHBOARD
```

Allows cleanup without cross-module deletion.

---

# 19. Identifier Strategy

The project shall avoid string identifiers internally where possible.

Instead:

- Incrementing integers
- Composite integers
- Parallel metadata

Example:

```text
Range ID = 17

Spring references:

Range ID = 17
Event ID = 203
```

rather than:

```
SPRING_RANGE17
```

---

# 20. Event Record

Conceptual record:

```text
Event
{
    id
    module
    type
    direction
    lifecycle
    confidence
    creationBar
    confirmationBar
    sourceLevel
    invalidationLevel
}
```

Modules may extend this record.

---

# 21. Evidence Record

```text
Evidence
{
    id
    sourceModule
    sourceEvent
    direction
    categoryFlags
    lifecycle
    confidence
    baseWeight
    effectiveWeight
    persistence
}
```

---

# 22. Module Output Base Record

Every module publishes:

```text
ModuleOutput
{
    enabled
    available
    state
    direction
    confidence
    lifecycle
    newEvent
}
```

Module-specific outputs extend this.

---

# 23. Shared Conversion Functions

The Framework shall own conversion helpers such as:

```text
directionToString()

lifecycleToString()

signalStateToString()

setupTypeToString()

warningToString()

moduleToString()
```

No module should duplicate these functions.

---

# 24. Future Extension Policy

New enum values:

- Append only.
- Never reuse retired values.
- Document in CHANGELOG.
- Add migration note if user-visible.

---

# 25. Acceptance Criteria

Shared types are complete when:

- Every module references the same enums.
- Numeric values are frozen.
- Record contracts are defined.
- No duplicated enum definitions exist.
- Conversion functions are centralised.
- Future extensions remain backward compatible.
