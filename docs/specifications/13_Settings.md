# Settings Specification

---

## Document Information

**Document ID:** SPEC-113

**Version:** 0.1 Draft

**Status:** Draft

**Module:** Settings and Configuration

**Dependencies:**

- Framework
- Trend Engine
- Relative Volume Engine
- Market Structure Engine
- BOS and CHOCH Engine
- Liquidity Engine
- Volume Price Analysis Engine
- Wyckoff Engine
- Institutional Activity Engine
- Signal Scoring Engine
- Dashboard
- Alerts

**Dependent Modules:**

- All indicator modules
- Future migration and preset-management components

---

# 1. Purpose

The Settings module defines the complete configuration architecture for the indicator.

It shall govern:

- Input organisation
- Input naming
- Default values
- Profiles and presets
- Module enablement
- Module dependencies
- Validation
- Threshold governance
- Visibility controls
- Presentation settings
- Alert settings
- Debug settings
- Version migration
- Backward compatibility
- Safe fallback behaviour

The Settings module shall provide a stable and understandable interface between users and the analytical system.

It shall not perform market analysis.

---

# 2. Objectives

The Settings module shall:

- Organise inputs into logical groups.
- Minimise user confusion.
- Provide safe production defaults.
- Support Conservative, Balanced, Aggressive, and Custom profiles.
- Distinguish user-facing settings from internal constants.
- Centralise shared thresholds.
- Prevent incompatible configurations.
- Resolve dependencies deterministically.
- Validate numeric and selection inputs.
- Handle disabled and unavailable modules safely.
- Avoid duplicate settings across modules.
- Support compact, standard, advanced, and debug configuration levels.
- Maintain stable input names where practical.
- Document the operational impact of each setting.
- Support future migration between versions.
- Preserve indicator performance.
- Avoid exposing excessive calibration complexity by default.

---

# 3. Design Principles

## 3.1 Settings Are Configuration, Not Logic

Settings may control analytical behaviour, but analytical decisions shall remain inside their owning modules.

The Settings module shall not calculate:

- Trend
- Structure
- Liquidity
- VPA
- Wyckoff
- Institutional state
- Signal scores

## 3.2 Safe Defaults

Default settings shall prioritise:

- Confirmed-bar behaviour
- Low repaint risk
- Moderate sensitivity
- Low visual clutter
- Conservative alerting
- Bounded object creation
- Broad market compatibility

## 3.3 Progressive Disclosure

Most users shall not be required to configure every threshold.

The user interface shall expose:

1. Core settings
2. Module settings
3. Display settings
4. Alert settings
5. Advanced settings
6. Debug settings

Complex calibration inputs should remain hidden or grouped under Advanced.

## 3.4 One Setting, One Owner

Every input shall have one owning module.

Shared settings shall be owned centrally where multiple modules depend on them.

Examples:

- RVOL length belongs to RVOL.
- Pivot length belongs to Market Structure.
- Dashboard position belongs to Dashboard.
- Global confirmed-bar mode belongs to Settings or Framework.
- Alert cooldown belongs to Alerts.

## 3.5 Stable User Interface

Input names, group names, and option strings should remain stable after Version 1.0.

Changes shall require:

- Changelog entry
- Migration note
- Version review
- Compatibility assessment

## 3.6 Profiles Shall Be Deterministic

Selecting a profile shall produce a defined configuration outcome.

Profiles shall not vary silently by symbol or timeframe unless explicitly designed as adaptive profiles.

## 3.7 Custom Settings Remain Explicit

When Custom profile is active, user-defined thresholds shall govern.

The system shall not silently overwrite Custom values.

## 3.8 Dependency Safety

Disabling a module shall not cause downstream runtime failure.

Dependent modules shall:

- Omit unavailable evidence.
- Reduce confidence where appropriate.
- Enter Unavailable only when required inputs are essential.
- Avoid misleading zero values.

---

# 4. Settings Architecture

Recommended top-level input groups:

```text
00 — General
01 — Profiles
02 — Trend
03 — Relative Volume
04 — Market Structure
05 — BOS and CHOCH
06 — Liquidity
07 — VPA
08 — Wyckoff
09 — Institutional Activity
10 — Signal Scoring
11 — Dashboard
12 — Alerts
13 — Visuals
14 — Advanced
15 — Debug
```

Numeric prefixes are recommended to preserve logical ordering in TradingView.

---

# 5. Configuration Levels

Supported configuration levels:

## 5.1 Basic

Displays:

- Master enable
- Profile
- Core sensitivity
- Signal mode
- Dashboard mode
- Alert mode

## 5.2 Standard

Displays normal production settings for all enabled modules.

## 5.3 Advanced

Displays:

- Thresholds
- Decay settings
- Confidence controls
- Correlation controls
- Hysteresis
- Object limits
- Special filters

## 5.4 Debug

Displays:

- Internal state controls
- Diagnostic visuals
- Candidate events
- Raw scores
- Development-only overrides

Suggested default:

```text
Standard
```

---

# 6. General Settings

## 6.1 Enable Indicator

Type:

Boolean

Default:

Enabled

When disabled:

- Analytical calculations may be skipped where practical.
- Dashboard shall be hidden.
- Visuals shall be hidden.
- Alerts shall be disabled.
- No new state events shall be published.

---

## 6.2 Configuration Level

Type:

Selection

Options:

- Basic
- Standard
- Advanced
- Debug

Default:

Standard

---

## 6.3 Global Profile

Type:

Selection

Options:

- Conservative
- Balanced
- Aggressive
- Custom

Default:

Balanced

---

## 6.4 Global Confirmation Mode

Type:

Selection

Options:

- Confirmed Bar Only
- Developing and Confirmed
- Developing Only

Default:

Confirmed Bar Only

This setting shall provide the global default.

Individual modules may allow stricter behaviour but should not silently become less strict without explicit configuration.

