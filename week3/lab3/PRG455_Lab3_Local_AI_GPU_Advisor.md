# PRG455 — Event Driven Programming
## Lab 3: Local AI GPU Advisor

**Prerequisite:** Lecture 3 (Basic GUI Widgets & Touch Input). You should
be comfortable with `M5Label`, `M5Button`, `M5Slider`, the
`VALUE_CHANGED` event pattern, and the text-wrapping technique from
Lecture 3 §4 before starting this lab.

---

### Scenario

A small AI lab is helping researchers pick which GPU to buy for running
local Qwen models on their own hardware, based on the model they plan
to run, how it's quantized, and how much context window they need. Your
job is to build that advisor as a touchscreen panel running directly on
the CoreS3.

### What You're Building

A single-screen M5UI application with:

1. A **title**.
2. Three **sliders**, each paired with a label that shows the currently
   selected value in readable form:
   - **Model** — one of six Qwen models (see table below).
   - **Quantization** — 4-bit (partial) or 16-bit (full).
   - **Context window** — 8K tokens or 256K tokens.
3. A **Recommend** button that computes a GPU recommendation from the
   three current slider values and displays it.
4. A **Reset** button that returns all three sliders (and their labels)
   to their starting position and clears the recommendation.

---

### Functional Requirements

**FR1 — Title.** Visible, legible, doesn't overlap other widgets.

**FR2 — Model slider.** An `M5Slider` with 6 positions (`min_value=0`,
`max_value=5`), each position corresponding to one of these models in
this order:

| Position | Model |
|---|---|
| 0 | Qwen2-VL-2B |
| 1 | Qwen2.5-3B |
| 2 | Qwen3-4B |
| 3 | Qwen2.5-7B |
| 4 | Qwen2.5-14B |
| 5 | Qwen2.5-32B |

A label next to or below the slider must update live, as the slider is
dragged, to show the current model's name (e.g. "Model: Qwen2.5-7B").

**FR3 — Quantization slider.** An `M5Slider` with 2 positions
(`min_value=0`, `max_value=1`): position 0 = "4-bit", position 1 =
"16-bit". A label must update live to show the current selection.

**FR4 — Context window slider.** An `M5Slider` with 2 positions
(`min_value=0`, `max_value=1`): position 0 = "8K tokens", position 1 =
"256K tokens". A label must update live to show the current selection.

**FR5 — Recommend button.** On press, it must:
1. Read the current position of all three sliders.
2. Compute the estimated VRAM requirement using the **exact formula in
   the Decision Logic section below** — this is not open to
   interpretation, since it's what makes the lab gradable.
3. Display the matching recommendation string (one of the four listed
   in Decision Logic) in the result area.
4. **Wrap the displayed text** using the technique from Lecture 3 §4 —
   none of the four possible recommendation strings fit on one line at
   a readable font size, and the result area must be sized with enough
   vertical room for the wrapped text, entirely within the visible
   240px screen height. No part of the result may be cut off or run
   past the bottom edge of the screen.

**FR6 — Reset button.** On press, it must, in one action:
- return all three sliders to position 0,
- update all three labels back to their position-0 text
  ("Model: Qwen2-VL-2B", "Quantization: 4-bit", "Context: 8K tokens"),
  and
- clear the result area completely.

**FR7 — Touch-only operation.** The entire panel must be usable by
touch alone on the physical CoreS3.

### Decision Logic (required — implement exactly)

