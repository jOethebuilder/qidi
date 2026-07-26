# QIDI Box + Spoolman Integration (QIDI Plus 4 Fork)

A community-maintained fork of the original **QIDI Box + Spoolman integration project** by **Ivantify**.

This fork expands the original project by adding support for the **QIDI Plus 4** and enabling operation with **dual QIDI Boxes** for expanded filament management.

---

# Original Project

This repository is based on the original work created by **Ivantify**.

The original project provided:

- QIDI Q2 support
- Single QIDI Box integration
- Spoolman filament management support
- Klipper configuration and macros for QIDI Box operation

Original repository:

https://github.com/Ivantify/qidi

---

# Fork Maintainer

**Joe Nitta (@jOethebuilder)**

GitHub:

https://github.com/jOethebuilder

---

# About This Fork

The goal of this fork is to expand the original QIDI Box + Spoolman integration to support additional QIDI hardware configurations.

The main development focus has been adapting the original project for the **QIDI Plus 4** and adding support for running **two QIDI Boxes together**.

This fork has been developed and tested on a **QIDI Plus 4 with dual QIDI Boxes**.

---

# Changes Added In This Fork

## QIDI Plus 4 Support

Added support and configuration changes for the QIDI Plus 4, including:

- Updated configuration files for Plus 4 operation
- Updated macros required for QIDI Box communication
- Adjusted filament handling for Plus 4 hardware
- Added Plus 4 specific setup documentation
- Tested operation with Spoolman integration

---

## Dual QIDI Box Support

Added support for running **two QIDI Boxes simultaneously**.

Features:

- Expanded filament capacity from 4 positions to 8 positions
- Added configuration changes required for dual-box operation
- Updated filament selection and switching support
- Tested with:
  - QIDI Plus 4
  - Two QIDI Boxes
  - Spoolman integration

---

# Supported Printers

| Printer | Status |
|---|---|
| QIDI Q2 | ✅ Supported (original project) |
| QIDI Plus 4 | ✅ Tested with dual QIDI Boxes |
| QIDI Plus4 Max | ⚠️ Expected to work (not tested) |

Community testing and feedback are welcome.

---

# Requirements

This project requires:

- QIDI printer running Klipper
- QIDI Box hardware
- Spoolman server
- Moonraker integration
- Basic understanding of Klipper configuration

---

# Installation

Before making any changes:

**Always back up your existing printer configuration files.**

The configuration files included in this repository are intended to be adapted to your specific printer setup.

Basic installation steps:

1. Install and configure Spoolman.
2. Install the required Klipper/Moonraker configuration files.
3. Add the QIDI Box configuration.
4. Add the required macros.
5. Restart Klipper.
6. Verify communication between the printer, QIDI Box, and Spoolman.

---

# How It Works

This integration connects the QIDI Box filament system with Spoolman.

When a filament spool is selected and loaded through the QIDI Box macros, the active spool information is assigned in Spoolman.

During printing:

- The active filament spool is tracked.
- Filament usage is recorded.
- Your filament inventory stays updated.

---

# Basic Usage

## Adding Filament To Spoolman

Before printing:

1. Add your filament spools to Spoolman.
2. Enter the spool information:
   - Brand
   - Material type
   - Color
   - Weight
   - Remaining filament amount

---

## Loading Filament

Before starting a print:

1. Load filament into the desired QIDI Box slot.
2. Select the correct filament position.
3. Assign the correct spool in Spoolman.
4. Confirm the active spool.

---

## Printing

Once the filament is loaded:

- Start your print normally.
- Spoolman will track the active spool.
- Filament usage will be recorded during the print.

---

## Changing Filament

When changing filament:

1. Unload the current filament.
2. Load the new filament.
3. Update the active spool assignment.

---

# Important Usage Notes

## Clearing Active Spool After Manual Actions

Spoolman tracks the active spool based on printer macros and filament events.

If filament is changed outside of the normal print workflow, the active spool may remain assigned.

You must clear the active spool manually after:

- Manually canceling a print
- Unloading filament using the QIDI printer HMI touchscreen
- Performing manual filament changes

This keeps Spoolman accurately tracking your filament inventory.

---

# Troubleshooting

## Spoolman Shows The Wrong Active Spool

If Spoolman shows filament that is no longer loaded:

1. Verify the current filament loaded in the QIDI Box.
2. Run the clear active spool macro.
3. Assign the correct spool again.

---

## Dual QIDI Box Issues

If using two QIDI Boxes:

- Verify both boxes are configured correctly.
- Confirm each filament position is assigned correctly.
- Check that the correct macros are loaded.

---

# Testing

Current testing has been completed on:

**Printer:**
- QIDI Plus 4

**Hardware:**
- Dual QIDI Boxes

**Software:**
- Klipper
- Moonraker
- Spoolman

Additional testing from the community is welcome for other QIDI models.

---

# Credits

## Original Project

Created by:

**Ivantify**

Original contributions:

- QIDI Q2 support
- Single QIDI Box integration
- Initial Spoolman integration
- Original Klipper configuration and macros

---

## Fork Development

Additional development, testing, documentation, and modifications:

**Joe Nitta (@jOethebuilder)**

Added:

- QIDI Plus 4 support
- Dual QIDI Box support
- Expanded 8-slot filament management
- Updated configurations
- Additional documentation
- Real-world testing

---

# License

This fork retains the original **MIT License**.

All original copyright notices remain intact.