---

## 6.5 Calculation Scope

Type:

Selection

Options:

- Full Analysis
- Signals Only
- Analysis Only
- Dashboard Only
- Debug

Suggested default:

Full Analysis

### Full Analysis

All enabled modules operate.

### Signals Only

Modules required for scoring operate, while nonessential visuals are suppressed.

### Analysis Only

Analytical modules operate, but Signal Scoring and signal alerts may be disabled.

### Dashboard Only

Uses available outputs while suppressing optional chart visuals.

### Debug

Enables approved diagnostic paths.

---

## 6.6 Global Sensitivity

Type:

Selection

Options:

- Low
- Medium
- High
- Custom

Default:

Medium

Global Sensitivity may adjust approved module parameters only when the profile is not Custom.

It shall not override explicitly custom module settings.

---

## 6.7 Minimum History Bars

Type:

Integer

Suggested default:

200

Purpose:

Defines the minimum historical context required before the full pipeline may publish mature outputs.

Individual modules may require different minimum history.

---

## 6.8 Confirmed History Only

Type:

Boolean

Suggested default:

Enabled

Purpose:

Restricts production state transitions to confirmed historical bars.

---

## 6.9 Reset State on Symbol or Timeframe Change

Type:

Boolean

Suggested default:

Enabled

Purpose:

Ensures no stale persistent state is carried across chart contexts.

---

# 7. Profile Architecture

Profiles shall define coordinated settings across modules.

Required profiles:

- Conservative
- Balanced
- Aggressive
- Custom

Profiles may control:

- Pivot sensitivity
- BOS and CHOCH confirmation
- Liquidity tolerances
- VPA confidence thresholds
- Wyckoff confidence thresholds
- Institutional dominance thresholds
- Scoring thresholds
- Signal confirmation bars
- Alert thresholds
- Visual density

---

# 8. Conservative Profile

The Conservative profile shall prioritise:

- Higher confirmation thresholds
- Confirmed-bar operation
- Stronger structural requirements
- More confirming modules
- Lower conflict tolerance
- Higher Setup Quality requirement
- Higher Entry Readiness requirement
- Longer cooldown
- Fewer alerts
- Reduced candidate visuals
- Stronger duplicate and correlation control

Illustrative values:

```text
Minimum Signal Confidence: 75
Minimum Setup Quality: 72
Minimum Entry Readiness: 75
Minimum Confirming Modules: 4
Directional Dominance: 20
State Confirmation Bars: 2
Alert Confidence: 75
```

Values remain provisional.

---

# 9. Balanced Profile

The Balanced profile shall prioritise:

- Moderate sensitivity
- Confirmed-bar operation
- Multi-module confirmation
- Moderate conflict tolerance
- Moderate signal thresholds
- Standard dashboard
- Signal-focused alerts

Illustrative values:

```text
Minimum Signal Confidence: 65
Minimum Setup Quality: 65
Minimum Entry Readiness: 70
Minimum Confirming Modules: 3
Directional Dominance: 15
State Confirmation Bars: 1
Alert Confidence: 70
```

---

# 10. Aggressive Profile

The Aggressive profile shall prioritise:

- Earlier event recognition
- Lower thresholds
- Fewer persistence requirements
- Greater reversal sensitivity
- More Watch and Qualified states
- More frequent alerts
- Higher tolerance for incomplete context

It shall still avoid unconfirmed historical repainting by default.

Illustrative values:

```text
Minimum Signal Confidence: 58
Minimum Setup Quality: 58
Minimum Entry Readiness: 62
Minimum Confirming Modules: 2
Directional Dominance: 10
State Confirmation Bars: 1
Alert Confidence: 62
```

---

# 11. Custom Profile

When Custom is selected:

- User-defined module settings shall apply.
- Profile defaults shall not silently overwrite custom values.
- Validation shall still apply.
- Dependency rules shall still apply.
- Unsafe combinations may produce warnings, fallbacks, or Unavailable state.

---

# 12. Profile Override Policy

Two possible policies exist:

## Policy A — Profile Fully Controls Settings

Profile values override related custom inputs.

## Policy B — Profile Provides Defaults

Profile values initialise behaviour, while module overrides remain possible.

Recommended production policy:

```text
Profile Provides Defaults with Explicit Module Overrides
```

An input such as:

```text
Use Profile Default
```

may be used for advanced module controls.

Final implementation remains an open decision.

---

# 13. Module Enablement

Each analytical module shall expose an enable setting.

Required module controls:

- Enable Trend
- Enable RVOL
- Enable Market Structure
- Enable BOS and CHOCH
- Enable Liquidity
- Enable VPA
- Enable Wyckoff
- Enable Institutional Activity
- Enable Signal Scoring
- Enable Dashboard
- Enable Alerts

---

# 14. Dependency Graph

Required analytical dependency order:

```text
Framework
   │
   ├── Trend
   ├── RVOL
   ├── Market Structure
   │      │
   │      └── BOS and CHOCH
   │              │
   │              └── Liquidity
   │
   ├── VPA
   │
   └── Wyckoff
          │
          └── Institutional Activity
                 │
                 └── Signal Scoring
                        │
                        ├── Dashboard
                        └── Alerts
```

The actual dependency graph is not strictly linear, but processing order shall remain deterministic.

---

# 15. Dependency Resolution

When a required upstream module is disabled:

## 15.1 Hard Dependency

The dependent module shall become Unavailable.

Example:

```text
BOS and CHOCH requires Market Structure.
```

If Market Structure is disabled:

```text
BOS and CHOCH = Unavailable
```

## 15.2 Soft Dependency

The dependent module continues with reduced capability or confidence.

Example:

```text
Institutional Activity can operate without volume when strict volume mode is disabled.
```

## 15.3 Optional Enhancement

The dependent module operates normally but omits optional evidence.

Example:

```text
Scoring may operate without Wyckoff when Require Wyckoff Context is disabled.
```

---

# 16. Dependency Classification

Suggested dependency categories:

```text
0 = None
1 = Optional
2 = Soft
3 = Hard
```

Each module interface shall document its dependencies.

---

# 17. Automatic Dependency Enablement

Possible setting:

```text
Automatically Enable Required Dependencies
```

Type:

Boolean

Suggested default:

Enabled

Example:

When the user enables Liquidity while BOS and CHOCH is disabled:

- Market Structure may be enabled.
- BOS and CHOCH may be enabled.
- The user-facing configuration shall reflect or explain this dependency.

Pine Script input values cannot be programmatically changed in the settings panel.

Therefore implementation may instead:

- Treat dependencies as internally active.
- Display dependency status.
- Publish a configuration warning.
- Mark the module Unavailable.

Recommended production policy:

```text
Do not silently simulate enabled settings.
Respect user inputs and mark hard-dependent modules Unavailable.
```

---

# 18. Configuration Warnings

The system shall expose warnings for invalid or degraded configurations.

Examples:

- Liquidity enabled while BOS/CHOCH disabled
- BOS/CHOCH enabled while Market Structure disabled
- Signal Scoring enabled with too few analytical modules
- Strict Volume requirement enabled while volume is unavailable
- Strong Institutional state required while Institutional module is disabled
- Alert module enabled while Signal Scoring is disabled in Signal Only mode
- Debug visuals enabled with object limits too low

Warnings may appear:

- In Dashboard Debug mode
- In a status row
- Through internal diagnostic outputs
- In documentation

The indicator shall not create repetitive alert messages for configuration warnings by default.

---

# 19. Shared Threshold Governance

Thresholds used by multiple modules shall be centralised where appropriate.

Candidates include:

- Global minimum confidence
- Confirmed-bar mode
- ATR length
- Volume availability policy
- Object limits
- State confirmation bars
- Debug mode
- Theme
- Missing-data policy

A module-specific threshold shall not be duplicated globally unless the distinction is clear.

---

# 20. Confidence Scale

All modules shall use a common confidence scale:

```text
0 to 100
```

Suggested semantic bands:

| Confidence | Meaning |
|---|---|
| 0 | Unavailable or invalid |
| 1–39 | Weak |
| 40–59 | Developing |
| 60–74 | Moderate |
| 75–89 | Strong |
| 90–100 | Exceptional |

These bands are presentation guidance, not universal event thresholds.

---

# 21. Score Scale

All normalised directional and setup scores shall use:

```text
0 to 100
```

Raw internal scores may exceed 100 before normalisation.

---

# 22. Price Tolerance Governance

Price tolerances should use consistent units.

Supported tolerance modes:

- Ticks
- Percentage
- ATR multiple
- Hybrid

Suggested production default:

```text
ATR multiple
```

Settings shall clearly identify the unit.

Ambiguous inputs such as:

```text
Tolerance = 1
```

shall be avoided.

Preferred:

```text
Liquidity Sweep Tolerance (ATR)
```

---

# 23. Length and Lookback Naming

Naming standard:

```text
Trend Length
RVOL Length
Pivot Left Bars
Pivot Right Bars
Evidence Lookback Bars
Maximum Signal Age
```

Avoid generic labels such as:

```text
Length
Period
Sensitivity
```

unless the group makes ownership unambiguous.

---

# 24. Boolean Naming

Boolean inputs shall begin with an action or clear state.

Preferred:

- Enable Wyckoff
- Show Dashboard
- Require Structural Trigger
- Include Confidence
- Alert on Invalidation

Avoid:

- Wyckoff
- Dashboard
- Confirmation
- Labels

---

# 25. Selection Naming

Selection inputs shall identify what is being selected.

Preferred:

- Scoring Profile
- Dashboard Mode
- Alert Mode
- Confirmation Mode
- Evidence Decay Mode

---

# 26. Tooltips

Every non-obvious setting should include a concise tooltip.

A tooltip shall explain:

- What the setting controls
- What increasing the value does
- Any dependencies
- Any repaint or alert implications
- Whether it affects performance

Example:

```text
Minimum Entry Readiness:
Minimum trigger-readiness score required before a setup may enter Ready state. Higher values reduce signal frequency.
```

---

# 27. Trend Settings Group

The Trend group shall include only settings owned by the Trend Engine.

Examples:

- Enable Trend
- Trend Classification Mode
- Trend Length
- Fast Smoothing Length
- Slow Smoothing Length
- Slope Lookback
- Trend Strength Threshold
- Neutral Threshold
- Higher-Timeframe Trend
- Higher-Timeframe Confirmation
- Show Trend Visuals
- Show Trend Confidence

Exact inputs remain governed by `02_Trend_Engine.md`.

---

# 28. RVOL Settings Group

Examples:

- Enable RVOL
- RVOL Length
- RVOL Baseline Method
- High RVOL Threshold
- Extreme RVOL Threshold
- Volume Availability Policy
- Missing Volume Behaviour
- Show RVOL
- Show RVOL State

Exact inputs remain governed by `03_RVOL_Engine.md`.

---

# 29. Market Structure Settings Group

Examples:

- Enable Market Structure
- Pivot Left Bars
- Pivot Right Bars
- Minimum Swing Distance
- Equal High Tolerance
- Equal Low Tolerance
- Structure History Limit
- Show Swing Labels
- Show Protection Levels

Exact inputs remain governed by `04_Market_Structure.md`.

---

# 30. BOS and CHOCH Settings Group

Examples:

- Enable BOS and CHOCH
- Break Confirmation Mode
- Require Close Beyond Level
- Minimum Break Distance
- Acceptance Bars
- Failed Break Window
- Show BOS
- Show CHOCH
- Show Failed Breaks

Exact inputs remain governed by `05_BOS_CHOCH.md`.