This is a simplified estimation model for teaching purposes, not a
precise real-world VRAM calculator (actual VRAM use also depends on
model architecture details like attention head count, which vary
per-model in ways this simplified formula doesn't capture) — but it's
internally consistent and every input combination has one unambiguous
correct answer, which is what matters for grading.

**Step 1 — Base VRAM (GB), from model parameter count at 16-bit:**

| Model | Parameters (B) | Base VRAM (params × 2) |
|---|---|---|
| Qwen2-VL-2B | 2 | 4 GB |
| Qwen2.5-3B | 3 | 6 GB |
| Qwen3-4B | 4 | 8 GB |
| Qwen2.5-7B | 7 | 14 GB |
| Qwen2.5-14B | 14 | 28 GB |
| Qwen2.5-32B | 32 | 64 GB |

**Step 2 — Quantization multiplier:** 4-bit → × 0.25, 16-bit → × 1.0

**Step 3 — Context window addition:** 8K → + 1 GB, 256K → + 16 GB

**Step 4 — Estimated VRAM (GB) = (Base VRAM × Quantization multiplier) + Context addition**

**Step 5 — GPU selection**, smallest tier that's large enough:

| If estimated VRAM is... | Recommendation string to display |
|---|---|
| ≤ 16 GB | `16GB, GeForce RTX 5080, NVIDIA, $1,200 - $1,450` |
| ≤ 24 GB | `24GB, GeForce RTX 4090, NVIDIA, $1,850 - $2,100` |
| ≤ 32 GB | `32GB, GeForce RTX 5090, NVIDIA, $2,000 - $2,900+` |
| > 32 GB | `64GB+, Radeon Pro W7900 / RTX 6000 Ada, AMD / NVIDIA, $3,500 - $7,000+` |

Use a plain hyphen (`-`), not an en dash, in the displayed strings —
CoreS3's built-in fonts only cover ASCII 32–126, and an en dash will
render as a box rather than the character you intended.

### Code Requirements

- Standard `setup()` / `loop()` structure, `M5.begin()` → `m5ui.init()`
  → build widgets → `page0.screen_load()`, with the try/except
  error-reporting wrapper used throughout this course.
- Use **M5UI widgets only** — labels, buttons, and sliders, nothing
  else needed for this lab.
- Comment your code — explain the VRAM formula and the slider-to-label
  mapping logic in your own words.
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

- [ ] Title is visible and doesn't overlap other widgets.
- [ ] Dragging the Model slider updates its label correctly at all 6
      positions.
- [ ] Dragging the Quantization slider updates its label correctly at
      both positions.
- [ ] Dragging the Context slider updates its label correctly at both
      positions.
- [ ] Recommend produces the correct GPU string for at least 4
      different slider combinations, cross-checked against the
      Decision Logic appendix table.
- [ ] The full recommendation text is visible on screen with no part
      cut off at the bottom edge, for the longest possible string
      (the 64GB+ option).
- [ ] Reset returns all three sliders and labels to their defaults and
      clears the result, in one tap.

---

### Submission Instructions

1. Create a `lab3/` directory in your course GitHub repository (the
   private repo with `danny-arroway` added as a collaborator).
2. Place your final `.py` file in `lab3/`.
3. Commit and push before the deadline: **[INSTRUCTOR: insert due date]**.
4. If you used AI assistance, `PROMPTS.md` must be present and current
   in your repo root.

---

### Marking Rubric — Out of 100

Graded by running the submitted program on a physical CoreS3 and
working through the checklist above, cross-referencing outputs against
the appendix decision table for spot checks.

| # | Criterion | What's checked | Points |
|---|---|---|---|
| 1 | Title | Present, legible, no overlap | 5 |
| 2 | Three sliders present with correct ranges | Model (0-5), Quantization (0-1), Context (0-1) | 15 |
| 3 | Live label updates | All three labels correctly reflect their slider's current position at every valid value | 20 |
| 4 | VRAM formula correctness | Recommend produces the exact correct output for arbitrary slider combinations, matching the Decision Logic table | 25 |
| 5 | Result display | Full text visible, correctly wrapped, no clipping at the screen edge | 10 |
| 6 | Reset button | All three sliders, all three labels, and the result area correctly reset in one tap | 15 |
| 7 | Code quality | Comments explain the formula and slider-label logic; standard program structure; `PROMPTS.md` present if AI was used | 10 |
| | **Total** | | **100** |

