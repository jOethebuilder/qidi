# QIDI Box + Spoolman Integration
## QIDI Plus 4 / Dual QIDI Box Community Fork

A community-maintained fork of the original **QIDI Box + Spoolman integration project** by **Ivantify**.

This fork expands the original project by adding support for the **QIDI Plus 4** and enabling operation with **dual QIDI Boxes** for expanded filament management.

---

# Original Project

This repository is based on the original work created by **Ivantify**.

The original project provided:

- QIDI Q2 support
- Single QIDI Box integration
- Spoolman filament management
- Klipper configuration and macros for QIDI Box operation

Original project:

https://github.com/Ivantify/qidi/tree/main/qidi-box-spoolman

---

# Fork Maintainer

**jOethebuilder**

Forked from:

https://github.com/jOethebuilder/qidi/tree/main/qidi-box-spoolman

---

# About This Fork

The purpose of this fork is to expand the original QIDI Box + Spoolman integration for additional QIDI hardware configurations.

The main development focus has been adapting the original project for:

- QIDI Plus 4
- Dual QIDI Boxes
- 8 filament position management

This fork has been developed and tested on:

- QIDI Plus 4
- Two QIDI Boxes
- Spoolman
- Fluidd
- Moonraker

---

# Changes Added In This Fork

## QIDI Plus 4 Support

Added support and configuration changes for the QIDI Plus 4:

- Updated configuration files
- Updated QIDI Box integration
- Updated macros
- Added Plus 4 setup instructions
- Tested operation with Spoolman

---

## Dual QIDI Box Support

Added support for running two QIDI Boxes together.

Features:

- Expanded filament management from 4 positions to 8 positions
- Added Box 2 configuration
- Added Box 2 spool tracking
- Updated tool-to-slot mapping
- Tested with:
  - QIDI Plus 4
  - Dual QIDI Boxes
  - Spoolman

---

# Supported Printers

| Printer | Status |
|---|---|
| QIDI Q2 | Supported (original project) |
| QIDI Plus 4 | Tested |
| QIDI Plus 4 + Dual QIDI Boxes | Tested |
| QIDI Plus4 Max | Expected to work (not tested) |

Community testing and feedback are welcome.

---

# Requirements

Before installing:

- QIDI printer running Klipper
- QIDI Box hardware
- Moonraker installed
- Spoolman installed
- Fluidd access
- Ability to edit configuration files

---

# Installation

## Before Starting

**Always backup your existing printer configuration files.**

Do not overwrite your complete `printer.cfg`.

The files in this repository should be merged with your existing QIDI configuration.

---

# Add Configuration Files

In Fluidd:

1. Open:

```
Configuration
```

2. Click:

```
Create File
```

This creates a new Klipper configuration file in your printer configuration folder.

---

## Single QIDI Box Setup

Create a new file:

```
spoolman_qidi_box1.cfg
```

Copy the contents from this repository into the file.

Save the file.

---

## Dual QIDI Box Setup

If you are using a second QIDI Box, or adding one later, create:

```
spoolman_qidi_box2.cfg
```

Copy the Box 2 configuration from this repository.

Save the file.

---

# Add Includes To printer.cfg

The include order is important.

`box.cfg` must load before the Spoolman integration files.

## Single QIDI Box

```ini
[include box.cfg]

[include spoolman_qidi_box1.cfg]
```

---

## Dual QIDI Box

```ini
[include box.cfg]

[include spoolman_qidi_box1.cfg]

[include spoolman_qidi_box2.cfg]
```

Only add:

```ini
[include spoolman_qidi_box2.cfg]
```

if you are using a second QIDI Box or adding one later.

---

# Restart Klipper

After saving changes:

Open Fluidd Console:

```
RESTART
```

Check that Klipper starts without configuration errors.

---
# Verify Spoolman Macros

Before continuing, verify that the required Spoolman macros exist.

Open:

```
Fluidd → Console
```

Run:

```
HELP
```

Verify these macros are available:

```
SET_ACTIVE_SPOOL
CLEAR_ACTIVE_SPOOL
ACTIVATE_SPOOL_FOR_TOOL
```

