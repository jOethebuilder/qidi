# QIDI Q2 / Plus4 + QIDI Box + Spoolman (Fluidd) — correct slot-based spool tracking

This update helps Spoolman track filament usage correctly on **QIDI printers with QIDI Box** by linking Spoolman spools to **physical Box slots** (Slot 1–4), not to tools (T0–T3).

**Why this matters:** on QIDI, the mapping **Tool → Box slot** can change between jobs/sessions. If you assign spools directly to `T0..T3`, your “colors” can look swapped. Slot-based assignment stays correct even when QIDI changes the tool/slot mapping behind the scenes.

✅ Works with **Qidi Studio**, **Orca Slicer**, and any other slicer.  
✅ Tested: Qidi Q2 (firmware v.1.1.0-1.1.1) + single Qidi Box  
✅ Also includes a fix for the SAVED_VARIABLES Fluidd 1.30.x bug (used by Qidi) — see https://github.com/fluidd-core/fluidd/pull/1563 — which preserves spool setup across restarts and power cycles.   
⚠️ Should also work on Plus4 (the Box logic is the same), but not tested yet.   
⚠️ Do it on your own risk and only when printer idle, and take a backup first. (You’ve been warned.)

---

## Quick start

**Good news:** with this setup, your Spoolman slot assignments are **saved properly** and are restored **correctly after a printer reboot**.

1) Open **Fluidd → Spoolman → Change Spool**  
   Assign spools to the **physical Box slots**:
    - `BOX1_SLOT1` → choose the spool physically loaded in Slot 1
    - `BOX1_SLOT2` → Slot 2
    - `BOX1_SLOT3` → Slot 3
    - `BOX1_SLOT4` → Slot 4

2) Print normally from **Qidi Studio** (no slicer/profile changes needed).  
   When the printer uses `T0/T1/T2/T3`, our macros automatically:
    - read QIDI’s current Tool → Slot mapping (`value_t0..value_t3`)
    - pick the correct **physical slot**
    - set the correct **active spool** in Spoolman  
      → Spoolman subtracts filament from the spool that’s actually printing.

3) **Important (Fluidd 1.30.x bug):**  
   Latest QIDI firmware images ship with Fluidd **1.30.x**, which has a bug where spool selections may only “save once” and then revert to old colors after the next change or reboot.  
   We **fix that bug here** (see: **“Known bug: Fluidd 1.30.x…”** section).

Result: Spoolman always tracks the correct filament, even if QIDI changes tool/slot mapping.

---

## Requirements

- You can open Fluidd and run console commands (Moonraker/Klipper).
- Spoolman is enabled in `moonraker.cfg` and visible in Fluidd — see https://moonraker.readthedocs.io/en/latest/configuration/#spoolman
- Spoolman Klipper macros exist — see https://moonraker.readthedocs.io/en/latest/configuration/#setting-the-active-spool-from-klipper
  - `SET_ACTIVE_SPOOL`
  - `CLEAR_ACTIVE_SPOOL`

If you don’t have those macros, add them at the end of the `printer.cfg`.

---

## Install

### 1) Add config files

In Fluidd → **Configuration**:

1) Create `spoolman_qidi_box1.cfg` and paste contents from this repo:  
   [`spoolman_qidi_box1.cfg`](./spoolman_qidi_box1.cfg)
2) (Optional) Open `gcode_macro.cfg` → find `M603` macro (unload filament)→ Add `CLEAR_ACTIVE_SPOOL` to the end of M603 (Put it after `M118 Unload finish`):

```ini
    M118 Unload finish
    CLEAR_ACTIVE_SPOOL
```

This prevents Spoolman from continuing to deduct from the previously active spool after an unload.

### 2) Include `spoolman_qidi_box1.cfg`  in `printer.cfg` (order matters)

Open `printer.cfg` and ensure order like this:

```ini
[include box.cfg]                 # QIDI vendor file (Box logic)
[include spoolman_qidi_box1.cfg]  # this repo (must be AFTER box.cfg)
```

### 3) Restart Klipper

In Fluidd Console:

```
RESTART
```

---

## Add Spoolman to the Fluidd dashboard

Fluidd → Dashboard → **Adjust Dashboard Layout**  
Enable widgets:
- **Spoolman**
- **Macros**

Save layout, then hard refresh the browser (**Ctrl+F5**).

Tip: Pin `CHEATSHEET_BOX1` in the Macros widget for quick access.

---


## Debug / “what is set right now?”

Run in Fluidd Console:

```
CHEATSHEET_BOX1
```

It prints:
- the spool_id assigned to each physical slot macro (`BOX1_SLOT1..4`)
- the current QIDI tool→slot map (`value_t0..value_t3`)

---

## Persistence across restarts (important)

Spool selections should survive reboot because Fluidd stores them in `saved_variables.cfg` and we restore them on startup.

