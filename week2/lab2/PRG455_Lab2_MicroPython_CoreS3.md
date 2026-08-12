# PRG455 — Lab 2 MicroPython on the M5Stack Core S3

**Week 2 · Student handout · Complete before your Week 3 lab**

This handout takes you from an unopened Core S3 to a device running MicroPython, connected to
your PC, streaming data over the USB serial port, and displaying text on its screen.

Everything you flash and configure here is used for **every lab, lab test, and project** in this
course. Once your device is working, **do not change the firmware version.** A device running
different firmware from the rest of the class will behave differently during a lab test, and
that is your problem to solve at 9:15 on a test morning.

**Estimated time:** 60–90 minutes.

**Prerequisite:** you must have completed the GitHub handout and the Claude Pro / VS Code
handout first. Python 3.12+ must already be installed and working on your Laptop/PC.

---

## Part 0 — What you have, and what you are installing

### The hardware

The **M5Stack Core S3** is a packaged ESP32-S3 development device:

| Feature | Detail |
|---|---|
| Processor | ESP32-S3, dual-core, with Wi-Fi and Bluetooth |
| Flash / PSRAM | 16 MB flash, 8 MB PSRAM |
| Display | 2.0" colour touch LCD, 320 × 240 |
| Sensors | 6-axis IMU (accelerometer + gyroscope), microphone, camera, light/proximity sensor |
| Power | USB-C, internal battery, managed by a power-management IC |
| Expansion | Grove Port A (I²C) on the side |
| Buttons | Power / reset button (also called **G0**) on the left side; the whole screen is a touch surface |

The USB-C port connects directly to the ESP32-S3's built-in USB hardware. There is no separate
USB-to-serial chip, which means **you do not need to install a driver** on Windows 10/11 or on
current macOS. If a guide tells you to install CP210x or CH340 drivers, that guide is for a
different M5Stack board.

### The software you are putting on it

MicroPython is a version of Python 3 that runs directly on a microcontroller, with no operating
system underneath it. There is no compiler and no linker — you copy a `.py` file onto the device
and it runs.

The firmware used in this course is **M5Stack's UIFlow2 firmware**, which *is* MicroPython, with
M5Stack's drivers for the screen, touch panel, IMU, and power management already built in. You
get a standard MicroPython REPL and a standard filesystem; you simply also get working hardware
drivers, which plain MicroPython on this board does not give you.

> **We will not use the UIFlow2 block-based web editor at any point in this course.** We are
> using the firmware only. All of your code will be Python that you write in VS Code and store
> in your Git repository, exactly like the rest of the course.

### ⚠️ The course standard firmware version

```
COURSE STANDARD FIRMWARE

UIFlow2 v2.5.0 for CoreS3

Start Mode: run main.py directly ("always")

Flashed on (date): _______________________
```

**Do not accept M5Burner's offer to update the firmware later in the term.** If your device
behaves differently from everyone else's, this is the first thing your instructor will check.

### ⚠️ The Start Mode setting is not optional

UIFlow2 firmware can boot in one of three modes. Only one of them works for this course.

| Start Mode | What the device does at boot | Use it? |
|---|---|---|
| **Run main.py directly ("always")** | Black screen, runs your `main.py`, REPL available over USB | ✅ **This is the course setting** |
| Show startup menu and network setup | Displays the UIFlow2 launcher with MAC address and a QR code | ❌ No |
| Only network setup | Connects to Wi-Fi, no launcher screen | ❌ No |

If you leave the device in the launcher mode, **the REPL will appear to connect but will ignore
everything you type.** The launcher is a MicroPython application that runs at boot and listens on
the USB connection for the UIFlow2 web editor. It owns the input stream, so your keystrokes go to
the editor protocol instead of to Python. Part 3 sets this correctly during flashing; Part 4
verifies it.

---

## Part 1 — Unbox and first power-on (10 min)

### Step 1.1 — Check what is in the box

You should have the Core S3 and a USB-C cable. Keep the box; it is the least bad place to store
the device in your bag.

### Step 1.2 — ⚠️ The cable

**Many USB-C cables are charge-only and carry no data.** This causes more lost lab time than any
other single problem on this course, because the device charges normally and looks fine while
the computer never sees it at all.

- Use the cable supplied with the device if there was one.
- If you use your own, use one that came with a phone or a drive — not a cheap charging cable
  from a discount bin.
