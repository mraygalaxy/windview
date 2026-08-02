# WindView — Feature & Technical Reference

This document covers every feature, control, and physics model used in `windview.html`.

---

## Feature overview

| Category | Feature |
|----------|---------|
| Visualization | Animated wind clock (rotatable) |
| Visualization | Board with heading arrow |
| Visualization | Projected tacking path |
| Visualization | Animated true-wind particles |
| Visualization | Two adjustable tacking boundary lines |
| Visualization | Fixed shoreline / land reference |
| Visualization | On-canvas angle of attack HUD |
| Physics | Iterative apparent-wind speed model |
| Physics | Hydrofoil vs. flat-board drag modes |
| Physics | Wing lift/drag from area and aspect ratio |
| Physics | Body air drag (rider) |
| Physics | VMG (Velocity Made Good) calculation |
| Controls | Play / Pause / Restart |
| Controls | Wind clock rotation (wind shift simulation) |
| Controls | Angle of attack ±5° |
| Controls | Tacking lane width ± |
| Controls | Wind speed (mph) |
| Controls | Foiling mode checkbox |
| Controls | Wing area, rider weight, foil area, board volume |
| Data | Export simulation profile as JSON |
| Data | Import simulation profile from JSON |

---

## Canvas layout

The canvas is divided into two fixed zones:

| Zone | Height | Description |
|------|--------|-------------|
| Water | 90% of canvas | The sailing surface — all animation happens here |
| Land | 10% of canvas | Fixed shoreline at the bottom; origin of all launches |

The wind clock is centered in the water zone. All canvas elements are responsive and rescale on window resize.

---

## Wind clock

The wind clock is a circle drawn at the center of the water zone with all 12 hour positions labeled.

### Default orientation (windAngle = 0°)

- **12** is at the top of the screen. Wind blows **downward** from 12 toward 6.
- **3** is at screen right, **9** at screen left — these mark the crosswind axis.
- **6** is at the bottom — dead downwind, toward the shoreline.

### Rotation behavior

Pressing the wind direction buttons rotates the entire clock clockwise or counter-clockwise. This represents a real **wind shift** — a change in the compass direction the wind is blowing from.

- The land/shore does **not** rotate. It is always fixed at the bottom of the screen.
- The tacking lines rotate with the clock because they are always parallel to the wind direction.
- The board's heading and bounce behavior adjust automatically.

### Visual elements on the clock

| Element | Color | Meaning |
|---------|-------|---------|
| 12 label | Red | Wind source — where the wind comes from |
| 6 label | Orange | Downwind — where the wind is going |
| 3 and 9 labels | Cyan | Crosswind axis |
| Other numbers | Dim white | Intermediate angles |
| Red arrow | Red | Wind direction indicator (from 12 toward 6) |
| Horizontal cyan line | Cyan | 3–9 axis (perpendicular to wind) |

---

## Tacking boundary lines

Two dashed lines run parallel to the wind direction, one on each side of the clock center.

- **Yellow line** — port (left) tacking boundary
- **Cyan line** — starboard (right) tacking boundary

### Purpose

These lines represent the lateral boundaries of the sailor's available water — the left and right edges of a bay, lake, or sailing lane. The board bounces off them automatically, simulating a perfect tack or jibe.

### Width adjustment

The **Tacking Width** ± buttons change the spacing between the two lines. Spacing is stored as a multiplier on the clock radius:

| Spacing value | Effect |
|---------------|--------|
| ~0.12 (minimum) | Lines very close together — many tacks required to go upwind |
| 1.0 (default) | Lines pass through the 3 and 9 positions of the clock |
| 2.6 (maximum) | Lines well outside the clock circle — very wide lane |

### Bounce mechanics

When the board's perpendicular distance from the clock center (measured along the 3–9 axis) reaches the half-lane width, its tack direction flips. The along-wind component of its velocity is unchanged. This models a **perfect tack or jibe** with no downwind drift.

> **Planned advanced feature:** A tack/jibe efficiency parameter (0–100%) that adds realistic downwind drift during maneuvers, reducing upwind efficiency on slower or less skilled turns.

