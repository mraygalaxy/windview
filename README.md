# WindView — Wind Sports Simulator

A self-contained browser app for visualizing and learning the fundamentals of wind-powered water sports. Open `windview.html` in any modern browser — no installation, no internet required.

---

## Who this is for

If you have ever stood at a beach or lake and wondered *"the wind is blowing sideways — how do I possibly get back upwind to where I started?"* this app is for you.

WindView is aimed at **beginners and intermediate riders** learning wing foiling, windsurfing, kitesurfing, or any other wind-powered water sport. It visualizes the single most confusing concept in all of sailing: the **wind clock** — and shows you, in real time, how your angle to the wind determines your speed, your path, and how efficiently you can climb back upwind.

---

## The core concept: the wind clock

Imagine a clock face lying flat on the surface of the water. The wind always blows from the **12 o'clock position** straight toward the **6 o'clock position**.

```
         12  ← wind comes FROM here
          ↓
  9 ←    · ←→   → 3
          ↓
          6  ← wind blows TO here (downwind)
```

Your board can reach almost any direction — **but not straight into the wind**. There is a "no-go zone" of roughly 38° on either side of 12 o'clock where the wind cannot push you forward no matter what you do.

### Going upwind

To reach a destination that is directly upwind (at 12 o'clock), you cannot go straight there. Instead you zigzag:

- First you sail toward **1:30 or 2 o'clock** (angled into the wind on one side)
- Then you turn and sail toward **10:30 or 10 o'clock** (the mirror angle on the other side)
- Each turn alternates between **tacking** (turning through the wind) and **jibing** (turning away from the wind)

This zigzag is called **beating to windward**. The tighter your angle (closer to 12), the more directly you head upwind, but the slower you go because the wing generates less power. The trade-off between angle and speed is what WindView helps you explore.

### Angles explained (0° to 90°)

In this app, your **angle of attack** is measured from the crosswind direction:

| Angle | Clock position | What it means |
|-------|---------------|---------------|
| 0°    | 3:00 or 9:00  | Pure crosswind — maximum power, no upwind progress |
| 30°   | 2:00 or 10:00 | Efficient upwind angle for most conditions |
| 45°   | 1:30 or 10:30 | Common beginner upwind angle |
| 60°   | 1:00 or 11:00 | Aggressive upwind angle — requires skill and good wind |
| 80°+  | ~12:00        | No-go zone — you will stall and drift backward |
| −45°  | 4:30 or 7:30  | Broad reach — fast, going downwind |
| −90°  | 6:00          | Dead downwind — slowest for most modern rigs |

### VMG — Velocity Made Good

**VMG** stands for *Velocity Made Good* — the component of your speed that is actually going in the direction you want (upwind). It is the most important number in sailing.

- A board doing **20 mph at 30° angle** has a VMG of about **10 mph upwind**.
- A board doing **8 mph at 80° angle** has a VMG of only **7.9 mph upwind** — and is much slower overall.

The best upwind VMG comes from finding the sweet spot between angle and speed — not the highest angle, and not the fastest speed. WindView displays VMG continuously so you can see this relationship in action.

---

## Your setup (defaults)

The physics model is pre-configured to match a typical wing foil setup:

- **Wing:** 6.0 m² (good all-around size for 15–25 knot winds)
- **Rider weight:** 200 lbs including gear
- **Hydrofoil:** 1500 cm² front wing (a mid-size performance foil)
- **Board:** 100 L (large enough to float the rider before foiling)

You can change any of these in the controls panel to explore how equipment affects performance.

---

## How to use WindView

### Quick start

1. Open `windview.html` in Chrome, Firefox, or Safari.
2. The wind clock appears in the center of the water. Wind blows downward (from 12 toward 6) by default.
3. Press **▶ Play** and watch the board zigzag upward across the screen, bouncing between the two dashed lines.
4. Press **⏸ Pause** at any time to freeze the simulation.
5. Use the controls on the right to change settings and press Play again.

### What you are watching

- **The blue-green water area** is the lake or ocean surface, viewed from directly above (like from an airplane).
- **The yellow-green strip at the bottom** is the shore — where you launch from.
- **The circle with numbers** is the wind clock. It rotates when you simulate a wind shift.
- **The two dashed lines** (yellow and cyan) are your tacking boundaries — the width of your sailing lane.
- **The small white shape** is your board. The yellow arrow shows its heading.
- **The dashed yellow trail ahead** shows the projected path the board will take, including future bounces.
- **The soft blue arrows** flowing across the water are the wind particles — they show the true wind direction.

### Controls reference

| Control | What it does |
|---------|-------------|
| **▶ Play / ⏸ Pause** | Start or pause the board animation. If the board stopped, changes to **↺ Restart** |
| **Wind Direction** | Rotates the entire wind clock. Simulates a wind shift during your session. ±5° for fine tuning, ±15° for big shifts |
| **Angle of Attack** | Changes the board's heading relative to the wind. +5° = point more into wind; −5° = bear away downwind |
| **Tacking Width** | Moves the two boundary lines closer or farther apart. A narrow lane = more tacks to go upwind. A wide lane = fewer tacks but longer legs |
| **Wind Speed** | Sets the true wind speed in knots. Affects calculated board speed and wind particle animation speed |
| **Hydrofoiling mode** | Checkbox that switches between foiling (very low drag) and flat-board (higher drag) physics |
| **Wing area** | Size of your wing/sail in square meters. Larger wing = more power at lower wind speeds |
| **Rider weight** | Your weight including gear in lbs. Heavier rider needs more wind to foil |
| **Foil area** | The surface area of your hydrofoil front wing in cm². Larger = more lift at lower speeds but more drag |
| **Board volume** | Only active when not foiling. Volume in liters; affects hull drag estimate |

### Simulating a wind shift

Real-world wind rarely blows from exactly one direction all session. Use the **Wind Direction** buttons to rotate the clock and see how a shift affects your tacking angles and VMG. For example:

- A **header** (wind shifts toward you) closes your upwind angle and forces you to tack.
- A **lift** (wind shifts away from you) opens your angle and lets you point higher.

### Saving and loading sessions

Each collection of settings (location, wind, equipment) is a **simulation profile**. You can:

1. Type a name in the **Location name** field (e.g. "Lake Tahoe — north end").
2. Click **⬇ Export JSON** to download a `.json` file to your computer.
3. Later, click **⬆ Import JSON** to reload those exact settings and continue exploring.

This lets you build a library of locations and conditions — useful for comparing different lakes, or for planning sessions before you arrive.

---

## Understanding the numbers

### Speed (knots)

The calculated board speed using the physics model for your current settings. As a rough guide:

| Condition | Typical speed |
|-----------|--------------|
| Light wind (10 mph), flat board | 5–7 mph |
| Medium wind (18 mph), flat board | 8–13 mph |
| Medium wind (18 mph), foiling | 18–26 mph |
| Strong wind (30 mph), foiling | 28–40 mph |

Wing foilers routinely sail **faster than the wind** — this is normal and expected. The hydrofoil eliminates most water drag, letting the wing generate far more drive than resistance.

### VMG (Velocity Made Good)

A positive VMG means you are making upwind progress. A VMG of 0 means you are going purely crosswind. A negative VMG (rare, only at very downwind angles) means you are moving away from your upwind destination.

**Rule of thumb:** your best upwind VMG usually occurs somewhere between 35°–55° angle of attack, depending on wind speed and equipment. Use WindView to find your personal sweet spot.

---

## Tips for beginners

- **Start with the defaults.** Press Play and just watch the board move. Notice how it bounces between the lines and slowly climbs toward the top of the screen.
- **Try a wider angle first.** Set angle of attack to 30°–35° and watch how fast the board goes but how slowly it makes upwind progress (low VMG). Then increase it toward 50–55° and notice VMG improves even though speed drops slightly.
- **Watch what happens near 80°.** As you approach the no-go zone, speed drops sharply and VMG collapses. This is why sailing instructors say *"if you can't feel the wind in your face, you're too close to the wind."*
- **Try switching off foiling mode.** The flat board speed drops dramatically — this is the difference foiling makes.
- **Rotate the wind clock to 45°.** Notice how the tacking lines tilt with the wind, but the shore stays fixed. This represents a wind blowing diagonally across your beach.

---

## File structure

```
windview/
├── windview.html     ← The entire app (open this in a browser)
├── README.md         ← This file — beginner guide
└── FEATURES.md       ← Technical feature and physics documentation
```

The app is entirely self-contained in a single HTML file with no external dependencies.