- Plug directly into the computer. **Avoid USB hubs, docks, and monitor ports** for flashing.
  Once everything works you can experiment; not before.

### Step 1.3 — Power it on

Connect the Core S3 to your computer with the USB-C cable. It will boot into whatever demo
firmware the factory installed and display something on screen.

If nothing appears, press and hold the **power button on the left side** for about two seconds.

### Step 1.4 — Handle it properly

- Touch a metal surface before handling the device. Static discharge kills microcontrollers, and
  a dead one is not covered by anything.
- The Grove connector on the side is polarised. It fits one way. Do not force it.
- Do not short the pins on the bottom connector.
- The device has an internal lithium battery. Do not crush it, puncture it, leave it in a hot
  car, or continue using it if it swells or gets hot. Report any damage to your instructor.

---

## Part 2 — Install M5Burner (10 min)

M5Burner is M5Stack's official flashing tool. It downloads firmware and writes it to the device.

1. Go to the official M5Stack download page:
   <a href="https://docs.m5stack.com/en/download" target="_blank" rel="noopener">https://docs.m5stack.com/en/download</a>
2. Download the **M5Burner** version for your operating system.
   - **Windows and Linux:** the download is an archive. **Extract it first**, then run the
     application from the extracted folder. Running it from inside the zip fails in confusing
     ways.
   - **macOS:** drag it to Applications and launch it from there. macOS will very likely block
     it the first time — see Step 2.1 below.
3. Launch M5Burner.

### Step 2.1 — ⚠️ macOS blocks M5Burner the first time

M5Burner is not distributed through the App Store, so macOS refuses to run it until you
explicitly allow it. **The old advice of right-clicking the app and choosing "Open" no longer
works** on current versions of macOS — right-clicking now offers only *Move to Trash*, which is
not what you want.

Do this instead:

1. Double-click **M5Burner** in Applications. A dialog appears saying macOS could not verify
   that the app is free of malware.
2. **Click "Done."** Do **not** click *Move to Trash*. This first attempt is required — it is
   what tells macOS there is something to approve.
3. Open **System Settings** (Apple menu → System Settings).
4. Click **Privacy & Security** in the sidebar.
5. **Scroll down to the Security section**, near the bottom of the page. You will see a message
   naming M5Burner and saying it was blocked.
6. Click **Open Anyway**.
7. Authenticate with Touch ID, your Mac password, or your Apple Account password.
8. A final confirmation dialog appears. Click **Open Anyway** again.

M5Burner now launches, and it will launch normally every time from now on.

> **If there is no "Open Anyway" button in Privacy & Security**, macOS has forgotten the blocked
> launch — the button only appears for a short period after the attempt. Go back to step 1, try
> to open the app again, then return to System Settings immediately.

### Step 2.2 — You do not need an M5Stack account

M5Burner will offer to let you register or log in. **Skip it.** The account exists for M5Stack's
cloud features — binding a device for the web editor's Access Code workflow — and this course
does not use any of them. Flashing over USB works perfectly well while logged out.

If you register anyway, no harm done; it just is not required.

### Verification 1

M5Burner opens, you are logged in, and you can see a list of device categories in the left-hand
panel.

---

## Part 3 — Flash the firmware (20 min)

Read this entire part before you start. The **Start Mode** setting in Step 3.4 is the one that
determines whether the rest of this handout works.

### Step 3.1 — Find the firmware

1. In M5Burner's device list, select the **CoreS3** category.
2. Locate **UIFlow2 v2.5.0** — the course standard version from Part 0.
3. Click **Download** and wait for it to finish. Downloading and burning are two separate
   actions.

### Step 3.2 — Connect the device

Connect the Core S3 to your computer with the USB-C cable, directly, no hub.

M5Burner should report **"Found New Device."** If it does not, the cable is charge-only or the
port is bad — go back to Step 1.2 before doing anything else.

### Step 3.3 — Download mode is usually automatic

The ESP32-S3 must be in download mode before firmware can be written, but on the Core S3 the
chip has native USB and M5Burner can normally put it there by itself. **In most cases you do not
have to press anything.** Go straight to Step 3.4.

If the burn fails immediately, or M5Burner reports a "Get mac failed" error, put the device into
download mode by hand:

> **Long-press the G0 button (the button on the left side of the device) and keep holding it
> until the indicator changes from RED to GREEN.**

