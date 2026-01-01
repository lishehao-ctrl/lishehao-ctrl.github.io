---
title: "DingTalk To-Do Creator from Email"
tags: ["Python", "IMAP", "Email Parsing", "DingTalk", "OAuth2", "AlibabaCloud SDK", "dotenv", "JSON"]
---

## Overview
DingTalk To-Do Creator from Email polls an IMAP inbox, filters business emails by subject and date rules, extracts a small set of ECO fields from the message body, creates DingTalk To-Do tasks for configured recipients, and records processed `Message-ID`s in a local JSON file to keep runs idempotent.

## Problem
Manually converting qualifying ECO workflow emails into DingTalk To-Do tasks is repetitive and commonly leads to:
- Duplicate task creation across repeated mailbox scans
- Missed follow-ups when emails arrive in bursts
- Inconsistent extraction of key fields from email bodies

## Solution
This project implements a minimal automation pipeline that:
- Connects to an IMAP mailbox over SSL and fetches recent emails within a bounded date window
- Deduplicates using `Message-ID` persisted in `processed_messages.json`
- Filters by a required subject keyword (`ECO审批流程`) and “creation date” (sent date + configured offsets) not being in the future
- Extracts fields from the email body using regex:
  - `ecn编码`, `ecn名称`, `产品名称`, `工作负责人`
- Creates a DingTalk To-Do per configured recipient
- Sends a separate error To-Do to a configured error recipient list if the script fails

## Architecture
- `main.py`
  - Orchestrates fetch → parse/filter → extract → create To-Do → persist state
- `inbox.py`
  - `safe_get()` IMAP fetch with SSL and up to 3 retries (exponential backoff)
  - Uses `SINCE`/`BEFORE` date search with a configurable search window (`mapping_search_window`)
- `mailparser.py`
  - Parses raw bytes into `EmailMessage`
  - Applies filtering rules (dedup, subject keyword, date constraints)
  - Extracts header fields and body fields (text/plain or text/html → text via BeautifulSoup)
- `dingtalk.py`
  - Fetches app `access_token` via DingTalk OAuth2 SDK (appKey/appSecret)
  - Resolves `userId -> unionId` via `topapi/v2/user/get`
  - Creates To-Do tasks via DingTalk To-Do SDK; retries once on failure
- `state.py`
  - Reads/writes `processed_messages.json` (creates empty state if missing/invalid)
  - Uses a relative-path helper to work in normal execution and frozen builds
- `mapping.py` + `dingtalk_recipients.py`
  - Loads env/config and validates required inputs early (`.env`, `config/dingtalk_recipients.json`)

Known code issue (as-is in the current sources):
- In `dingtalk.py`, the nested `cal_due_time()` is defined without parameters but is called with an argument when building the To-Do request, which will raise a `TypeError` unless corrected.

## Tech Stack
- Python
- IMAP over SSL (`imaplib`, `ssl`)
- Email parsing (`email`, `BytesParser`, `parsedate_to_datetime`)
- HTML-to-text (`beautifulsoup4`)
- Time offsets (`python-dateutil` / `relativedelta`)
- Config loading (`python-dotenv`)
- DingTalk integration:
  - `alibabacloud-dingtalk` (OAuth2 + To-Do API)
  - `requests` (user lookup endpoint)

## Impact
- Automates creation of DingTalk To-Dos from qualifying emails.
- Prevents duplicate processing via persisted `Message-ID` state.
- Centralizes configuration and fails fast on missing/invalid inputs.

## Links
- GitHub: https://github.com/lishehao-ctrl/test-automation-multitimer