If these macros do not exist:

1. Check your existing configuration files for the Spoolman macro definitions.

Common locations:

```
printer.cfg
macros.cfg
spoolman.cfg
```

2. If they are missing, add the required Spoolman Klipper macros from the Spoolman documentation:

https://github.com/Donkie/Spoolman/blob/master/docs/klipper.md

3. Restart Klipper after adding the macros.

In Fluidd Console:

```
RESTART
```

---

# ⚠️ Required Slicer Start G-Code Setup

A slicer start G-code change is required for the first filament used in a print to be tracked correctly.

Add:

```gcode
ACTIVATE_SPOOL_FOR_TOOL T=[initial_tool]
```

Place this line before:

```gcode
T[initial_tool]
```

Example:

```gcode
ACTIVATE_SPOOL_FOR_TOOL T=[initial_tool]

PRINT_START BED=[bed_temperature_initial_layer_single] HOTEND=[nozzle_temperature_initial_layer] CHAMBER=[chamber_temperatures] EXTRUDER=[initial_tool]

SET_PRINT_STATS_INFO TOTAL_LAYER=[total_layer_count]

M83

T[initial_tool]
```

**Do not replace the complete QIDI start G-code.**

Only add the `ACTIVATE_SPOOL_FOR_TOOL` line to your existing QIDI start sequence.

---

# Known Issue: Fluidd 1.30.x Spoolman Save Bug

Some QIDI firmware versions ship with Fluidd 1.30.x.

There is a known issue where Spoolman slot assignments may appear to save correctly but do not persist after reboot.

## Symptoms

- The first spool assignment works correctly.
- Changing a spool assignment appears correct in Fluidd.
- After restarting the printer, the previous spool assignment returns.

Example:

You change:

```
BOX1_SLOT3
```

to a different spool.

Fluidd shows the new spool.

After reboot, the old spool appears again.

---

## Cause

Fluidd 1.30.x saves the variable name using uppercase letters:

```
BOX1_SLOT3__SPOOL_ID
```

Klipper's `save_variables` system stores variable names in lowercase:

```
box1_slot3__spool_id
```

Because of this mismatch, spool changes may not restore correctly after reboot.

---

## Fix

A patch script is included to correct the Fluidd behavior.

Upload:

```
patch_fluidd_spoolman_savevars.sh
```

to:

```
/home/mks/printer_data/config/
```

Run:

```bash
bash /home/mks/printer_data/config/patch_fluidd_spoolman_savevars.sh
```

After the patch completes, refresh Fluidd:

```
CTRL + SHIFT + R
```

The patch creates a backup before making changes.

---

# How It Works

This integration connects the QIDI Box filament system with Spoolman.

The QIDI Box manages the physical filament locations.

Spoolman manages:

- Filament inventory
- Remaining filament
- Filament usage tracking
- Active spool information

The process works like this:

```
Physical QIDI Box Slot
          ↓
QIDI Tool Selection
          ↓
Tool To Slot Mapping
          ↓
Spoolman Active Spool
```

During printing:

- The active filament spool is selected automatically.
- Filament usage is recorded.
- Spoolman subtracts filament from the spool actually being used.
- Your filament inventory stays updated.

---

# Basic Usage

## Adding Filament To Spoolman

Before printing:

Add your filament spools to Spoolman.

Enter:

- Brand
- Material type
- Color
- Filament diameter
- Original spool weight
- Remaining filament amount

Each physical spool should have its own entry.

---
# Assigning Spools To QIDI Box Slots

Open:

```
Fluidd → Spoolman → Change Spool
```

Assign the spools to match the physical QIDI Box slots.

Example:

```
BOX1_SLOT1 → Physical Slot 1
BOX1_SLOT2 → Physical Slot 2
BOX1_SLOT3 → Physical Slot 3
BOX1_SLOT4 → Physical Slot 4
```

For dual QIDI Boxes:

```
BOX2_SLOT1 → Physical Slot 1
BOX2_SLOT2 → Physical Slot 2
BOX2_SLOT3 → Physical Slot 3
BOX2_SLOT4 → Physical Slot 4
```

The Spoolman assignment must match the filament physically loaded in each slot.

---

# Printing

Print normally from:

```
QIDI Studio
```

No slicer profile changes are required.

When the printer uses:

```
T0
T1
T2
T3
```

the macros:

- Read the current QIDI tool mapping.
- Determine the physical QIDI Box slot.
- Select the correct spool.
- Set the active spool in Spoolman.

Spoolman will then track filament usage from the spool actually printing.

---

# Changing Filament

When changing filament:

1. Unload the current filament.
2. Load the new filament.
3. Update the spool assignment in Spoolman if needed.
4. Verify the correct spool is active.

---

# Important Usage Notes

## Clearing Active Spool After Manual Actions

The system normally handles active spool tracking automatically.

You should clear the active spool if filament is changed outside the normal workflow.

Run:

```
CLEAR_ACTIVE_SPOOL
```

after:

- Manually canceling a print.
- Unloading filament using the QIDI printer HMI touchscreen.
- Manually changing filament without using the normal macros.

This prevents Spoolman from continuing to subtract filament from the wrong spool.

---

# Optional Unload Macro Update

Adding `CLEAR_ACTIVE_SPOOL` to the unload macro is optional.

If your workflow requires it, add:

```ini
M118 Unload finish
CLEAR_ACTIVE_SPOOL
```

to the end of your unload macro.

Most users will not need this change.

---

# Add Spoolman To The Fluidd Dashboard

In Fluidd:

Go to:

```
Dashboard → Adjust Dashboard Layout
```

Enable:

- Spoolman widget
- Macros widget

Save the layout.

Refresh Fluidd:

```
CTRL + F5
```

Recommended macros to pin:

```
CHEATSHEET_BOX1
CHEATSHEET_BOX2
```

---

# Dual QIDI Box Verification

If using two QIDI Boxes, verify both configurations.

Open:

```
Fluidd → Console
```

Run:

```
CHEATSHEET_BOX1
```

and:

```
CHEATSHEET_BOX2
```

These commands show:

- Slot assignments
- Spool IDs
- Current tool-to-slot mapping

Example:

```
BOX1_SLOT1
BOX1_SLOT2
BOX1_SLOT3
BOX1_SLOT4

BOX2_SLOT1
BOX2_SLOT2
BOX2_SLOT3
BOX2_SLOT4
```

All eight positions should show the correct spool IDs.

---

# Debug / Current Settings

To see what is currently assigned:

Run:

```
CHEATSHEET_BOX1
```

This displays:

- The `spool_id` assigned to each physical slot:
  - `BOX1_SLOT1`
  - `BOX1_SLOT2`
  - `BOX1_SLOT3`
  - `BOX1_SLOT4`

- The current QIDI tool-to-slot mapping:
  - `value_t0`
  - `value_t1`
  - `value_t2`
  - `value_t3`

For dual QIDI Boxes:

Run:

```
CHEATSHEET_BOX2
```

This displays the same information for:

- `BOX2_SLOT1`
- `BOX2_SLOT2`
- `BOX2_SLOT3`
- `BOX2_SLOT4`

---

# Persistence Across Restarts

Spool selections are stored using:

```
saved_variables.cfg
```

Assignments should survive printer restarts.

If old spool assignments or colors return after reboot, see:

```
Known Issue: Fluidd 1.30.x Spoolman Save Bug
```

---

# Credits

## Original Project

Created by:

**Ivantify**

Original contributions:

- QIDI Q2 support
- Single QIDI Box support
- Initial Spoolman integration
- Original macros and configuration

Original project:

https://github.com/Ivantify/qidi/tree/main/qidi-box-spoolman

---

## Fork Development

Maintained by:

**jOethebuilder**

Added:

- QIDI Plus 4 support
- Dual QIDI Box support
- 8-slot filament management
- Configuration updates
- Testing
- Documentation

---

# License

This fork retains the original MIT License.

All original copyright notices remain intact.