---

# 31. Liquidity Settings Group

Examples:

- Enable Liquidity
- Liquidity Source Levels
- Sweep Tolerance
- Acceptance Threshold
- Minimum Wick Rejection
- Equal-Level Liquidity
- Liquidity History Limit
- Show Liquidity Levels
- Show Sweeps
- Show Traps

Exact inputs remain governed by `06_Liquidity.md`.

---

# 32. VPA Settings Group

Examples:

- Enable VPA
- Spread Baseline Length
- Volume Baseline Length
- Narrow Spread Threshold
- Wide Spread Threshold
- High Volume Threshold
- Close Location Threshold
- Minimum VPA Confidence
- Primary Classification Only
- Show VPA Labels
- Show Candidate Events in Debug

Exact inputs remain governed by `07_VPA.md`.

---

# 33. Wyckoff Settings Group

Examples:

- Enable Wyckoff
- Range Detection Mode
- Minimum Range Bars
- Maximum Range Bars
- Minimum Boundary Touches
- Range Width Threshold
- Range Confidence Threshold
- Phase Confidence Threshold
- Event Confidence Threshold
- Allow Nested Ranges
- Maximum Active Ranges
- Show Range
- Show Phase
- Show Wyckoff Events

Exact inputs remain governed by `08_Wyckoff.md`.

---

# 34. Institutional Settings Group

Examples:

- Enable Institutional Activity
- Evidence Aggregation Mode
- Evidence Lookback Bars
- Evidence Decay Mode
- Minimum Evidence Confidence
- Minimum Institutional Confidence
- Directional Dominance Threshold
- Conflict Threshold
- Minimum Confirming Modules
- Require Structural Context
- Require Volume Context
- Show Institutional State
- Show Institutional Confidence

Exact inputs remain governed by `09_Institutional.md`.

---

# 35. Signal Scoring Settings Group

Examples:

- Enable Signal Scoring
- Scoring Profile
- Setup Mode
- Minimum Bullish Score
- Minimum Bearish Score
- Minimum Setup Quality
- Minimum Entry Readiness
- Minimum Signal Confidence
- Directional Dominance Threshold
- Maximum Conflict
- Minimum Confirming Modules
- Entry Trigger Mode
- Entry Proximity Limit
- Maximum Signal Age
- State Confirmation Bars
- State Exit Margin
- Enable No-Trade Filters
- Show Signal Labels

Exact inputs remain governed by `10_Scoring.md`.

---

# 36. Dashboard Settings Group

Examples:

- Show Dashboard
- Dashboard Mode
- Dashboard Position
- Dashboard Theme
- Dashboard Size
- Dashboard Transparency
- Show Module Confidence
- Show Evidence Summary
- Maximum Evidence Items
- Show Scores
- Show Setup Quality
- Show Entry Readiness
- Show Unavailable Modules
- Show Debug Rows

Exact inputs remain governed by `11_Dashboard.md`.

---

# 37. Alerts Settings Group

Examples:

- Enable Alerts
- Alert Mode
- Alert Publication Method
- Alert Confirmation Mode
- Alert Frequency Policy
- Minimum Alert Confidence
- Minimum Signal Score
- Minimum Setup Quality
- Minimum Entry Readiness
- Alert Cooldown Bars
- Maximum Alerts Per Bar
- Same-Bar Alert Policy
- Alert Message Detail
- Enable Webhook Format
- Enable Signal Alerts
- Enable Institutional Alerts
- Enable Wyckoff Alerts
- Alert on Invalidation

Exact inputs remain governed by `12_Alerts.md`.

---

# 38. Visual Settings Group

Global chart visual controls may include:

- Show Trend Background
- Show Structure Labels
- Show Liquidity Levels
- Show VPA Labels
- Show Wyckoff Range
- Show Institutional Markers
- Show Signal Markers
- Show Trigger Levels
- Show Invalidation Levels
- Label Size
- Line Width
- Historical Visual Limit
- Visual Theme
- Bullish Colour
- Bearish Colour
- Neutral Colour
- Conflict Colour
- Unavailable Colour

Module-specific visual controls should remain in their module groups where practical.

The Visual group should contain shared appearance settings.

---

# 39. Theme Settings

Supported themes:

- Auto
- Dark
- Light
- Custom

The Theme Manager shall publish semantic colours for:

- Bullish
- Bearish
- Neutral
- Conflict
- Information
- Unavailable
- Candidate
- Confirmed
- Invalidated
- Debug

Modules shall consume semantic colours rather than defining independent colour meaning.

---

# 40. Custom Colour Policy

Custom colours may be supported for:

- Bullish
- Bearish
- Neutral
- Conflict
- Information
- Unavailable
- Dashboard background
- Dashboard text

Default themes shall remain accessible without customisation.

---

# 41. Accessibility Settings

Possible settings:

- High-Contrast Mode
- Use Text Direction Labels
- Use Symbols
- Reduce Transparency
- Large Dashboard Text
- Colour-Blind-Friendly Palette

The first release should at minimum:

- Avoid colour-only interpretation.
- Use explicit text.
- Offer readable contrast.
- Support adjustable text size.

---

# 42. Advanced Settings Group

Advanced settings may include:

- Correlation controls
- Evidence decay
- Category caps
- Module caps
- Hysteresis
- Gap penalties
- Missing-data penalties
- Maximum active evidence
- Maximum active ranges
- Maximum active setups
- Historical storage limits
- Object limits
- State transition policies
- Same-bar event policies

Advanced settings shall not be required for standard use.

---

# 43. Debug Settings Group

Debug settings may include:

- Enable Debug Mode
- Show Candidate Events
- Show Raw Scores
- Show Evidence IDs
- Show Active Evidence Count
- Show Module Availability
- Show Dependency Warnings
- Show Lifecycle States
- Show State Transition Bars
- Show Alert Rejection Reasons
- Show Object Counts
- Show Range IDs
- Show Setup IDs
- Show Confirmation Bars

