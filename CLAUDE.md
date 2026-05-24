# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the app

Open `nfe-sender.html` directly in a browser — no build step, server, or dependencies required. All logic is self-contained in that single file.

## Architecture

`nfe-sender.html` is a single-file vanilla JS app (no frameworks, no external libraries) with three integrated systems:

**OpenAI Vision extraction** — on file upload, calls `gpt-4o` with the image as base64 to extract `date` and `amount` from the payment receipt. The extracted amount is used as the source of truth to trim/adjust the Friday list (`days` array).

**Gmail OAuth implicit flow** — the app uses Google's OAuth 2.0 `response_type=token` redirect. The page redirects to Google, then Google redirects back with an `access_token` in the URL hash. `handleGoogleRedirect()` captures it on load and strips it from the URL. The token is persisted in `localStorage` and reused until near-expiry. Sending uses the Gmail REST API directly (`/v1/users/me/messages/send`) with a hand-built MIME message encoded in base64url.

**Friday/holiday calculator** — `calcFridays()` computes all Fridays in the selected month, then cross-references against Brazilian public holidays (fixed + moveable: Easter, Good Friday, Carnival Mon/Tue, Corpus Christi). Holidays fall on a Friday get bumped to the next non-holiday Friday, unless that date would duplicate an existing scheduled Friday.

## Key state

- `days` — array of `{ date, moved, original }` objects representing scheduled session dates; drives the chips UI, email body, and total amount.
- `DEF.price` — hardcoded session price (R$ 620); used to infer expected session count from receipt amount.
- All credentials (OpenAI key, Google Client ID, Google token) live only in `localStorage`; nothing is sent to any backend.

## Reference month logic

The reference month for the NFE is always the month *prior* to the payment date extracted from the receipt. When AI extraction sets `extracted.date`, the month selector is automatically set to `date - 1 month`.
