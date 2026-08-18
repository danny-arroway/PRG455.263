# PRG455 — Event Driven Programming
## Lab 4: IMU Tilt & Calibration Panel

**Prerequisite:** Lecture 4 (Motion Sensing & the IMU). You should be
comfortable with `M5.Imu.getAccel()`, the clamp-and-scale pattern,
`M5Bar`, `M5Switch`, and the poll-driven vs. event-driven update
patterns from Lecture 4 §5 before starting this lab.

---

### Scenario

A small robotics team needs a quick diagnostic panel for checking
whether a device is sitting level before it starts a run. Sitting
still on a bench, the panel should show, at a glance, how far off
level the device currently is — and let the operator switch into a
more sensitive mode when fine-tuning the final placement. Your job is
to build that diagnostic panel as a touchscreen app running directly
on the CoreS3.

### What You're Building

A single-screen M5UI application with:

1. A **title**.
2. A **tilt bar** (`M5Bar`) that continuously reflects the device's
   current X-axis tilt, read from the IMU in `loop()`.
3. A **live value label** showing the current raw X-axis reading in g,
   updated on the same schedule as the bar.
4. A **calibration switch** (`M5Switch`) that toggles between **Normal**
   and **Sensitive** interpretation of the tilt, changing how the raw
   reading is scaled onto the bar.
5. A **mode label** that updates, on toggle, to show which mode is
   currently active.

There is no Recommend/Reset button pair in this lab — the bar and
value label update continuously and automatically, on their own,
without any button press, for as long as the program runs.

---

### Functional Requirements

**FR1 — Title.** Visible, legible, doesn't overlap other widgets.

**FR2 — Tilt bar.** An `M5Bar` with `min_value=0`, `max_value=100`,
updated every pass through `loop()` from a live IMU read — **not** from
any button or touch event. The bar's value must be produced by the
**exact clamp-and-scale formula in the Decision Logic section below**;
this is not open to interpretation, since it's what makes the lab
gradable.

**FR3 — Live value label.** An `M5Label` showing the current raw
X-axis accelerometer reading, updated on the same `loop()` pass as the
bar, formatted to exactly three decimal places with a trailing ` g`
(e.g. `X-axis: 0.482 g`). Negative readings must display their sign
(e.g. `X-axis: -0.117 g`).

**FR4 — Calibration switch.** An `M5Switch` with two states:

| State | Meaning |
|---|---|
| Off (default) | **Normal** mode — full-scale range is ±1.0 g |
| On | **Sensitive** mode — full-scale range is ±0.5 g |

Toggling the switch must immediately change which range the tilt bar
formula uses on the *next* `loop()` pass — there is no button to press
to "apply" the new mode.

**FR5 — Mode label.** An `M5Label`, next to or below the switch, that
updates the instant the switch is toggled (i.e. from the switch's
`VALUE_CHANGED` handler, not from `loop()`) to read exactly
`"Mode: Normal"` or `"Mode: Sensitive"`.