The screen stays blank in download mode. That is correct, not a fault.

### Step 3.4 — Burn, and set the Start Mode

1. In M5Burner, click **Burn** on the CoreS3 firmware entry.
2. Select the serial port for your device from the dropdown.
   - **Windows:** something like `COM5`.
   - **macOS:** something like `/dev/cu.usbmodem101`.
   - **Linux:** something like `/dev/ttyACM0`.
   - If several ports are listed, unplug the device, note which one disappears, plug it back in,
     and choose that one.
3. ⚠️ **Set the Start Mode / boot behaviour to run `main.py` directly — the "always" option.**
   This is the setting from Part 0, and it is the difference between a device you can work with
   and one that ignores everything you type. Do not choose the option that shows the UIFlow2
   startup menu.
4. M5Burner asks for **Wi-Fi SSID and password**.
   - **Do not enter Seneca's network credentials.** The campus network uses enterprise
     authentication that this device cannot use, and you should not be typing your college
     password into a firmware tool regardless.
   - Enter your home Wi-Fi, a phone hotspot, or leave it blank. In this Start Mode the device
     does not connect to a network at all, and everything in lab 2 works over the USB cable.
5. Click **Start** / **Next** and wait. Do not unplug the cable. Do not close the laptop lid.
   This takes a minute or two.
6. When it reports success, reset the device.

### Verification 2

**The device reboots to a black screen.** No UIFlow2 logo, no MAC address, no QR code, nothing.

That is exactly what success looks like in this mode — the device is running an empty `main.py`
and waiting for you. A black screen here is good news.

> **If you see the UIFlow2 startup screen instead** — a menu with a Develop tab, your device's
> MAC address, and a QR code — the Start Mode is wrong and the REPL in Part 6 will not accept
> input. Fix it now rather than later:
>
> Keep the USB cable connected, restart the device, then in M5Burner click **Configure**, select
> your port, load the current configuration, change the start mode to run `main.py` directly
> ("always"), and write it back. Reset the device and confirm you get a black screen.

---

## Part 4 — Confirm the serial port on your PC (5 min)

Your PC now needs to see the device as a serial port. This is the same connection your Tkinter
programs will use from Week 7 onward, so confirm it works now.

**Windows.** Open **Device Manager** (right-click Start → Device Manager). Expand **Ports (COM &
LPT)**. You should see a USB Serial Device with a COM number. Note the number.

**macOS.** Open Terminal:

```bash
ls /dev/cu.usbmodem*
```

Note the name — on the Core S3 it is typically `/dev/cu.usbmodem101`. **Use the `cu.` name, not
the `tty.` name** — the `tty.` device blocks waiting for a carrier signal that never arrives and
appears to hang.

**Linux.** Open a terminal:

```bash
ls /dev/ttyACM*
```

If you get a permission error later, add yourself to the `dialout` group:

```bash
sudo usermod -a -G dialout $USER
```

Then log out and back in.

### Verification 3

You can name your device's port. Write it in `lab2/SETUP.md` — you will type it repeatedly.

---

## Part 5 — Install `mpremote` on your PC (10 min)

`mpremote` is the official MicroPython command-line tool. It gives you a REPL, copies files to
and from the device, and runs scripts. It is how you will work with the device all term.

### Step 5.1 — Install

```
pip install mpremote
```

If `pip` is not recognised, try `pip3` or `python -m pip install mpremote`. On Linux you may need
`pip install --user mpremote`.

### Step 5.2 — Verify it sees the device

With the Core S3 plugged in and running normally (not in download mode):

```
mpremote devs
```

Your device's port should be listed. If the command is not recognised, close and reopen your
terminal so it picks up the new PATH.

---

## Part 6 — Your first REPL session (10 min)

The REPL is an interactive Python prompt running **on the device**. Anything you type executes on
the microcontroller.

### Step 6.1 — Connect

```
mpremote repl
```

You should get a `>>>` prompt, and typing should echo. Try `2 + 2` and press Enter.

If nothing appears at all, press `Ctrl+C` once — the device may be running a program, and
`Ctrl+C` interrupts it and drops you to the prompt.

