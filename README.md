# Adaptive Continuous Authentication via Keystroke Dynamics

A behavioral "digital twin" of a user's typing patterns that continuously re-verifies identity *during* a session — not just at login.

---

## Table of Contents

- [Overview](#overview)
- [Problem Statement](#problem-statement)
- [Why This Is Hard](#why-this-is-hard)
- [Approach](#approach)
  - [Continuous, Not One-Time](#continuous-not-one-time)
  - [Adaptive Modeling](#adaptive-modeling)
- [Architecture](#architecture)
  - [Data Collection](#data-collection)
  - [Feature Extraction](#feature-extraction)
  - [Model](#model)
  - [Drift Handling](#drift-handling)
  - [System Flow](#system-flow)
- [Tech Stack](#tech-stack)
- [Setup and Installation](#setup-and-installation)
- [Usage](#usage)
- [Results / Evaluation](#results--evaluation)
- [Limitations](#limitations)
- [Future Work](#future-work)
- [Project Status](#project-status)

---

## Overview

Passwords, PINs, and face/fingerprint unlock are all **one-time gates**: prove who you are once, and the system trusts you for the rest of the session. This project builds a second, silent layer of security that sits *underneath* those gates — a system that watches **how** a user types (rhythm, timing, pressure patterns) and continuously checks that the person typing right now is still the person who logged in.

Think of it as a **digital twin of typing behavior**: a model that learns a user's unique keystroke "signature" and keeps comparing live behavior against it, flagging anomalies in real time.

## Problem Statement

Traditional authentication has a blind spot:

- Once a session is unlocked, there's no ongoing check that the authenticated user is still the one at the keyboard.
- A stolen password, a spoofed face-unlock, or an unattended unlocked device all bypass one-time auth completely with no way to catch the takeover afterward.
- Existing continuous-auth research often treats typing behavior as static, which doesn't hold up in the real world.

This project addresses the gap: **continuous, behavior-based re-verification that runs quietly in the background of an already-authenticated session.**

## Why This Is Hard

Two real, well-documented flaws make naive keystroke authentication unreliable, and this project is designed around them rather than ignoring them:

1. **Newbie typists look alike.** Inexperienced or slow typists tend to produce similar raw timing patterns (long dwell times, irregular flight times), making it harder to distinguish one new user from another using timing alone.
2. **Typing patterns drift.** A person's own typing rhythm changes over time — with practice, fatigue, injury, stress, or even the keyboard they're using. A model trained once and never updated will start rejecting the legitimate user as their behavior naturally shifts.

A system that ignores either of these will either fail to distinguish users early on, or generate false rejections over time. Solving for both is the core design challenge.

## Approach

### Continuous, Not One-Time

Rather than replacing login, this system runs **after** authentication, silently re-verifying the user at intervals (or continuously) throughout the session. This means it can catch:

- Sessions hijacked after password compromise
- Face/fingerprint spoofing that got past the initial check
- Unattended devices being used by someone else mid-session

### Adaptive Modeling

Instead of a single fixed baseline captured once, the user's "digital twin" **updates over time**:

- The model adapts to gradual, legitimate drift (practice, fatigue, context)
- Sudden, large deviations from the adapted baseline are flagged as anomalies rather than absorbed as "normal drift"
- Early-session uncertainty (when the model doesn't have enough data yet) is handled differently from steady-state verification, to avoid over-trusting a small sample

## Architecture

*(Fill in as implementation solidifies — structure below is the intended shape.)*

### Data Collection
- Raw keystroke events captured: key-down / key-up timestamps
- Derived low-level signals: dwell time (how long a key is held), flight time (gap between keys), typing speed, error/backspace rate

### Feature Extraction
- Per-keystroke and per-digraph/trigraph timing features
- Rolling statistical features (mean, variance) over a sliding window to capture short-term behavior
- Session-level aggregates for baseline comparison

### Model
- Baseline "twin" built from an initial calibration period
- Anomaly scoring: live feature windows are compared against the twin's current expected distribution
- Decision threshold tuned to balance false accepts (impostor let through) vs false rejects (real user flagged)

### Drift Handling
- Periodic, bounded updates to the baseline using verified legitimate sessions (so the twin evolves with the user)
- Guardrails to prevent an attacker from slowly "retraining" the model to their own behavior
- Separate handling for short-term anomalies (stress, one bad session) vs long-term drift (practice, injury)

### System Flow
```
Login (password/face) 
      ↓
Session starts → background keystroke capture begins
      ↓
Feature extraction (rolling window)
      ↓
Compare against adaptive baseline ("digital twin")
      ↓
   Anomaly? ──No──→ Continue session, periodically update baseline
      │
     Yes
      ↓
Step-up challenge / alert / lock session
```

## Tech Stack

*(Fill in with what you're actually using, e.g.)*
- Language: 
- Keystroke capture: 
- ML/stats library: 
- Storage: 
- Interface (daemon / browser extension / OS hook): 

## Setup and Installation

*(Fill in once implementation exists, e.g.)*
```bash
git clone <repo-url>
cd <project-folder>
pip install -r requirements.txt
python main.py
```

## Usage

*(Fill in — how does someone run calibration, then run live monitoring?)*

## Results / Evaluation

*(This section matters most for resume/portfolio credibility — fill in once you have numbers)*
- Dataset used for evaluation (public dataset vs self-collected)
- False Acceptance Rate (FAR)
- False Rejection Rate (FRR)
- Equal Error Rate (EER)
- Time-to-detect for simulated impostor sessions
- Comparison against a static (non-adaptive) baseline model, to show the adaptive approach's benefit

## Limitations

- Keystroke dynamics alone cannot fully replace strong authentication — it's a *complementary* signal, not a standalone gate.
- Early-session verification is inherently less reliable due to limited data (the "cold start" problem for new typists).
- Adaptive updating introduces a trade-off: too permissive and an attacker could gradually shift the baseline toward their own behavior; too strict and legitimate drift causes false rejections.
- Behavior can vary across devices/keyboards (laptop vs external keyboard vs mobile), which may require separate baselines per device.
- Not yet tested against sophisticated replay or timing-mimicry attacks.

## Future Work

- Extend beyond keystroke timing to other behavioral signals (mouse movement, touch gestures) for a richer digital twin
- Cross-device baseline reconciliation
- Formal adversarial testing (mimicry attacks, gradual baseline poisoning)
- Package as a lightweight background service/daemon for real deployment
- Publish benchmark results against existing continuous-auth literature

## Project Status

Actively in development. This project is being built as a resume-worthy, in-depth system — not a toy demo — with multiple days of dedicated design and implementation work.
