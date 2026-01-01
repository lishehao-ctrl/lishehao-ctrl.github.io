---
title: "Auto Load-Off Test"
tags: ["Python", "Test Automation", "Hardware Validation", "SCPI"]
---

## Overview
Auto Load-Off Test is a Python-based hardware test automation tool designed to
replace repetitive manual load-off testing workflows in lab environments.

The system automates signal generation, measurement, data logging, and result
visualization, significantly improving test efficiency and consistency.

## Problem
Manual load-off testing requires repeated configuration of test instruments,
data recording, and plotting, which is:
- Time-consuming
- Error-prone
- Hard to reproduce consistently

## Solution
I designed and implemented an end-to-end automated testing pipeline that:
- Controls instruments programmatically
- Executes frequency sweeps automatically
- Captures and processes measurement data
- Generates plots and structured outputs

## Architecture
- GUI layer for test configuration (Tkinter)
- Instrument abstraction layer
- SCPI-based communication via PyVISA
- Automated data logging and visualization

## Tech Stack
- Python
- PyVISA
- SCPI
- Tkinter
- Matplotlib

## Impact
- Reduced manual test time by ~50%
- Improved repeatability of measurements
- Modular design allows easy extension for new test cases

## Links
- GitHub: https://github.com/lishehao-ctrl/auto-load-off-test
