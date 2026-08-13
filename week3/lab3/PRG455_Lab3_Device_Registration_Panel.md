# PRG455 — Event Driven Programming
## Lab 3: Device Registration Panel

**Prerequisite:** Lecture 3 (Basic GUI Widgets & Touch Input). You should
be comfortable with `M5Label`, `M5Button`, `M5Checkbox`-as-radio-group,
`M5TextArea` + `M5Keyboard`, and the `add_event_cb` event pattern before
starting this lab.

---

### Scenario

A facility is deploying networked IoT devices on its floor. Before a new
device can join the network, a technician must register it using a small
panel running directly on the device's own screen. You've been asked to
build that registration panel on the CoreS3.

### What You're Building

A single-screen M5UI application with:

1. A **title**, so the panel is clearly identifiable.
2. A **Name** field the technician types into, using the on-screen
   keyboard.
3. A **Device Type** selector with exactly three options — **Sensor**,
   **Actuator**, **Controller** — where only one can be selected at a
   time.
4. A **Register** button and a **Clear** button.
5. A **status area** that gives the technician feedback after each
   button press.

You are free to choose exact widget positions, colors, fonts, and text
wording (beyond what's specified below) — this spec defines *behavior*,
not pixel-perfect layout.

---

### Functional Requirements

**FR1 — Title.** On startup, the screen must display a title/heading
identifying the panel (e.g. "Device Registration"). It must be legible
and not overlap other widgets.

**FR2 — Name entry.** An `M5TextArea` for the device name, paired with an
`M5Keyboard` that:
- appears automatically when the name field is focused, and
- disappears automatically when the name field is defocused.

**FR3 — Device Type selection.** Exactly three `M5Checkbox` widgets
labeled **Sensor**, **Actuator**, and **Controller**, wired so that
checking one automatically unchecks the other two (see Lecture 3, §3.3).
It must be possible for **zero or one** of the three to be checked at any
time — never two or three simultaneously.

**FR4 — Register button.** When pressed, it must evaluate the current
form state:
- **Missing information:** if the name field is empty, or no device type
  is selected (or both), the status area must display an error message
  that makes clear what's missing, rendered in a visually distinct
  "error" color (e.g. red).
- **Complete information:** if a name has been entered *and* a device
  type is selected, the status area must display a success message that
  includes **both** the entered name and the selected device type,
  rendered in a visually distinct "success" color (e.g. green).

**FR5 — Clear button.** When pressed, it must reset the panel to its
initial state in one action:
- the name field is emptied,
- all three device type checkboxes are unchecked, and
- the status area is reset to its default (non-error, non-success) state.

**FR6 — Touch-only operation.** The entire panel must be usable by touch
alone on the physical CoreS3 — no reliance on the REPL, serial input, or
pre-set variables to exercise its behavior.

### Code Requirements

- Follow the standard `setup()` / `loop()` structure with
  `M5.begin()` → `m5ui.init()` → build widgets → `page0.screen_load()`,
  and the try/except error-reporting wrapper used throughout this course.
- Use **M5UI widgets** for all interactive elements (this lab is about
  M5UI, not raw M5GFX drawing).
- Comment your code — in particular, explain your radio-group logic and
  your Register-button validation logic in your own words.
- If you use an AI tool anywhere in building this, remember the standing
  course policy: **commit your own working code before you modify it
  further with AI assistance**, and keep your `PROMPTS.md` log up to
  date with what you asked and what you used.

---

### Suggested Approach (optional — you don't have to follow this order)

1. Get the static layout on screen first: title, name field, three
   checkboxes, two buttons, empty status label. Confirm it looks right
   before wiring up any behavior.
2. Wire up the keyboard show/hide on the name field's focus/defocus
   events.
3. Wire up the three checkboxes as a radio group, and manually test on
   the device that only one can ever be checked.
4. Implement Clear first — it's the simpler of the two buttons and gives
   you a fast way to reset state while testing Register.
5. Implement Register last, handling the missing-information case before
   the success case.

---

### Testing Checklist (self-check before you submit)

Run through this on your **physical CoreS3** — not just by reading the
code — before you commit:

- [ ] Title is visible and doesn't overlap other widgets.
- [ ] Tapping the name field shows the keyboard; tapping away hides it.
- [ ] Typed text actually appears in the name field.
- [ ] Tapping Sensor, then Actuator, then Controller — confirm only the
      most recently tapped one stays checked each time.
- [ ] Tapping the only checked box again leaves all three unchecked.
- [ ] Register with an empty name and no device type → error message.
- [ ] Register with a name but no device type → error message.
- [ ] Register with a device type but no name → error message.
- [ ] Register with both filled in → success message showing the
      correct name and correct device type.
- [ ] Clear resets the name field, unchecks all device types, and resets
      the status area — in one tap.
- [ ] Error and success messages are visually distinguishable (not just
      by text — by color).

---

### Submission Instructions

1. Create a `lab3/` directory in your course GitHub repository (the
   same private repo with `danny-arroway` added as a collaborator from
   Week 1 setup).
2. Place your final `.py` file in `lab3/`.
3. Commit and push before the deadline: **[INSTRUCTOR: insert due date]**.
4. If you used AI assistance at any point, make sure `PROMPTS.md` is
   present and up to date in your repo root, per the course AI-use
   policy.

---

### Marking Rubric — Out of 100

Graded by running the submitted program on a physical CoreS3 and working
through the checklist above. Each row is checked independently — partial
credit is available within each row for a partially-working
implementation (e.g. a radio group that mostly works but allows two boxes
checked briefly).

| # | Criterion | What's checked | Points |
|---|---|---|---|
| 1 | Title | Present, legible, doesn't overlap other widgets | 5 |
| 2 | Name entry | Text area accepts input; keyboard appears on focus and hides on defocus | 15 |
| 3 | Device Type radio behavior | Exactly 3 checkboxes; checking one always unchecks the other two; zero-selected state works correctly | 20 |
| 4 | Register button — present & wired | Button exists and its handler fires on tap | 5 |
| 5 | Clear button | Resets name field, all 3 checkboxes, and status area, correctly, in one tap | 15 |
| 6 | Validation (missing info) | Correct error message and distinct error styling for all three missing-info cases (no name / no type / neither) | 15 |
| 7 | Success behavior | Correct success message including both the entered name and selected device type, with distinct success styling | 15 |
| 8 | Code quality | Comments explaining radio-group and validation logic; standard setup/loop structure; `PROMPTS.md` present if AI was used | 10 |
| | **Total** | | **100** |