If after reboot you see **old spools/colors** coming back, you likely hit the **Fluidd 1.30.x Spoolman SAVE_VARIABLES bug** described below.

---

## Known bug: Fluidd 1.30.x saves wrong variable name (spools don’t update after the first change)

### Symptoms

- The **first** time you assign a spool to `BOX1_SLOTn`, it saves.
- The **second** time you change that slot, it *looks* changed in UI, but after reboot you get the old value again.
- In Chrome DevTools → Network → WS frames you see commands like:

```
SAVE_VARIABLE VARIABLE=BOX1_SLOT3__SPOOL_ID VALUE=25
```

### Why it happens

Klipper’s `save_variables` ends up storing variable names in **lowercase** on disk.
Older Fluidd (1.30.x) saves Spoolman selections using **uppercase** (`BOX1_SLOT3__SPOOL_ID`), which breaks updates/persistence.

Newer Fluidd fixes this by saving the variable name in **lowercase**:
`box1_slot3__spool_id`  — see https://github.com/fluidd-core/fluidd/pull/1563

### Fix: patch the Fluidd bundle on the printer (works on QIDI printers)

This is the practical fix if you can’t upgrade Fluidd on your printer.

**What it does:** it edits Fluidd’s compiled JS so it saves `box1_slotX__spool_id` instead of `BOX1_SLOTX__SPOOL_ID`.

1) SSH into the printer and upload  [`patch_fluidd_spoolman_savevars.sh`](./patch_fluidd_spoolman_savevars.sh) to:
  `/home/mks/printer_data/config/patch_fluidd_spoolman_savevars.sh`

2) Run (copy-paste):

```bash
bash /home/mks/printer_data/config/patch_fluidd_spoolman_savevars.sh
```

3) In your browser open Fluidd and do a hard refresh (**Ctrl+Shift+R**).

**Revert:**

The script creates a backup folder `~/fluidd.bak_YYYY-MM-DD_HHMMSS`. Within the same SSH session, the easy way to revert is to run:

```bash
rm -rf ${FLUIDD_DIR} && cp -a ${BACKUP_DIR} ${FLUIDD_DIR}"
```

---

## Safety notes

- Avoid manually editing `saved_variables.cfg` while the printer is running. Do it only when idle and after a backup.
- If you really want to do dangerous stuff, you can — just don’t be surprised when it bites you.

---

## Multi-box future (not tested)

This repo is structured to scale:

- `spoolman_qidi_box1.cfg`
- `spoolman_qidi_box2.cfg`(READY)
- `spoolman_qidi_box2.cfg`(future)
- etc.

Pattern:
- `BOX2_SLOT1..4` represent Box2 physical slots
- Tools `T4..T7` map via `value_t4..value_t7`

---
## Dual-Box (Plus4) Notes

This was originally built and tested for a single Box on a Q2. If you're 
running two QIDI Boxes on a Plus4 (or any dual-box setup), be aware of 
the following differences:

### 1. Tools are NOT fixed to a specific box
QIDI dynamically maps any tool number (T0-T7) to any physical slot across 
BOTH boxes, based on what's loaded and the sync step when you send a print. 
Do not assume T0-T3 = Box1 and T4-T7 = Box2 — check `value_t0`..`value_t7` 
in `saved_variables.cfg` to see the live mapping at any given time.

Because of this, **every T0-T7 macro in both spoolman_qidi_box1.cfg and 
spoolman_qidi_box2.cfg must check all 8 possible slots** (slot0-7), not 
just their "own" 4. The versions in this repo have been updated accordingly.

### 2. Active Spool doesn't auto-populate at print start by default
QIDI's own `PRINT_START` macro calls `BOX_PRINT_START` directly (its 
internal load routine), which bypasses our T0-T7 overrides entirely. This 
means Spoolman's Active Spool stays blank until a tool *change* happens 
mid-print — the very first tool of a print never triggers it.

**Fix:** add the `ACTIVATE_SPOOL_FOR_TOOL` macro (included at the end of 
`spoolman_qidi_box1.cfg`) — it sets the active spool without touching the 
physical loader, so it's safe to call before printing starts. Then, in 
your slicer's **Start G-code**, add this line before the `PRINT_START` line:

    ACTIVATE_SPOOL_FOR_TOOL T=[initial_tool]

This ensures Spoolman knows the correct spool *before* the initial 
purge/flush happens, so that filament usage is correctly counted too.

### 3. Unloading via Fluidd's Box Control panel does not clear Active Spool
Neither `M603` nor QIDI's `UNLOAD_FILAMENT`/`UNLOAD_Tn` macros call 
`CLEAR_ACTIVE_SPOOL`. This means Active Spool can show a stale spool_id 
after an unload — but it self-corrects the moment the next print's tool 
change fires. If you want it to clear immediately on unload instead, add 
`CLEAR_ACTIVE_SPOOL` to the relevant macro in `box.cfg` (vendor file, not 
covered by this repo).
