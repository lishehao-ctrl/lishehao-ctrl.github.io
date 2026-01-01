---
title: "DingTalk To-Do Creator from Email"
date: 2025-12-31
summary: "Polls an IMAP inbox, parses ECO workflow emails, creates DingTalk To-Do tasks, and ensures idempotency."
tags: ["Python", "IMAP", "DingTalk", "Automation"]
---

## Overview
DingTalk To-Do Creator from Email is a Python utility that reads emails from an IMAP inbox, extracts a small set of ECO-related fields from matching messages, creates DingTalk To-Do tasks for configured assignees, and stores processed Message-IDs in a local JSON file to prevent duplicate task creation.

## Problem
ECO workflow emails often require follow-up actions, but manually converting qualifying emails into DingTalk To-Do items is repetitive and easy to duplicate when the mailbox is scanned repeatedly or when runs overlap.

## Solution
This project implements a small automation pipeline that:
- Pulls recent emails from IMAP within a bounded date window
- Filters messages by subject keyword and Message-ID deduplication
- Extracts required ECO fields from the message body
- Creates DingTalk To-Do tasks for configured recipients
- Persists processed state locally to keep runs idempotent

## Architecture
- IMAP fetch layer (`inbox.safe_get`)
  - Connects via `imaplib.IMAP4_SSL`
  - Uses `SINCE` / `BEFORE` search window
  - Retries up to 3 times with exponential backoff
- Parsing + filtering (`mailparser`)
  - Parses raw bytes into `EmailMessage`
  - Filters by required subject keyword and processed Message-ID state
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
- Reduces repeated manual work by generating DingTalk To-Dos directly from qualifying emails.
- Prevents duplicate task creation across repeated inbox scans via Message-ID tracking.
- Centralizes configuration and validates required inputs early to avoid silent failures.

## Links
- GitHub: https://github.com/lishehao-ctrl/mail-to-dingtalk-todo
