# Password-analysis-Tool
**Password Analysis Tool:** A cybersecurity tool that checks password strength, breach exposure, similarity, attack resistance, and reuse, then provides a **security score and improvement suggestions**.
# Password Analysis & Risk Assessment Tool

A browser-based security tool that evaluates a password across five dimensions — breach exposure, mutation similarity, composition, attack resistance, and reuse — instead of just scoring length and character variety like a typical strength meter.

Built as a single self-contained HTML/CSS/JavaScript page. No backend server, no build step, no dependencies to install.

---

## Table of Contents

- [Overview](#overview)
- [Why This Exists](#why-this-exists)
- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [How It Works](#how-it-works)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Security & Privacy Design](#security--privacy-design)
- [Scoring Model](#scoring-model)
- [Limitations](#limitations)
- [Roadmap](#roadmap)
- [License](#license)

---

## Overview

Most password checkers only measure surface complexity — is there a digit, a symbol, an uppercase letter, is it 8+ characters. That gives a false sense of security: `P@ssword123` scores well on those metrics but is trivially guessable, because it's a leetspeak mutation of the most commonly leaked password on record.

This tool instead models how attackers actually compromise accounts, and reports a **Security Score (0–100)** plus a stamped verdict — **Cleared / At Risk / Compromised** — based on:

1. Whether the password has appeared in known data breaches
2. Whether it's a disguised variant of a common password
3. Its character composition and structural weaknesses
4. How it holds up against specific attack types
5. Whether it's been reused (locally, per-user)

## Why This Exists

Built as a cybersecurity/college-style project to demonstrate:
- Safe use of a public breach-checking API (k-anonymity, zero plaintext exposure)
- String-similarity algorithms applied to a security use case
- Practical entropy/attack-resistance estimation
- Client-side cryptographic hashing via the Web Crypto API
- Privacy-respecting local persistence (no server, no accounts, no plaintext storage)

## Features

### 1. Breach Registry Check
Checks the password against the [Have I Been Pwned Pwned Passwords API](https://haveibeenpwned.com/API/v3#PwnedPasswords) using the **k-anonymity method**: only the first 5 characters of the password's SHA-1 hash are sent over the network. The full password, and even its full hash, never leave the browser.

### 2. Mutation / Similar-Password Detection
Flags passwords that are lightly disguised versions of common passwords (`P@ssword123` → `password`) using:
- A bigram **Dice coefficient** similarity score (JS equivalent of Python's `difflib.SequenceMatcher.ratio()`)
- Leetspeak normalization (`@→a`, `0→o`, `3→e`, `1→i`, `$→s`, etc.)
- Substring containment checks against a ~100-entry common-password reference list

### 3. Smart Character Suggestions
Identifies exactly what's missing — uppercase, lowercase, digits, symbols, minimum length — plus structural red flags like keyboard sequences (`qwerty`, `12345`) and repeated-character runs (`aaa`, `111`). Suggestions are specific and actionable, not generic ("add a symbol" vs. "add special characters").

### 4. Attack-Type Resistance
Scores the password against four attack categories:

| Attack Type | What's Evaluated |
|---|---|
| Brute Force | Length × character-set size → entropy → estimated crack time |
| Dictionary Attack | Presence of a known base word |
| Credential Stuffing | Breach registry exposure |
| Pattern Attack | Keyboard walks, repeated characters |

### 5. Password Reuse Detection
Since this is a client-only tool with no backend, reuse tracking uses the browser's persistent key-value storage:
- The password is hashed with **SHA-256** (never stored in plaintext)
- The hash is checked against — and then added to — a history list scoped to a user-chosen account label
- History is personal (not shared across users) and can be cleared at any time

## Architecture

```text
User enters password
        │
        ▼
 Password Analyzer (client-side JS)
        ├── HIBP Breach Check         → api.pwnedpasswords.com (k-anonymity)
        ├── Mutation Detection        → bigram similarity vs. common-password list
        ├── Smart Suggestions         → composition + pattern analysis
        ├── Attack-Type Resistance    → entropy + crack-time estimate
        └── Reuse Detection           → SHA-256 hash vs. local history
        │
        ▼
 Security Score (0–100) + Verdict + Recommendations
```

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML5, CSS3, vanilla JavaScript (no framework) |
| Cryptography | Web Crypto API (`crypto.subtle`) — SHA-1 for k-anonymity lookup, SHA-256 for reuse hashing |
| Breach data source | Have I Been Pwned Pwned Passwords API |
| Similarity analysis | Custom bigram/Dice-coefficient implementation |
| Persistence | Browser-scoped key-value storage (per-user, no server/database) |
| Fonts | Special Elite (display), IBM Plex Mono (body/data) via Google Fonts |

No npm install, no build pipeline, no external JS libraries beyond the Google Fonts stylesheet.

## How It Works

1. **Input** — User types a password (and optionally an account label) into the intake form.
2. **Local analysis runs immediately** — composition check, mutation detection, and suggestions don't require network access.
3. **Breach check** — the password is SHA-1 hashed locally; only the 5-character prefix is sent to the HIBP API; the response is matched against the full hash locally.
4. **Reuse check** — the password is SHA-256 hashed locally and compared against a stored history for that account label; the hash is recorded if new.
5. **Scoring** — all signals are combined into a 0–100 score and a Cleared/At Risk/Compromised verdict.
6. **Render** — results are displayed as a itemized report with tags, an attack-resistance table, and a stamped verdict.

## Getting Started

No installation required.

```bash
# Clone the repository
git clone https://github.com/<your-username>/password-analysis-tool.git
cd password-analysis-tool

# Open directly in a browser
open password-analysis-tool.html   # macOS
# or just double-click the file / drag it into a browser tab
```

That's it — everything runs client-side. An internet connection is only needed for the breach-registry check and for loading the Google Fonts stylesheet.

## Project Structure

```text
password-analysis-tool/
│
├── password-analysis-tool.html   # Single-file application (HTML + CSS + JS)
└── README.md                     # This file
```

> This project intentionally ships as a single file for portability. If extended into a Flask/Python backend (see [Roadmap](#roadmap)), the structure would expand to separate `analyzer/`, `templates/`, and `static/` directories.

## Security & Privacy Design

- **No plaintext password ever leaves the browser.** The only network request is a 5-character SHA-1 hash prefix sent to the HIBP API.
- **No plaintext password is ever stored**, locally or remotely. Reuse detection stores only a SHA-256 hash.
- **No accounts, no server, no third-party tracking.** The account label field is a free-text local grouping key, not an authentication system.
- **k-anonymity, not obscurity.** The breach-check method is the same one used by HIBP's own official tooling and browser extensions.
- Users can clear their locally stored reuse history at any time via the in-app control.

> **Note:** This is an educational analysis tool, not an authentication system. If adapting the reuse-detection concept for a real login system, use a proper password-hashing algorithm (Argon2id, bcrypt, or scrypt) rather than a plain SHA-256 hash, which is unsuitable for storing real account credentials.

## Scoring Model

Starting from a baseline of 100, the score is adjusted by:

| Signal | Effect |
|---|---|
| Missing uppercase / lowercase / digit | −10 each |
| Missing special character | −12 |
| Length < 8 | −30 |
| Length 8–11 | −14 |
| Length ≥ 16 | +4 |
| Found in breach data | −40 |
| Confirmed not in breach data | +4 |
| Mutation of a common password | −15 |
| Contains a keyboard/number sequence | −10 |
| Contains a repeated-character run | −8 |

Final score is clamped to 0–100 and mapped to a verdict:

- **80–100** → Cleared
- **55–79** → At Risk
- **0–54** → Compromised

## Limitations

- The common-password reference list (~100 entries) is a representative subset, not a full rockyou-scale dictionary — for production use, swap in a larger corpus.
- Crack-time estimates assume a fixed offline guessing rate (10 billion guesses/sec) as a reference point; real-world rates vary by hashing algorithm and hardware.
- The HIBP check requires network access; if unreachable, that section reports "unknown" rather than failing the whole analysis.
- Reuse history is stored in browser-scoped storage, so it doesn't follow the user across devices or browsers.

## Roadmap

- [ ] Optional Flask/Python backend for server-side reuse tracking across devices
- [ ] Configurable/expandable common-password corpus (file upload or larger bundled list)
- [ ] Chart.js visualization of score history over time
- [ ] Passphrase-specific scoring mode (e.g. for Diceware-style passwords)
- [ ] Exportable PDF audit report

## License

MIT — free to use, modify, and distribute. Attribution appreciated but not required.