Debug settings shall be disabled by default.

---

# 44. Internal Constants

Not every parameter shall be exposed as an input.

Internal constants may include:

- Enumeration values
- Maximum hard storage bounds
- Safety clamps
- Correlation-group identifiers
- Event-priority values
- Version identifiers
- Default fallback states
- Hard object limits below TradingView maximums

Internal constants shall be documented in code.

---

# 45. Input Count Governance

TradingView indicators can become difficult to use when input count is excessive.

The project shall:

- Avoid exposing every internal weight.
- Group related controls.
- Use profiles.
- Use advanced mode for calibration.
- Prefer derived thresholds.
- Avoid duplicate visual controls.
- Avoid one Boolean per minor event unless necessary.

A formal maximum user-facing input count should be established before implementation.

---

# 46. Calibration Governance

Weights and thresholds shall be classified as:

## 46.1 User Controls

Safe and understandable settings exposed to users.

## 46.2 Advanced Controls

Expert settings with meaningful documentation.

## 46.3 Internal Calibration Constants

Values controlled by development and testing.

Institutional and Signal Scoring evidence weights should initially remain internal or profile-driven rather than fully user editable.

---

# 47. Default Value Governance

Every default shall be justified by:

- Specification logic
- Test results
- Cross-market compatibility
- Performance
- User clarity
- Repaint policy
- Alert frequency

Defaults shall not be chosen solely from one chart example.

---

# 48. Validation Framework

Each input shall define:

- Type
- Minimum
- Maximum
- Step
- Default
- Valid options
- Dependency
- Fallback
- Error behaviour

Validation shall occur before module processing where practical.

---

# 49. Numeric Validation

Numeric inputs shall be:

- Clamped where safe.
- Rejected through input limits where possible.
- Protected from division by zero.
- Protected from negative lengths.
- Protected from invalid percentages.
- Protected from excessive storage values.

---

# 50. Cross-Setting Validation

Examples:

- Fast Length must be lower than Slow Length where required.
- Strong Threshold must not be lower than Minimum Threshold.
- Maximum Range Bars must exceed Minimum Range Bars.
- Entry Ready Threshold must not be lower than Qualified Threshold where applicable.
- Maximum Signal Age must be positive.
- Minimum Confirming Modules must not exceed enabled evidence sources without warning.
- Conflict Threshold and Dominance Threshold must not create impossible state resolution.

---

# 51. Safe Fallback Behaviour

When invalid combinations occur:

1. Use a safe fallback where deterministic.
2. Publish a configuration warning.
3. Avoid runtime failure.
4. Avoid misleading output.
5. Mark dependent state Unavailable where no safe fallback exists.

Example:

```text
Fast Length >= Slow Length
```

Possible fallback:

- Swap internally
- Clamp Fast Length
- Mark Trend unavailable

The preferred policy shall be defined per module.

---

# 52. Missing Volume Settings

Required global or RVOL-owned controls:

- Missing Volume Behaviour
- Allow Non-Volume Analysis
- Require Volume for VPA
- Require Volume for Institutional
- Require Volume for Signals

Suggested missing-volume behaviour options:

- Continue Without Volume
- Reduce Confidence
- Disable Volume Modules
- Require Volume

Default:

```text
Continue Without Volume and Reduce Confidence
```

---

# 53. Historical Storage Settings

Possible storage controls:

- Swing History Limit
- Structural Event History Limit
- Liquidity Level History Limit
- VPA Event History Limit
- Wyckoff Range History Limit
- Institutional Evidence Limit
- Signal Setup History Limit
- Alert History Limit

User-facing storage controls should be limited.

Hard safety ceilings shall remain internal.

---

# 54. Object Limit Settings

Possible controls:

- Maximum Structure Labels
- Maximum Liquidity Lines
- Maximum VPA Labels
- Maximum Wyckoff Objects
- Maximum Signal Labels
- Maximum Trigger Lines

The system shall enforce a global object budget.

---

# 55. Global Object Budget

The Framework should define approved budgets for:

- Labels
- Lines
- Boxes
- Tables

Each module shall receive a bounded allocation.

When the global budget is under pressure:

1. Delete old debug objects.
2. Delete old candidate objects.
3. Delete old informational objects.
4. Preserve active state objects.
5. Preserve recent Triggered and Invalidated events.
6. Preserve the Dashboard table.

---

# 56. Performance Mode

Optional setting:

```text
Performance Mode
```

Options:

- Full
- Balanced
- Lightweight

### Full

Maximum supported diagnostics and visuals.

### Balanced

Production default.

### Lightweight

Reduces:

- Historical objects
- Evidence history
- Secondary visuals
- Debug calculations
- Optional text construction

Analytical semantics should remain as consistent as possible.

---

# 57. Repainting Settings

User-facing repaint-related controls shall be explicit.

Possible controls:

- Confirmed Bar Only
- Show Developing Candidates
- Allow Intrabar Dashboard Updates
- Allow Provisional Alerts

Default:

- Confirmed states
- Candidate visuals disabled
- Provisional alerts disabled

The system shall not label provisional behaviour as non-repainting.

---

# 58. Real-Time Settings Behaviour

During real-time bars:

- Inputs remain fixed until the user changes settings.
- Developing states may change.
- Confirmed state transitions shall remain stable.
- Dashboard may update intrabar if enabled.
- Alerts follow Confirmation Mode.
- Historical confirmed events shall not be rewritten.

---

# 59. Settings Change Behaviour

When a user changes settings:

- Pine Script recalculates historical state.
- Historical outputs may legitimately differ under the new configuration.
- This is not considered repainting under a fixed configuration.
- Active IDs and state may be regenerated.
- Old external alerts cannot be withdrawn.
- Dashboard shall reflect the new configuration.