> ### ⚠️ The prompt appears but typing does nothing
>
> This is the classic symptom of the wrong Start Mode. You see a MicroPython banner and a `>>>`
> prompt, but the keyboard does nothing at all — no echo, no response, no error.
>
> **What is happening:** the banner is real but stale. MicroPython buffers the messages printed
> while the device was booting and sends them the moment your computer connects, so you are
> reading a recording rather than talking to a live prompt. The interpreter is busy running the
> UIFlow2 launcher, which listens on the USB connection for the web editor and swallows
> everything you type.
>
> **Confirm it:**
>
> ```
> mpremote connect /dev/cu.usbmodem101 exec "print(1)"
> ```
>
> If this reboots the device or hangs instead of printing `1`, the launcher has the port.
>
> **Fix it:** the device is in the wrong Start Mode. Check the screen — if you see a UIFlow2 menu
> with a Develop tab, a MAC address, and a QR code rather than a black screen, go back to
> Verification 2 in Part 3 and use M5Burner's **Configure** option to set `main.py` to run
> directly ("always"). After the change the device boots black and the REPL responds normally.

### Step 6.2 — Prove you are talking to the device

Type these one at a time:

```python
>>> 2 + 2
```

```python
>>> import sys
>>> sys.implementation
```

This prints the MicroPython version running on your device. **Copy the exact output — you will
paste it into `lab2/SETUP.md`.**

```python
>>> import os
>>> os.listdir()
```

This lists the files in the device's filesystem. You will see `main.py` and possibly others.

```python
>>> help('modules')
```

This lists every module available in this firmware. Look for `M5` in the list — that is
M5Stack's hardware library.

### Step 6.3 — Prove it is really the hardware

```python
>>> import time
>>> time.ticks_ms()
```

Run it twice. The number increases — that is milliseconds since the device booted, counted by
the microcontroller.

### Step 6.4 — Exit

Press `Ctrl+]` to leave the REPL and return to your PC's terminal.

> **Keep this in mind all term:** only one program can hold the serial port at a time. If
> `mpremote` says the port is busy, you have another REPL, a Thonny window, or a Tkinter program
> still holding it. From Week 7 this becomes the single most common cause of "my program can't
> find the device."

### Verification 4

You reached `>>>`, ran commands, saw a version string, and exited cleanly.

---

## Part 7 — Basic test A: streaming data over serial (10 min)

This is the test that matters most for this course. Everything from Week 7 onward depends on the
device printing lines that your PC program reads.

### Step 7.1 — Create the file

In VS Code, inside your `PRG455.263` repository, create `lab2/heartbeat.py`:

```python
# PRG455 Lab 2 - serial heartbeat
# Prints one CSV line per second over the USB serial connection.

import time

count = 0

while True:
    count = count + 1
    uptime_ms = time.ticks_ms()
    print("HB,{},{}".format(count, uptime_ms))
    time.sleep(1)
```

Read it before you run it. Three things to notice:

- `print()` on a microcontroller does not go to a screen. It goes **out of the USB cable** to
  whatever is listening. That is how your Tkinter application will receive data.
- The output format is deliberate: a tag, then comma-separated fields. In Week 8 you will design
  a protocol; this is the simplest possible version of one.
- `while True:` runs forever. On a PC that would be a bug. On a microcontroller with no operating
  system, it is normal — there is nothing else for the processor to do.

### Step 7.2 — Run it on the device without installing it

```
mpremote run lab2/heartbeat.py
```

This uploads and runs the file temporarily without saving it to the device. You should see a new
line appear once per second:

```
HB,1,4213
HB,2,5215
HB,3,6217
```

Watch it for ten seconds. Confirm the counter increments and the millisecond value rises by
roughly 1000 each time.

Press `Ctrl+C` to stop it.

### Verification 5

Lines appear once per second and the numbers increase. **Copy five lines of the output into
`lab2/SETUP.md`.** This is your proof that the PC-to-device serial link works, which is the
foundation of the entire second half of this course.

---

## Part 8 — Basic test B: the display (10 min)

Now confirm the screen and the M5 hardware library.

### Step 8.1 — Create the file

Create `lab2/display_test.py`:

```python
# PRG455 lab 2 - display test

import M5
from M5 import Widgets
import time

M5.begin()

Widgets.fillScreen(0x000000)

title = Widgets.Label("PRG455.263", 10, 20, 1.0, 0xFFFFFF, 0x000000,
                      Widgets.FONTS.DejaVu24)
name = Widgets.Label("YOUR NAME HERE", 10, 60, 1.0, 0x00FF00, 0x000000,
                     Widgets.FONTS.DejaVu18)
ticker = Widgets.Label("", 10, 100, 1.0, 0xFFFF00, 0x000000,
                       Widgets.FONTS.DejaVu18)

count = 0
while True:
    M5.update()
    count = count + 1
    ticker.setText("count: {}".format(count))
    print("DISP,{}".format(count))
    time.sleep(1)
```