---

## Board animation

### Starting position

The board always starts near the bottom-center of the water zone, just above the shoreline.

### Velocity

Board velocity = `calcSpeed() × 4 px/knot-equivalent`

The `4 px/knot` scale factor keeps the animation at a visually legible speed regardless of screen size.

### Heading vector

The board heading in screen space is computed from three values:

1. `boardAngle` — angle from crosswind toward upwind (user-set)
2. `tack` — +1 or −1 (flips at each boundary bounce)
3. `windAngle` — current clock rotation

```
headX = tack × perpX × cos(boardAngle) + upwindX × sin(boardAngle)
headY = tack × perpY × cos(boardAngle) + upwindY × sin(boardAngle)
```

Where `perpX/Y` is the 3–9 axis unit vector and `upwindX/Y` is the unit vector pointing from 6 toward 12.

### Stop conditions

The simulation stops (and the Play button changes to Restart) when:

| Condition | Message |
|-----------|---------|
| `board.y >= waterHeight` | "Board returned to land!" |
| `board.y <= 0` | "Made it upwind! 🎉" |
| `board.x < 0` or `board.x > width` | "Board went off screen" |

---

## Projected path

A dashed yellow line traces the board's future tacking path forward from its current position. It:

- Follows the same heading vector as the board
- Bounces at the same tacking lines
- Traces up to 10 bounces or 3000 steps
- Stops at canvas edges or the shoreline

---

## Wind particles

60 particles drift across the water area in the true wind direction. Each particle:

- Has a random speed factor (0.35×–1.0× of base wind speed)
- Has a random opacity (scales with wind speed — stronger wind = more visible)
- Has a random streak length (14–36 px)
- Wraps around all four canvas edges when it exits the water zone
- Displays a small arrowhead pointing in the wind direction

---

## Angle of attack HUD

A semi-transparent overlay in the top-left corner of the canvas shows:

- **Angle of attack** — current `boardAngle` in degrees (e.g. `+45°`)
- **TWA from upwind** — True Wind Angle measured from dead upwind (e.g. `45°`)
- **Heading** — the corresponding clock position for the current tack (e.g. `1:30`)

---

## Physics model

### Overview

Speed is calculated by iteratively converging on the equilibrium boat speed where **driving force = total drag**, accounting for apparent wind (which shifts as the board accelerates).

### Inputs

| Parameter | Symbol | Default | Source |
|-----------|--------|---------|--------|
| True wind speed | Vw | 15 mph | User |
| True wind angle | TWA | 45° | Derived from boardAngle |
| Wing area | A_wing | 6.0 m² | User |
| Wing lift coefficient | CL | 1.2 | Fixed |
| Wing drag coefficient | CD_wing | 0.09 | Fixed (includes induced drag) |
| Air density | ρ_air | 1.225 kg/m³ | Fixed |
| Water density | ρ_water | 1025 kg/m³ | Fixed |
| Rider frontal area | A_body | 0.55 m² | Fixed |
| Body drag coefficient | CD_body | 0.9 | Fixed |

### Foiling mode

| Parameter | Symbol | Default | Source |
|-----------|--------|---------|--------|
| Foil drag coefficient | CD_foil | 0.025 | Fixed |
| Foil area | A_foil | 1500 cm² = 0.15 m² | User |

### Flat-board mode

| Parameter | Symbol | Derivation |
|-----------|--------|------------|
| Board drag coefficient | CD_board | 0.11 |
| Effective wetted area | A_board | (boardVolume / 100) × 0.52 m² |

A 100 L board yields approximately 0.52 m² effective drag area. This is a rough empirical approximation — actual hull resistance varies with hull shape, speed, and wave conditions.

### Iteration (apparent wind loop)