Documentation shall distinguish:

```text
Recalculation after settings change
```

from:

```text
Repainting under unchanged settings
```

---

# 60. Symbol and Timeframe Change Behaviour

On symbol or timeframe change:

- All modules shall recalculate.
- Persistent state shall reset.
- Event IDs shall be namespaced or regenerated.
- Alert duplicate history shall reset safely.
- No prior chart context shall remain active.
- Dashboard shall not display stale values.

---

# 61. Versioning

The Settings architecture shall expose an internal settings-schema version.

Conceptual value:

```text
SETTINGS_SCHEMA_VERSION = 1
```

The schema version shall change when:

- Input semantics change materially.
- Profile definitions change materially.
- Webhook settings change incompatibly.
- Enumeration options are removed or renamed.
- Migration is required.

---

# 62. Input Naming Stability

After Version 1.0:

- Input titles should not be changed casually.
- Group names should remain stable.
- Selection option strings should remain stable.
- Removed settings should be documented.
- Replacement settings should be identified.

TradingView may not support automatic migration of all user presets.

Therefore naming stability is operationally important.

---

# 63. Deprecated Settings

When a setting becomes obsolete:

- Mark it deprecated in documentation.
- Preserve it temporarily where practical.
- Ignore it safely if no longer used.
- Provide a replacement path.
- Remove it only in a documented breaking release.

---

# 64. Profile Versioning

Profiles shall have documented versions.

Example:

```text
Balanced Profile v1
```

When profile calibration changes materially:

- Changelog entry required.
- Release notes required.
- Regression testing required.
- Users should be informed that signals may differ.

---

# 65. Reset Behaviour

Possible setting:

```text
Reset to Profile Defaults
```

Pine Script cannot programmatically reset input controls in the settings panel.

Therefore the implementation may instead provide:

- Documented profile defaults
- Preset tables in documentation
- Input tooltip guidance
- Versioned default references

The interface shall not imply a reset capability that TradingView cannot perform.

---

# 66. Preset Documentation

The project should publish a reference table for:

- Conservative
- Balanced
- Aggressive
- Lightweight
- Research or Debug

Each preset table shall list:

- Key thresholds
- Enabled modules
- Confirmation policy
- Dashboard mode
- Alert mode
- Expected signal frequency

---

# 67. Market-Specific Presets

Potential future presets:

- Stocks
- Forex
- Cryptocurrency
- Futures
- Indices
- Low-volume assets

Version 1.0 should avoid automatic market-specific calibration unless extensively tested.

Manual profile guidance may be documented.

---

# 68. Timeframe Presets

Potential future presets:

- Scalping
- Intraday
- Swing
- Position

Version 1.0 should not silently change settings based on chart timeframe.

Any adaptive timeframe behaviour shall be explicit and documented.

---

# 69. Adaptive Settings

Adaptive settings may later respond to:

- Volatility
- Market regime
- Timeframe
- Session
- Instrument type
- Volume availability

Adaptive behaviour shall not be introduced into Version 1.0 without:

- Specification
- Deterministic rules
- Testing
- Dashboard disclosure
- Override controls

---

# 70. User Override Precedence

Suggested precedence:

1. Hard safety limits
2. Explicit Custom user setting
3. Module override
4. Global profile default
5. System fallback

This order shall remain deterministic.

---

# 71. Settings Resolution Pipeline

Recommended resolution order:

```text
Read Inputs
     │
     ▼
Resolve Global Profile
     │
     ▼
Apply Module Overrides
     │
     ▼
Validate Ranges
     │
     ▼
Validate Cross-Dependencies
     │
     ▼
Apply Safety Clamps
     │
     ▼
Publish Effective Settings
     │
     ▼
Run Analytical Pipeline
```

Modules should consume effective settings rather than repeatedly resolving profile logic.

---

# 72. Effective Settings Object

Conceptual structure:

```text
EffectiveSettings
{
    General
    Trend
    RVOL
    Structure
    BOSCHOCH
    Liquidity
    VPA
    Wyckoff
    Institutional
    Scoring
    Dashboard
    Alerts
    Visuals
    Debug
}
```

Pine Script implementation may use:

- Grouped scalar variables
- User-defined types
- Helper functions
- Stable naming prefixes

---

# 73. Effective Settings Publication

Each module shall receive:

- Resolved enable state
- Resolved profile values
- Resolved custom overrides
- Validated thresholds
- Dependency availability
- Debug state
- Visual state
- Alert state where relevant

Modules shall not independently reinterpret profile names.

---

# 74. Settings Naming Convention

Suggested code prefixes:

```text
cfgGeneral_
cfgTrend_
cfgRvol_
cfgStructure_
cfgBos_
cfgLiquidity_
cfgVpa_
cfgWyckoff_
cfgInstitutional_
cfgScoring_
cfgDashboard_
cfgAlerts_
cfgVisual_
cfgDebug_
```

Resolved values may use:

```text
effTrend_
effScoring_
effAlerts_
```

---

# 75. Input Variable Naming

Suggested Pine Script pattern:

```text
inputEnableTrend
inputTrendLength
inputShowDashboard
inputAlertMode
```

Alternative concise form:

```text
inTrendEnabled
inTrendLength
inDashboardShow
inAlertMode
```

Final code naming shall follow `CODING_STANDARD.md`.

---

# 76. Enumeration Governance

Selection options should map to stable internal enums.

Example:

```text
"Conservative" → PROFILE_CONSERVATIVE
"Balanced"     → PROFILE_BALANCED
"Aggressive"   → PROFILE_AGGRESSIVE
"Custom"       → PROFILE_CUSTOM
```

Analytical modules shall consume enums rather than compare user-facing strings repeatedly.

---

# 77. Selection Option Stability

After implementation begins:

