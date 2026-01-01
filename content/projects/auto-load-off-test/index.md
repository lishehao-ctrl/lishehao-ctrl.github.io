---
title: "Auto Load-Off Test"
summary: "Python GUI that automates instrument-driven test runs: configures devices via SCPI/PyVISA, executes repeatable sweeps, logs results, and generates plots."
tags: ["Python", "Test Automation", "PyVISA", "SCPI", "Tkinter", "Data Logging"]
weight: 10
---

## Overview
Auto Load-Off Test is a Python-based test automation tool that replaces a repetitive “configure instruments → run sweep → record data → plot results” workflow with a single, repeatable run. It provides a GUI for configuring a test, drives lab instruments programmatically, collects measurements, and produces structured outputs for review.

## Problem
A manual test run typically involves repeatedly:
- Setting instrument parameters and keeping them consistent across runs
- Executing the same sweep procedure and timing each step correctly
- Recording measurements and organizing files for later comparison
- Regenerating plots when parameters change

This is time-consuming, easy to make inconsistent, and difficult to reproduce exactly.

## Solution
This project implements an end-to-end automation pipeline that:
- Encodes test configuration as explicit inputs (rather than “remembered” instrument settings)
- Runs sweeps automatically with deterministic step order
- Captures measurements and timestamps in a structured format
- Generates plots and saves artifacts for later analysis

## Architecture
- **GUI (Tkinter)**: test configuration, run control, and status/log display
- **Device abstraction layer**: instrument interfaces separated from the UI
- **Transport + protocol**: SCPI commands over VISA sessions (PyVISA)
- **Outputs**: logged data files and plots generated from the same captured dataset

## Tech Stack
- Python
- Tkinter
- PyVISA (VISA session management)
- SCPI (instrument command protocol)
- multi-threading (UI responsiveness during runs)
- Matplotlib (plotting)
- NumPy (data handling)

## What makes this project non-trivial
- Treating a lab “procedure” as a software pipeline: inputs → deterministic execution → reproducible outputs
- Separating UI concerns from instrument control so devices can be swapped/extended with minimal changes
- Managing long-running sweeps without losing logs/data and keeping run artifacts consistent

## Links
- GitHub: https://github.com/lishehao-ctrl/auto-load-off-test