**Replace `YOUR NAME HERE` with your actual name.** Your instructor checks the screen at
sign-off.

### Step 8.2 — Run it

```
mpremote run lab2/display_test.py
```

The screen should show the course code, your name, and a counter that increments once per
second. `Ctrl+C` to stop.

> **If a line raises `AttributeError` or `ImportError`,** the API names in your firmware version
> differ slightly from this listing. Do not guess and do not spend twenty minutes on it. Open a
> REPL and investigate:
>
> ```python
> >>> import M5
> >>> dir(M5)
> >>> from M5 import Widgets
> >>> dir(Widgets)
> ```
>
> Then ask Claude, giving it the exact error text **and** the output of `dir()`. Record the
> problem and the fix in `PROMPTS.md` — this is precisely the kind of verification work this
> course grades.

### Step 8.3 — Notice something

`M5.update()` is called every time through the loop. It is how the library polls the touch
screen, buttons, and power chip. If you stop calling it, the hardware stops responding — while
your program keeps running perfectly happily.

Keep that in mind. Code that runs without error and hardware that behaves correctly are two
different things, and telling them apart is most of what this course is about.

### Verification 6

Your name is on the screen and the counter is incrementing. **Photograph the screen.** The
photograph goes in your repository.

---

## Part 9 — Basic test C: reading a sensor (10 min, stretch)

Optional in Week 1, but attempt it — it is your first real sensor read.

Create `lab2/imu_test.py`:

```python
# PRG455 lab 2 - IMU read

import M5
import time

M5.begin()

while True:
    M5.update()
    ax, ay, az = M5.Imu.getAccel()
    print("ACC,{:.3f},{:.3f},{:.3f}".format(ax, ay, az))
    time.sleep(0.2)
```

Run it with `mpremote run lab2/imu_test.py` and **tilt the device while it runs.** The numbers
change. One axis reads close to 1.0 when that axis points down — that is gravity, and it is the
simplest calibration check there is.

If `M5.Imu` does not exist in your firmware, use `dir(M5)` in the REPL to find the correct name
and record what you found.

---

## Part 10 — Installing a program permanently (10 min)

So far you have run programs temporarily. To make one run automatically at power-on, it must be
saved on the device as `main.py`.

### Step 10.1 — Look at what is already there

```
mpremote fs ls
```

### Step 10.2 — Back up the existing main.py

```
mpremote fs cp :main.py lab2/main_original.py
```

This copies the file **from** the device (`:main.py`) **to** your PC. Commit it — if you ever
need to get back to the factory behaviour, you will want it.

### Step 10.3 — Install your own

```
mpremote fs cp lab2/heartbeat.py :main.py
```

Unplug the device and plug it back in. It now runs your heartbeat program automatically.

### Step 10.4 — ⚠️ Getting control back

A program running in an infinite loop keeps running. To interrupt it:

```
mpremote repl
```

then press `Ctrl+C`. You are back at `>>>` and the loop has stopped.

**Learn this now, while it does not matter.** A device running a tight loop that you cannot
interrupt is genuinely frustrating during a lab test. If `Ctrl+C` does not work, hold the power
button to force the device off, then hold `Ctrl+C` in `mpremote repl` while it boots.

If everything else fails, re-flash the firmware with M5Burner. Nothing you can do in Python
permanently damages the device.

---

## Part 11 — Commit your work

Everything you created belongs in `lab2/` in your `PRG455.263` repository:

```
lab2/
├── README.md
├── SETUP.md              ← version outputs, port name, sample serial output
├── heartbeat.py
├── display_test.py
├── imu_test.py           ← if attempted
├── main_original.py      ← the factory main.py you backed up
├── screen.jpg            ← photo of your name on the device screen
└── PROMPTS.md            ← any Claude interaction during this lab
```

Add this section to `lab2/SETUP.md`:

```markdown
## Core S3 setup

**Firmware version flashed:** (UIFlow2 v2.5.0)
**Start Mode set to:** (run main.py directly / "always")
**Date flashed:**
**Serial port on my machine:**
**Operating system:**

### Output of sys.implementation in the REPL
[paste]

### Five lines of heartbeat.py output
[paste]

### Problems I hit and how I solved them
[Anything that did not work first time. Be specific — the exact error text,
 not "it didn't work".]
```

Commit and push:

```
setup: lab2 Core S3 MicroPython bring-up
```

Confirm the files are visible on github.com before you leave the lab.

---

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| macOS will not open M5Burner; right-clicking only offers "Move to Trash" | Gatekeeper has blocked it. Try to open it once, click **Done**, then System Settings → Privacy & Security → scroll to Security → **Open Anyway**. Full steps in Part 2, Step 2.1. |
| M5Burner never says "Found New Device" | Charge-only cable, in nine cases out of ten. Try a different cable, then a different USB port, and remove any hub. |
| No COM port / no `/dev/cu.usbmodem*` at all | Same cause. Cable first, always. |
| Burn fails partway through | Another program is holding the port — close every terminal, Thonny window, and VS Code serial monitor. Then re-enter download mode and retry. Lower the baud rate in M5Burner if it fails again. |
| Screen stays blank during flashing | **Correct.** Download mode blanks the screen. |
| Screen is black after a successful flash | **Correct.** In the course Start Mode the device boots black and runs `main.py`. A black screen is what success looks like. |
| Device shows a UIFlow2 menu with a MAC address and QR code | Wrong Start Mode. M5Burner → **Configure** → set `main.py` to run directly ("always"). Until you do, the REPL will ignore your keyboard. |
| **REPL prompt appears but typing does nothing** | The UIFlow2 launcher owns the USB connection and is swallowing your input. The banner you can see was buffered during boot. Fix the Start Mode — Part 6, Step 6.1. |
| `mpremote ... exec` reboots the device instead of running | Same cause. Wrong Start Mode. |
| Burn fails immediately, or "Get mac failed" | Auto-reset into download mode did not work. Long-press G0 until the indicator goes red → green, then retry. Remove any USB hub first. |
| `mpremote: no device found` | Device is in download mode, not running mode — unplug and replug. Or another program holds the port. |
| Port is busy / access denied / permission error | Something else has the port open. On Linux, add yourself to `dialout` and log out and back in. |
| REPL connects but shows nothing | Press `Ctrl+C` once. A program is running. |
| Garbled characters in the REPL | Wrong port selected, or you used the `tty.` device on macOS instead of `cu.`. |
| `AttributeError` on an `M5.` call | Firmware API differs from the listing. Use `dir(M5)` in the REPL, then ask Claude with the error text and the `dir()` output. Record it in `PROMPTS.md`. |
| Device reboots repeatedly | Usually a program crashing in a loop at startup. `mpremote repl`, `Ctrl+C` during boot, then `mpremote fs cp lab2/main_original.py :main.py` to restore. |
| Device gets warm | Normal during flashing and Wi-Fi use. Hot enough to be uncomfortable to hold is not — disconnect and report it. |
| Everything worked, then stopped after I updated the firmware | You changed the pinned version. Re-flash the course standard version from Part 0. |

**When you ask for help — in the lab or from Claude — bring the exact error text.** "It doesn't
work" cannot be diagnosed. The precise message, the command you ran, and your operating system
can be.

---

## Sign-off checklist

Bring your Core S3 and your laptop to the lab.

| # | Check | Initials |
|---|---|---|
| 1 | Device runs UIFlow2 v2.5.0, recorded in `SETUP.md` | |
| 2 | Device boots to a **black screen** — no UIFlow2 menu, MAC, or QR code | |
| 3 | Student can name their serial port and find it on their own machine | |
| 4 | `mpremote repl` reaches `>>>` **and accepts typed input**; `sys.implementation` prints a version | |
| 5 | `heartbeat.py` streams CSV lines once per second, live | |
| 6 | Device screen shows `PRG455.263` and the student's name | |
| 7 | Student demonstrates interrupting a running program with `Ctrl+C` | |
| 8 | Factory `main.py` backed up to the repository | |
| 9 | `lab2/` committed and pushed, visible on github.com | |

Checks 5 and 7 are the ones that matter. Streaming data over the cable is what the whole second
half of this course is built on, and being able to stop a runaway program is what stops a bad
lab test from becoming a failed one.