- Existing options should not be reordered without review.
- Existing semantic meanings should not change silently.
- New options should be appended where practical.
- Internal enum values shall remain stable.

---

# 78. Disabled State Semantics

When a module is disabled:

- Enabled status = false
- Primary state = Unavailable
- Confidence = 0 or unavailable indicator
- Event publication = disabled
- Visuals = hidden
- Alerts = disabled
- Downstream evidence = absent

Zero confidence shall not be displayed as genuine low confidence when the module is unavailable.

---

# 79. Unavailable Versus Neutral

Settings and downstream modules shall distinguish:

```text
Unavailable
```

from:

```text
Neutral
```

Unavailable means the calculation cannot or should not be performed.

Neutral means the calculation is valid but no directional state is present.

---

# 80. Warning Severity

Configuration warnings may use:

- Information
- Degraded
- Invalid
- Critical

Examples:

## Information

Debug mode enabled.

## Degraded

Volume unavailable; confidence reduced.

## Invalid

Liquidity enabled without required structure pipeline.

## Critical

No analytical modules available for Signal Scoring.

---

# 81. Dashboard Integration

The Dashboard shall be able to display:

- Active Global Profile
- Confirmation Mode
- Performance Mode
- Disabled modules
- Unavailable modules
- Configuration warning count
- Highest-priority warning
- Debug state
- Volume availability policy

Standard mode should show only critical configuration issues.

Debug mode may show all effective settings.

---

# 82. Alert Integration

The Alerts module shall consume:

- Effective alert enablement
- Alert mode
- Confirmation mode
- Thresholds
- Cooldowns
- Event enablement
- Message format
- Webhook mode
- Debug state

Settings changes shall not retroactively publish historical events by default.

---

# 83. Documentation Requirements

Each user-facing setting shall be documented with:

- Name
- Group
- Type
- Default
- Valid range or options
- Purpose
- Effect of increasing or decreasing
- Dependencies
- Performance impact
- Repaint impact
- Alert impact
- Profile behaviour

Documentation may be split between module specifications and a future user guide.

---

# 84. Testing Requirements

Settings shall be tested across:

- All profiles
- All configuration levels
- Every module enabled
- Every module disabled
- Individual hard dependencies disabled
- Soft dependencies unavailable
- Missing volume
- Custom thresholds
- Boundary values
- Invalid combinations
- Symbol changes
- Timeframe changes
- Theme changes
- Dashboard modes
- Alert modes
- Performance modes
- Debug mode
- Historical recalculation

---

# 85. Functional Test Cases

## Test SET-001 — Balanced Defaults

Given:

- Global Profile is Balanced.
- No custom overrides are active.

Expected:

- All Balanced effective settings resolve correctly.
- Production confirmation remains confirmed-bar only.
- Standard Dashboard and Signal Only alert defaults apply.

---

## Test SET-002 — Conservative Profile

Given:

- Conservative profile is selected.

Expected:

- Confidence and readiness thresholds increase.
- Confirming-module requirements increase.
- Alert frequency decreases.
- State persistence increases where specified.

---

## Test SET-003 — Aggressive Profile

Given:

- Aggressive profile is selected.

Expected:

- Watch and Qualified thresholds decrease.
- Signal frequency may increase.
- Confirmed historical-state policy remains intact unless explicitly changed.

---

## Test SET-004 — Custom Profile

Given:

- Custom profile is selected.
- User-defined Signal Confidence equals 68.

Expected:

- The custom value remains effective.
- Profile defaults do not overwrite it.

---

## Test SET-005 — Hard Dependency Failure

Given:

- Market Structure is disabled.
- BOS and CHOCH is enabled.

Expected:

- BOS and CHOCH becomes Unavailable.
- A configuration warning is published.
- No runtime error occurs.

---

## Test SET-006 — Soft Dependency Failure

Given:

- Volume is unavailable.
- Institutional Activity is enabled.
- Require Volume Context is disabled.

Expected:

- Institutional Activity remains operational.
- Volume contribution is omitted.
- Confidence is reduced where specified.

---

## Test SET-007 — Strict Volume Requirement

Given:

- Volume is unavailable.
- Require Volume for Signals is enabled.

Expected:

- Ready and Triggered signals are withheld.
- Signal state becomes No Trade or Unavailable according to policy.

---

## Test SET-008 — Cross-Threshold Validation

Given:

- Strong Signal Threshold is set below Minimum Signal Score.

Expected:

- The effective Strong Signal Threshold is corrected or the configuration is rejected safely.
- A warning is available.
- No impossible state logic occurs.

---

## Test SET-009 — Invalid Range Lengths

Given:

- Minimum Range Bars exceeds Maximum Range Bars.

Expected:

- Safe correction or Wyckoff Unavailable state occurs.
- No runtime error occurs.

---

## Test SET-010 — Minimum Module Count Too High

Given:

- Minimum Confirming Modules equals six.
- Only three qualifying modules are enabled.

Expected:

- Signals cannot qualify.
- Configuration warning indicates the impossible requirement.
- Scores remain available where valid.

---

## Test SET-011 — Dashboard Disabled

Given:

- Dashboard is disabled.

Expected:

- No table is created.
- Analytical modules continue.
- Alerts continue if enabled.

---

## Test SET-012 — Alerts Disabled

Given:

- Alerts are disabled.

Expected:

- No alerts publish.
- Analytical events and Dashboard remain operational.

---

## Test SET-013 — Scoring Disabled

Given:

- Signal Scoring is disabled.
- Signal Alerts are enabled.

Expected:

- Signal alerts are unavailable.
- Upstream analytical alerts may remain operational.
- A configuration warning is available.

---

## Test SET-014 — Lightweight Performance Mode

Given:

- Performance Mode is Lightweight.

Expected:

- Secondary visuals and debug features are reduced.
- History limits decrease within safe bounds.
- Core analytical semantics remain operational.

