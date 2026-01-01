---
title: "Test Automation MultiTimer"
summary: "Python Tkinter GUI that automates time-bounded DMM monitoring over VISA/SCPI: configures measurement mode/range and samples on a fixed interval with logging and optional MAT export."
tags: ["Python", "Tkinter", "PyVISA", "VISA", "SCPI", "DMM", "Keysight 34461A", "MAT-File"]
weight: 40
---

## Overview
Test Automation MultiTimer is a Python desktop GUI that connects to a VISA resource (USB or LAN), configures a digital multimeter measurement mode (VOLT/CURR/RES), AC/DC (where applicable), and range, then repeatedly queries measurements on a user-defined interval until a specified monitoring duration is reached or the run is terminated.

## Problem this project solves
Manual, timed DMM monitoring is repetitive and easy to mess up: the instrument configuration and the sampling cadence must stay consistent for the whole run, and stopping at the correct duration (or stopping early) needs a clear control path.

## What the system does (bullet points)
- Lists VISA resources via PyVISA and supports manual address entry
- Supports USB resource strings or LAN IP entry (builds `TCPIP0::<ip>::INSTR`)
- Configures DMM mode, AC/DC selection, and range (including AUTO)
- Runs a timed sampling loop:
  - Issues SCPI `CONF:<mode>:<acdc>` and `MEAS:<mode>:<acdc>? <range>`
  - Records elapsed time stamps and measurement values in memory
  - Periodically triggers a save attempt (every 100 samples) and at run end
- Provides a terminal-style log view by redirecting stdout/stderr to a Tkinter text widget
- Allows terminating a run from the UI (sets an internal termination flag)

## Architecture / design (if applicable)
- **Entry point (`mm_test.py`)**
  - Starts the Tkinter application (`UI.generat_ui()` then `mainloop()`)
- **UI layer (`dmm_ui.py`)**
  - Tkinter UI for mode/AC-DC/range, sleep interval, duration unit (seconds/minutes/hours), and VISA addressing
  - Acquisition loop runs synchronously in the UI process and calls `update()` during waits to keep the interface responsive
  - Collects `time_stamps` and `power_data` arrays and builds a MAT-style payload (`time_stamps`, `power`, `configuration`)
- **Instrument layer (`dmm_driver.py`)**
  - `bATEinst_base`: PyVISA resource open/close and SCPI write/query helpers (`x_write`)
  - `instMultimeter`: SCPI-based `set_mode`, `set_range`, and `measure`
  - `instKS_34461A`: thin wrapper around `instMultimeter` with a named class for a Keysight 34461A-style device

## Key technical details (languages, frameworks, protocols, constraints)
- **Language**: Python
- **GUI**: Tkinter (`ttk`, `filedialog`, `scrolledtext`)
- **Instrument control**: PyVISA (`ResourceManager`, VISA sessions), SCPI over VISA
- **SCPI usage**:
  - Configure: `CONF:<mode>:<acdc> [<range>]`
  - Measure: `MEAS:<mode>:<acdc>? <range>`
- **Timing model**:
  - Main loop checks elapsed time against a computed runtime (based on seconds/minutes/hours)
  - Between samples, a high-frequency update loop (`update_frequency = 100`) calls `update()` and sleeps in short intervals
- **Persistence (as implemented in UI)**:
  - Builds MAT payload and conditionally calls `instKS_34461A.save_matfile(...)` if that method exists

## What makes this project non-trivial
- Coordinating instrument configuration (mode/AC-DC/range) with the correct measurement query to avoid invalid or inconsistent readings.
- Implementing a duration-bounded, interval-based sampler while still supporting user termination and UI updates during long waits.
- Managing partial-save behavior during long acquisitions (periodic save attempts) without breaking the sampling loop.

## Current status / limitations (if provided)
- The UI attempts MAT-file saving via `instKS_34461A.save_matfile(...)`, but `instKS_34461A` in `dmm_driver.py` does not define `save_matfile`; as-is, saves will be skipped unless the driver/base class is updated.
- A MAT save implementation exists in `equips_final.py` (`bATEinst_base.save_matfile` using `scipy.io.savemat`), but the current UI imports `instKS_34461A` from `dmm_driver.py`, so that implementation is not wired into the run path.
- Sampling runs in the UI thread; long runs rely on frequent `update()` calls rather than a background worker/thread, which can still impact responsiveness under load.