**FR6 — Touch-only operation.** The calibration switch must be usable
by touch alone on the physical CoreS3. (The bar and labels require no
touch interaction at all — that's the point of them.)

**FR7 — No blocking calls.** `loop()` must not contain `time.sleep()`
calls longer than a few milliseconds, or any other blocking operation.
The bar's live update and the switch's touch response must both stay
responsive at all times.

### Decision Logic (required — implement exactly)

This reuses the clamp-and-scale pattern from Lecture 4 §2 and §6, with
one fixed formula per mode — every possible raw X-axis reading, in
either mode, has one unambiguous correct bar value, which is what
matters for grading.

**Step 1 — Choose the half-range for the current mode:**

| Mode | Switch state | Half-range (±g) |
|---|---|---|
| Normal | Off | 1.0 |
| Sensitive | On | 0.5 |

**Step 2 — Clamp the raw X-axis reading to `[-half_range, +half_range]`.**
Readings beyond the half-range (from a hard bump or a fast motion) must
be pulled back to the nearest edge before Step 3 — an unclamped value
can otherwise scale to a bar value outside `0`–`100`.

**Step 3 — Convert the clamped reading to a 0.0–1.0 fraction:**

```
fraction = (clamped_x + half_range) / (2 * half_range)
```

**Step 4 — Bar value (integer, 0–100):**

```
bar_value = int(fraction * 100)
```

Worked check (Normal mode, half-range = 1.0): a level device reading
`ax = 0.000` must produce `fraction = 0.5` and `bar_value = 50` — the
bar's exact midpoint. A device tilted fully onto its X-axis reading
`ax = 1.000` must produce `bar_value = 100`; `ax = -1.000` must produce
`bar_value = 0`.

Worked check (Sensitive mode, half-range = 0.5): the same `ax = 0.000`
still produces `bar_value = 50`, but `ax = 0.500` now produces
`bar_value = 100` instead of requiring a full 1.0 g tilt to reach the
top of the bar — the whole point of the more sensitive mode.

### Code Requirements

- **Required header block.** Every submitted `.py` file must begin
  with the following header, as comments, with all fields filled in —
  no placeholders left in the submitted version:

  ```python
  # File:                 prg455.263.lab4.py
  # Author:               first, lastname
  # Date Submitted:       mm/dd/yyyy
  # Purpose:              solution to lab4
  # Student Number:       Seneca Polytechnic student number
  # Seneca E-mail:        Seneca student e-mail address
  # Seneca username:      Seneca My.Seneca username
  # Course Code/Section:  PRG455X
  # GitHub URL:           Student GitHub web address
  # Core S3 Device MAC:   (eg. 1CFED5BD87D2)
  ```

  Align the second field five tabs from the first, exactly as shown
  above. This is the same header required on every lab, lab test, and
  project submission this term — only the `File:` and `Purpose:` lines
  change (`prg455.263.lab4.py` / `solution to lab4`).
- Standard `setup()` / `loop()` structure, `M5.begin()` → `m5ui.init()`
  → build widgets → `page0.screen_load()`, with the try/except
  error-reporting wrapper used throughout this course.
- Use **M5UI widgets only** — labels, bar, and switch, nothing else
  needed for this lab.
- Comment your code — explain the clamp-and-scale formula, the
  Normal/Sensitive half-range logic, and which parts of your code run
  from `loop()` versus from the switch's event handler.
- If you use an AI tool anywhere in building this: **commit your own
  working code before you modify it further with AI assistance**, and
  keep `PROMPTS.md` up to date with what you asked and what you used.
- **Test with Run Once repeatedly before ever using Run Always** — a
  crashing program set to Run Always can only be recovered by
  reflashing the device through M5Burner.

---

### Testing Checklist (self-check before you submit)

Run through this on your **physical CoreS3**, not just by reading the
code:

- [ ] Required header block is present at the top of the file, every
      field filled in (no placeholders), formatted as specified in
      Code Requirements.
- [ ] Title is visible and doesn't overlap other widgets.
- [ ] With the device flat on a table (screen up), the tilt bar sits
      at or very near its midpoint (50) in Normal mode.
- [ ] Tilting the device left/right along its X-axis moves the bar
      smoothly toward 0 or 100, with no jumps, freezes, or crashes.
- [ ] The live value label updates continuously, in step with the bar,
      showing three decimal places and a correct sign.
- [ ] Toggling the calibration switch immediately changes the mode
      label text and visibly changes how far the device must tilt to
      reach the same bar position (Sensitive reaches full-scale at
      half the tilt Normal does).
- [ ] The bar keeps updating and the switch stays responsive to touch
      at the same time — no stalls, no missed touches.
- [ ] Bar value never leaves the 0-100 range, even under a hard bump
      or fast motion (clamp is working).

---

### Submission Instructions

1. Create a `lab4/` directory in your course GitHub repository (the
   private repo with `danny-arroway` added as a collaborator).
2. Place your final `.py` file in `lab4/`.
3. Commit and push before the deadline: **[INSTRUCTOR: insert due date]**.
4. If you used AI assistance, `PROMPTS.md` must be present and current
   in your repo root.

---

### Marking Rubric — Out of 100

Graded by running the submitted program on a physical CoreS3 and
working through the checklist above, cross-referencing bar values
against the Decision Logic worked checks for spot checks in both
modes.

| # | Criterion | What's checked | Points |
|---|---|---|---|
| 1 | Title | Present, legible, no overlap | 5 |
| 2 | Tilt bar present and correctly configured | `M5Bar`, range 0-100, updates every `loop()` pass from the IMU, not from a touch event | 15 |
| 3 | Clamp-and-scale formula correctness | Bar value matches the exact Decision Logic formula for arbitrary X-axis readings in both modes, including clamped edge cases | 25 |
| 4 | Live value label | Updates in step with the bar, exactly three decimal places, correct sign, correct trailing unit | 15 |
| 5 | Calibration switch and mode label | Switch present and touch-responsive; mode label updates immediately on toggle with the exact required text; half-range changes correctly between modes | 20 |
| 6 | Responsiveness | No blocking calls in `loop()`; bar keeps updating and switch stays touch-responsive simultaneously | 10 |
| 7 | Code quality | Comments explain the clamp-and-scale formula and the poll-driven vs. event-driven split; standard program structure; `PROMPTS.md` present if AI was used | 10 |
| | **Subtotal** | | **100** |
| — | **Required header block** | Deduction, not a scored criterion: **-10 points (10%) from the total above** if the required header block (see Code Requirements) is missing, incomplete, or has any placeholder text left in it | -10 |
| | **Total** | | **100** |