---

## Test SET-015 — Debug Mode

Given:

- Configuration Level is Debug.
- Show Candidate Events is enabled.

Expected:

- Candidate states and diagnostics are visible.
- Production confirmed states remain separately identifiable.
- Provisional alerts remain disabled unless explicitly enabled.

---

## Test SET-016 — Theme Change

Given:

- Theme changes from Dark to Light.

Expected:

- Semantic colours update.
- State meaning remains unchanged.
- No analytical recalculation dependency exists beyond normal Pine execution.

---

## Test SET-017 — Symbol Change

Given:

- Active state exists.
- Chart symbol changes.

Expected:

- Persistent state resets.
- Old event IDs are not reused.
- No stale Dashboard or alert state remains.

---

## Test SET-018 — Timeframe Change

Given:

- Active setup exists.
- Chart timeframe changes.

Expected:

- Setup lifecycle recalculates.
- Old Setup ID does not remain active.
- Effective settings remain consistent.

---

## Test SET-019 — Settings Change Recalculation

Given:

- Pivot sensitivity is changed.

Expected:

- Historical structure and dependent modules recalculate.
- Resulting historical changes are documented as configuration recalculation.
- No runtime error occurs.

---

## Test SET-020 — Unavailable Versus Neutral

Given:

- Trend Engine is disabled.

Expected:

- Trend displays N/A rather than Neutral.
- Trend confidence is not displayed as a valid zero-confidence result.

---

## Test SET-021 — Input Boundary Values

Given:

- Numeric inputs are set to their minimum and maximum allowed values.

Expected:

- The indicator compiles and runs.
- No array or object overflow occurs.
- Performance remains bounded.

---

## Test SET-022 — Object Budget Pressure

Given:

- Visual history settings are near maximum.
- Multiple modules create objects.

Expected:

- Global object budget rules are enforced.
- Lower-priority objects are removed first.
- Dashboard and active state visuals remain.

---

## Test SET-023 — Profile Override

Given:

- Balanced profile is selected.
- A permitted module override changes Minimum Wyckoff Confidence.

Expected:

- The module override takes precedence.
- Other Balanced settings remain unchanged.

---

## Test SET-024 — Historical Stability Under Fixed Settings

Given:

- Settings remain unchanged.
- Additional future bars are loaded.

Expected:

- Confirmed historical event bars remain stable.
- Current state evolves only through new evidence.

---

# 86. Acceptance Criteria

The Settings module shall be complete when:

- All inputs are organised into approved groups.
- Every setting has one owner.
- Conservative, Balanced, Aggressive, and Custom profiles resolve deterministically.
- Effective settings are published before analytical processing.
- Hard, soft, and optional dependencies are defined.
- Invalid configurations are handled safely.
- Cross-setting validation works.
- Disabled modules publish Unavailable rather than misleading Neutral states.
- Missing-volume policies work.
- Configuration warnings are exposed.
- Global confirmation behaviour is explicit.
- Debug and provisional behaviours are disabled by default.
- Object and history limits remain bounded.
- Theme and accessibility settings operate consistently.
- Settings changes trigger safe recalculation.
- Symbol and timeframe changes reset persistent state safely.
- Input naming and option strings follow approved conventions.
- Versioning and deprecation policies are documented.
- TradingView compilation succeeds.
- Required tests pass.

---

# 87. Open Design Decisions

The following items remain unresolved:

1. Final number and order of input groups.
2. Final Configuration Level options.
3. Final Global Profile values.
4. Whether module overrides are allowed under non-Custom profiles.
5. Final profile override precedence.
6. Final Conservative profile calibration.
7. Final Balanced profile calibration.
8. Final Aggressive profile calibration.
9. Whether a Lightweight profile is separate from Performance Mode.
10. Final global sensitivity behaviour.
11. Final minimum history requirement.
12. Final hard, soft, and optional dependency map.
13. Final dependency-warning presentation.
14. Whether hard dependencies may be internally activated.
15. Final missing-volume policy.
16. Final confidence semantic bands.
17. Final tolerance unit policy.
18. Final shared ATR length ownership.
19. Final input-count target.
20. Final user-facing calibration controls.
21. Which evidence weights remain internal.
22. Final object-budget allocation.
23. Final history-limit defaults.
24. Final Performance Mode behaviour.
25. Final Debug controls.
26. Final provisional-state controls.
27. Final custom colour set.
28. Final accessibility options.
29. Final effective-settings implementation.
30. Whether user-defined types are used for settings.
31. Final variable naming convention.
32. Final internal enum mappings.
33. Final configuration-warning enum.
34. Final warning-priority order.
35. Final symbol-change reset policy.
36. Final timeframe-change reset policy.
37. Final settings-schema version.
38. Final profile-version policy.
39. Final deprecation lifetime.
40. Final preset documentation format.
41. Whether market-specific presets are included in Version 1.0.
42. Whether timeframe presets are included in Version 1.0.
43. Whether adaptive settings are allowed in Version 1.0.
44. Final Dashboard display of configuration state.
45. Final alert behaviour after settings changes.
46. Whether a current-state alert option is exposed.
47. Final cross-threshold correction policy.
48. Whether invalid values are clamped or make a module unavailable.
49. Final global object-pressure deletion order.
50. Final maximum supported input count.

---

# 88. Future Enhancements

Possible future enhancements include:

- Importable preset documentation
- Market-specific profiles
- Timeframe-specific profiles
- Session-specific profiles
- Volatility-adaptive settings
- Regime-adaptive thresholds
- Guided setup wizard
- Automatic configuration diagnostics
- Profile comparison panel
- User preset export references
- Settings migration assistant
- Dynamic performance budgeting
- Advanced accessibility themes
- Simplified beginner mode
- Expert calibration mode
- Statistical parameter recommendations
- Offline optimisation tooling
- Versioned cloud preset documentation