```
For each iteration (max 100):
  1. Compute apparent wind vector:
       Va_fwd  = Vw × cos(TWA) − Vb        (headwind minus boat speed)
       Va_side = Vw × sin(TWA)              (crosswind, unchanged)
       Va      = √(Va_fwd² + Va_side²)

  2. Apparent wind angle from boat's bow:
       AWA = atan2(Va_side, Va_fwd)

  3. Wing forces:
       F_lift  = 0.5 × ρ_air × Va² × CL × A_wing
       F_Dwing = 0.5 × ρ_air × Va² × CD_wing × A_wing

  4. Net driving force along boat heading:
       F_drive = F_lift × sin(AWA) − F_Dwing × cos(AWA)
       If F_drive ≤ 0: boat speed = 0, stop

  5. Total drag:
       K = 0.5 × ρ_water × CD_water × A_water
         + 0.5 × ρ_air   × CD_body  × A_body
       F_drag = K × Vb²

  6. Equilibrium speed:
       Vb_eq = √(F_drive / K)

  7. Damped update:
       Vb_new = 0.35 × Vb + 0.65 × Vb_eq
       If |Vb_new − Vb| < 0.0008 m/s: converged, stop
       Vb = Vb_new
```

### No-go zone

If TWA < 38° (board heading within 38° of dead upwind) or TWA > 172° (nearly dead downwind), the function returns 0 mph. In practice:

- Below 38° TWA the wing stalls and generates no useful drive
- Above 172° the model breaks down (dead downwind sailing is a degenerate case for modern rigs)

### VMG calculation

```
VMG = boatSpeed × sin(boardAngle)
```

`boardAngle` is in degrees from crosswind. At 0° (crosswind), VMG = 0. At 90° (dead upwind, which is the no-go zone), VMG would theoretically equal boat speed — but the boat cannot reach that angle.

### Unit convention

All speeds are displayed in **miles per hour (mph)** in the UI, because this is more intuitive for users unfamiliar with nautical units. Internally, all physics calculations use **m/s**:

- 1 mph = 0.44704 m/s (conversion used in `calcSpeed()`)

---

## Simulation profile (JSON format)

Exported files contain all settings needed to fully restore a simulation:

```json
{
  "name":        "Lake Tahoe — north launch",
  "windAngle":   15,
  "boardAngle":  45,
  "spacing":     1.0,
  "windSpeed":   18,
  "launchTack":  1,
  "isForiling":  true,
  "wingArea":    6.0,
  "riderWeight": 200,
  "foilArea":    1500,
  "boardVolume": 100
}
```

| Field | Type | Unit | Description |
|-------|------|------|-------------|
| `name` | string | — | Location or session label |
| `windAngle` | number | degrees | Clock rotation (0–359) |
| `boardAngle` | number | degrees | −85 to +85 from crosswind |
| `spacing` | number | × clockR | Tacking lane half-width multiplier |
| `windSpeed` | number | mph | True wind speed |
| `launchTack` | number | +1 or −1 | Initial launch direction: +1 = starboard (toward 3), −1 = port (toward 9) |
| `isForiling` | boolean | — | true = hydrofoil, false = flat board |
| `wingArea` | number | m² | Wing/sail area |
| `riderWeight` | number | lbs | Combined rider + gear weight |
| `foilArea` | number | cm² | Hydrofoil front wing area |
| `boardVolume` | number | liters | Board volume (used when not foiling) |

---

## Planned advanced features

The following features are noted for future development:

### Tack/jibe drift
Real tacks and jibes cause the rider to drift slightly downwind during the maneuver. A **tack/jibe efficiency** slider (0–100%) would add a configurable downwind offset on each bounce, reducing upwind efficiency on slower turns. Currently all bounces are instantaneous and drift-free.

### Apparent wind overlay
An optional second arrow displayed on the board showing the **apparent wind** vector — the wind that the rider actually feels, which is the vector sum of true wind and the negative of boat velocity. At foiling speeds the apparent wind shifts dramatically forward (toward the bow), which is why wing foilers can sail surprisingly close to the wind at high speed.

---

## Browser compatibility

The app uses standard Canvas 2D API and ES6 JavaScript. It has been designed to work in:

- Chrome / Chromium 90+
- Firefox 88+
- Safari 14+
- Edge 90+

No external libraries, frameworks, or network requests are used. The entire application is self-contained in a single `.html` file and can be emailed, shared via USB, or opened directly from disk.
