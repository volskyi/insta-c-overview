# insta-c: overview

> **Source code is private.** This repository describes the system's logic and architecture.

**An Instagram growth assistant for a small local bookshop. It finds people who are likely to care
(Ukrainian readers living in Madrid and the rest of Spain), ranks them, and gives the owner a short daily list.
A person makes every follow; the tool decides who is worth it and how many are safe today.**

![Python](https://img.shields.io/badge/Python-3-3776AB)
![FastAPI](https://img.shields.io/badge/FastAPI-dashboard-009688)
![Playwright](https://img.shields.io/badge/Playwright-read--only-2EAD33)
![SQLite](https://img.shields.io/badge/SQLite-state-003B57)

## What makes it hard

Audience-building tools get accounts restricted. The goal is the opposite of a follow bot: slow,
targeted and impossible to confuse with automation, on the one account the business depends on.

## How it works

**Collect (read-only, separate account)**
- A second, read-only browser profile gathers people who comment on similar shops' recent posts; the business
  account is never used for scraping
- Slow by design: random pauses, small batches; **any login, challenge or checkpoint page halts everything**

**Score**
- Parses profile counters ("16,4 тис.", "1,2 млн") and bio text in Ukrainian, Russian, Spanish and English
- Location scoring: Madrid ranks highest, the rest of Spain next, clear signs of another country disqualify
- Filters spam/commercial profiles and a blocklist (competitors and their owners)

**Act (human in the loop)**
- Mobile dashboard shows the top of the queue with bio, score and source; the owner follows by hand and taps one button
- **Daily budget** ramps up week by week under a hard cap, unfollows capped at half of it, active hours only,
  one rest day a week, plus a kill switch
- Follow-back rate is tracked **per source**, so only sources that actually convert are kept

## Architecture

```
reader profile (Playwright, headless, Xvfb) ──► candidates ──► scoring ──► SQLite
                                                                   │
              owner's phone ◄── FastAPI dashboard (Tailscale only) ┘
```

Runs on a home server as a `systemd --user` service with a memory limit; the dashboard listens on the tailnet only.

## My role

Designed and built it, and run it for my own shop. Main themes: risk-aware automation, scoring messy
multilingual text, and keeping a human in the loop where it matters.
