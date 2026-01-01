---
title: "DingTalk To-Do Creator from Email"
summary: "Converts structured workflow emails into actionable tasks in an enterprise collaboration platform by polling IMAP, extracting ECO fields, creating To-Dos, and enforcing idempotency."
tags: ["Python", "IMAP", "Email Parsing", "Workflow Automation", "DingTalk"]
weight: 20
---

## Overview
DingTalk To-Do Creator from Email is a Python automation utility that monitors an IMAP mailbox for ECO workflow emails and converts them into assigned To-Do items in an enterprise collaboration platform (DingTalk). It extracts a small set of structured fields from each qualifying message, creates a task for configured assignees, and records processed `Message-ID`s to prevent duplicates across repeated scans.

## Problem
Operational workflow emails often encode “work to be done,” but that work stays trapped in the inbox. In practice this leads to:
- Manual copy/paste of key fields into a task system
- Duplicate task creation when the mailbox is scanned repeatedly or runs overlap
- Inconsistent extraction of required identifiers and owners from email bodies

## Solution
This project implements a small email-to-task pipeline that:
- Pulls recent emails from IMAP within a bounded date window
- Filters messages by subject keyword and `Message-ID` deduplication
- Extracts required ECO fields from the message body
- Creates To-Do tasks for configured recipients
- Persists processed state locally to keep runs idempotent

## Architecture
- IMAP fetch layer (`inbox.safe_get`)
  - Connects via `imaplib.IMAP4_SSL`
  - Uses `SINCE` / `BEFORE` search window
  - Retries up to 3 times with exponential backoff
- Parsing + filtering (`mailparser`)
  - Parses raw bytes into `EmailMessage`
  - Filters by required subject keyword and processed `Message-ID` state
  - Extracts body fields via regex from `text/plain` or `text/html` (HTML is converted to text via BeautifulSoup)
- DingTalk integration (`dingtalk`)
  - Retrieves app access token via Alibaba Cloud DingTalk OAuth2 SDK
  - Resolves `userId -> unionId` via DingTalk OpenAPI
  - Creates To-Do tasks via Alibaba Cloud DingTalk To-Do SDK (single retry to reduce transient failures)
- Local state (`state`)
  - Reads/writes `processed_messages.json` (created if missing/invalid)
- Configuration (`.env`, `config/dingtalk_recipients.json`)
  - Secrets and recipient lists are not tracked; example templates are provided

## Tech Stack
- Python
- IMAP (imaplib over SSL)
- Email parsing (Python `email` package)
- Requests (DingTalk OpenAPI calls)
- BeautifulSoup4 (HTML-to-text)
- python-dateutil (relative time offsets)
- python-dotenv (environment config)
- Alibaba Cloud DingTalk SDKs:
  - OAuth2 (`alibabacloud-dingtalk`)
  - To-Do API (`alibabacloud-dingtalk`)

## Impact
- Converts inbox-based workflow notifications into assigned tasks with consistent field extraction.
- Prevents duplicate task creation across repeated inbox scans via `Message-ID` tracking.
- Centralizes configuration and validates required inputs early to avoid silent failures.

## Links
- GitHub: https://github.com/lishehao-ctrl/mail-to-dingtalk-todo
