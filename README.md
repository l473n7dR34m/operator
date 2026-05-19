# Operator

![Python](https://img.shields.io/badge/Python-3.11%2B-blue?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-async%20backend-009688?logo=fastapi&logoColor=white)
![Status](https://img.shields.io/badge/Status-Production%20%E2%80%94%20daily%20use-brightgreen)
![License](https://img.shields.io/badge/License-Private-lightgrey)

A self-hosted AI operator platform. Deploy focused AI assistants wired to your tools, available on the channels your team already uses, running entirely on your own infrastructure.

Built and maintained by [Ryan Bullivant](https://github.com/l473n7dr34m) — Sysadmin / IT Lead, 10 years in IT operations, automation, and integrations.

---

## Live Example: Dex

> The reference deployment. Ryan's personal AI operator, in daily production use since early 2026.

Dex runs on this platform and handles a real daily workload across Telegram and a local web UI. Context is preserved between channels — one brain, multiple entry points.

**What it does in production:**

- Triage and draft emails via Gmail, with full thread context
- Create, update, and delete Google Calendar events from plain-language instructions
- Deliver daily briefings: news, weather, surf report, today's schedule — one message, no fuss
- Monitor inventory via the Unleashed Sentinel and fire Telegram alerts the moment stock goes RED
- Automate a weekly grocery run: browse Coles, clear the cart, add items sorted by price
- Generate tailored job application cover letters from a job description
- Transcribe Telegram voice messages via Whisper API before processing
- Run a personalised TV guide weekly, delivered by email
- Track fire danger and flood warnings (NSW RFS / BOM) for a remote rural property
- Log sessions, maintain memory across restarts, recover gracefully after context resets

Responds in seconds. Occasionally sarcastic.

> **Screenshot:** Dex responding in Telegram — inbox triage session

> **Screenshot:** Nexus web UI — chat panel with Dex, live dashboard sidebar

---

## Skills Demonstrated

Quick reference for hiring managers. Full detail in each section below.

| Area | Specifics |
|------|-----------|
| **Python** | FastAPI backend, async handlers, subprocess management, HMAC auth, scheduled jobs |
| **API integrations** | Gmail, Google Calendar, Google Drive, Slack, Telegram, Shopify, Unleashed, Stripe, Spotify, Starlink, GitHub, Anthropic |
| **Auth** | Google OAuth2 (full flow), Bot token auth (Slack, Telegram), API key + HMAC signing (Unleashed) |
| **LLM / AI tooling** | Anthropic API (Sonnet, Opus), MCP protocol, tool-use orchestration, prompt engineering, context management |
| **Automation** | Cron scheduling, browser automation (Playwright), voice transcription pipeline (Whisper), event-driven hooks |
| **System design** | Multi-channel unified context, modular operator registry, hot-swappable skill system, hook-based safety layer |
| **Frontend** | Nine live dashboards (HTML/CSS/JS), PWA web UI, real-time data refresh |
| **Infrastructure** | Self-hosted, VPS-portable, offline-capable (Starlink gRPC against local hardware) |

---

## What It Is

Operator is a framework for deploying AI assistants into real workflows. Each **operator** has a defined identity, scope, and skill set. Users interact via Telegram, Slack, or a web UI. All channels share a single conversation log — context is preserved regardless of which channel was used last.

Operators are not general-purpose chatbots. Each one is scoped to a specific role: a defined system prompt, a curated skill set, and a controlled set of allowed tools. A deployment might include an Inbox Operator, a Projects Operator, and an Inventory Operator — each one focused, each one fast.

> **Screenshot:** Operator registry view — agents list with status and config overview

---

## Architecture

The platform runs in two layers depending on whether server portability is needed:

```
                        CHANNELS
        ┌─────────────┬──────────────┬─────────────┐
        │  Telegram   │    Slack     │  Nexus UI   │
        └──────┬──────┴──────┬───────┴──────┬──────┘
               │             │              │
               └─────────────┼──────────────┘
                             ▼
                  Unified conversation log
               (conversation.jsonl — tagged
                by source channel + timestamp)
                             │
               ┌─────────────┴─────────────┐
               ▼                           ▼
      Claude CLI operators          FastAPI web UI
      + MCP tool connectors         + Python tooling layer
      (Gmail, Calendar, Drive,      (Google OAuth2, Slack API,
       Canva — auth via             custom integrations)
       claude.ai OAuth)             — portable to any VPS
       — desktop/local
               │                           │
               └─────────────┬─────────────┘
                             ▼
                     Operator registry
                     (agents/ directory)
                     config.json + system.md
                     + skills/ per operator
```

**MCP** (Model Context Protocol) is an open standard that lets an AI model call external tools at runtime — like a function library, but the AI decides when and how to use each tool. The CLI layer uses this to give operators access to Gmail, Calendar, Drive, and Canva without writing integration code — OAuth is handled by claude.ai.

The **web UI layer** reimplements the same integrations natively in Python, making it fully portable to a server without a CLI dependency.

> **Screenshot:** Architecture diagram or Nexus UI showing channel source tags in the conversation log

---

## Operator Registry

Each operator is a self-contained directory:

```
agents/
  dex/
    config.json       — id, name, model, channel, log path
    system.md         — persona, scope, allowed tools, response format
    skills/           — operator-specific skill overrides
```

The web UI loads all agents at startup. Operators can be added, updated, or swapped without touching application code.

> **Screenshot:** agents/ directory structure or web UI operator management panel

---

## Skills System

Operators are extended through **skills** — modular instruction sets that load dynamically at runtime. Each skill is a markdown file: what it does, when to invoke it, and step-by-step execution instructions.

```
skills/
  cover-letter/       — tailored job application letters
  job-search/         — SEEK/LinkedIn lookup and filtering
  meal-planner/       — weekly meal prep and grocery lists
  surf-report/        — swell conditions via BoM and Swellnet
  calendar/           — Google Calendar queries and updates
  gmail/              — inbox triage and email drafting
  weather-alerts/     — fire danger and flood warnings (NSW RFS / BOM)
  coles-shop/         — automated grocery cart via Playwright browser
  unleashed-sentinel/ — inventory monitoring and reorder alerts
  tv-guide/           — personalised weekly TV picks delivered by email
  ... (30+ skills total)
```

No code changes required to add a capability. Write the instruction file, register the skill name in `system.md`, and the operator uses it.

### Orchestrator

Multi-step workflows are defined as **systems** — pipeline files that chain skills together with checkpoints. The orchestrator invokes each skill in sequence, passes outputs forward, and pauses at defined checkpoints for human confirmation before continuing.

Available pipelines: job application, morning briefing, weekly review, garden check.

> **Screenshot:** Morning briefing pipeline output — news, weather, surf, calendar in a single Telegram message

---

## Integrations

Fourteen live integrations. Each one is implemented, authenticated, and running in the reference deployment.

| Integration | Auth method | What it does |
|-------------|-------------|--------------|
| **Gmail** | Google OAuth2 | Read threads, search, draft, send, label |
| **Google Calendar** | Google OAuth2 | List, create, update, delete events |
| **Google Drive** | Google OAuth2 | Read, search, copy, create files |
| **Slack** | Slack Bolt (bot token) | Post, read, manage channels, structured logs |
| **Telegram** | Bot API | Messages, file attachments, voice, emoji reactions |
| **Canva** | MCP (OAuth) | Design creation, editing, export |
| **Shopify** | REST Admin API | Orders, products, fulfilment status |
| **Unleashed** | REST API + HMAC signing | Inventory, stock levels, purchase orders |
| **Stripe** | REST API | Charges, subscriptions, revenue reporting |
| **Spotify** | Web API | Playback state, history, library |
| **Starlink** | gRPC (local dish API) | Signal quality, latency, uptime, obstruction map |
| **GitHub** | REST API | Repos, commits, issues, pull requests |
| **Whisper** | Groq / OpenAI API | Voice transcription (whisper-large-v3 / whisper-1) |
| **Anthropic API** | Python SDK | All AI inference — Sonnet and Opus models |

> **Screenshot:** Integrations dashboard or a Telegram alert fired from the Unleashed Sentinel

---

## Dashboards

The Nexus web UI includes nine live per-service dashboards alongside the chat interface. Each pulls live data from its API on demand.

| Dashboard | What it shows |
|-----------|---------------|
| **Gmail** | Unread count, recent threads, label overview |
| **Slack** | Channel activity, unread messages |
| **Shopify** | Orders, revenue, fulfilment queue |
| **Unleashed** | Inventory levels, stock alerts, reorder queue |
| **GitHub** | Repo activity, open PRs, recent commits |
| **Stripe** | Revenue, recent charges, subscription status |
| **Spotify** | Now playing, recently played, top tracks |
| **Starlink** | Signal quality, latency, uptime, obstruction map |
| **Anthropic** | API usage, token spend, model breakdown |

Self-contained HTML pages served by the FastAPI backend. No external dependencies — all data fetched server-side.

> **Screenshot:** Nexus dashboard panel — Unleashed stock levels or Starlink signal stats

---

## Hooks

Session behaviour is controlled by an event hook system. Hooks are Python scripts that run outside the AI context — reliable for logging and safety enforcement regardless of model behaviour.

| Hook | Fires when | What it does |
|------|-----------|--------------|
| `UserPromptSubmit` | User sends a message | Mirrors message to Nexus web UI in real time |
| `PostToolUse` | Any tool call completes | Logs tool calls; mirrors Telegram replies to Nexus |
| `Stop` | Session ends | Writes session log, updates context files, fires post-compaction recovery |
| `PreToolUse` | Before any tool call | Enforces safety rules — blocks destructive operations without explicit confirmation |

This hook layer is how the platform enforces consistent safety behaviour that cannot be overridden by prompt manipulation.

---

## Unified Conversation Context

All channels share a single conversation history. A message sent on Telegram, a reply via Slack, a follow-up in the web UI — the operator sees them all in order, tagged by source.

```jsonl
{"role": "user",     "source": "telegram", "ts": "...", "content": "..."}
{"role": "assistant","source": "telegram", "ts": "...", "content": "..."}
{"role": "user",     "source": "nexus",    "ts": "...", "content": "..."}
```

No context loss between channels. One brain, multiple entry points.

---

## Voice Pipeline

Voice messages sent via Telegram are transcribed before processing:

```
Telegram voice note → download → Whisper API (Groq or OpenAI) → transcript → operator
```

Transcription is confirmed to the user in one line before the response. Supports Groq (`whisper-large-v3`) and OpenAI (`whisper-1`).

---

## Scheduled Tasks

Operators support cron-based scheduling. Jobs can:

- Run any skill or pipeline on an interval
- Fire alerts when conditions are met (e.g. stock drops below threshold)
- Deliver digests at a fixed time (weekly TV guide, morning briefing)
- Trigger recovery pings after context compaction

Schedules persist across sessions and are defined per operator.

---

## Persona and Identity

Operators have defined identities — not just instructions. Each operator has a name, a consistent personality, a voice, and a visual appearance concept used across UI elements.

**Dex** (the reference deployment): dry, precise, occasionally sarcastic, and completely unbothered. Minimal words. Exact numbers. Treats every task like it's beneath them but does it perfectly.

Visual concept: dark hair, angular face, one eye glowing blue (cybernetic), dark high-collar suit. Blue accent lighting.

Identity is defined in `system.md` alongside behavioural instructions. The same file drives both personality and the Nexus UI's visual theme for that operator.

> **Screenshot:** Nexus UI showing Dex's visual identity and persona reflected in the interface

---

## Deployment

Self-hosted per instance. No multi-tenancy, no shared data, no external telemetry after setup.

**Requirements:**
- Python 3.11+
- Claude Code CLI (for CLI-channel operators)
- Channel credentials (Telegram bot token, Slack app credentials)
- Google OAuth2 credentials (for Gmail, Calendar, Drive)

**Start the web UI:**
```bash
python ui/server.py
```

**Start a Telegram operator (CLI session):**
```bash
claude --dangerously-skip-permissions --channels plugin:telegram@claude-plugins-official
```

Configuration lives in `CLAUDE.md` (operator behaviour), `agents/` (operator registry), and `.env` (credentials). Fully isolated — no remote access after handover.

---

## Tech Stack

**Backend:** Python 3.11, FastAPI, asyncio, httpx, Playwright

**AI / LLM:** Anthropic API (claude-sonnet-4, claude-opus-4), MCP protocol, Claude Code CLI, Whisper API (Groq / OpenAI)

**Integrations:** Google OAuth2, Telegram Bot API, Slack Bolt, Shopify REST, Unleashed REST + HMAC, Stripe, Spotify Web API, Starlink gRPC, GitHub REST

**Frontend:** HTML / CSS / JavaScript (nine dashboard UIs), PWA-capable web UI

**Infrastructure:** Self-hosted, VPS-portable, PowerShell scripting (Windows), designed for headless server deployment

---

## Status

Active development. The reference deployment (Dex) has been in daily production use since early 2026. The platform is being hardened for multi-client deployment.

---

*Built by Ryan Bullivant — Systems Administrator, automation builder, and reluctant perfectionist.*
*GitHub: [l473n7dr34m](https://github.com/l473n7dr34m)*
