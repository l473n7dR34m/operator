# operator

A self-hosted AI operator platform. Deploy focused AI assistants for your team — wired to your tools, available on the channels your people already use, running entirely on your own infrastructure.

---

## What it is

Operator is a framework for deploying AI assistants into real workflows. Each **operator** has a defined identity, scope, and skill set. Staff interact via Telegram, Slack, or a local web UI. All channels share a single conversation log — context is preserved regardless of which channel someone used last.

Operators are not general-purpose chatbots. Each one is scoped to a specific role with a defined system prompt, a curated skill set, and a fixed set of allowed tools. A deployment might include an Inbox Operator, a Projects Operator, and an Inventory Operator — each one focused, each one fast.

---

## Architecture

The platform runs in two layers depending on whether portability to a server is required:

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
      claude CLI operators          FastAPI web UI
      + MCP connectors              + Python tooling
      (Gmail, Calendar,             layer (Google
       Drive, Canva, etc.)          OAuth2, Slack API,
       via claude.ai OAuth)         custom integrations)
       — desktop/local              — portable to any
                                    server or VPS
               │                           │
               └─────────────┬─────────────┘
                             ▼
                     Operator registry
                     (agents/ directory)
                     config.json + system.md
                     + skills/ per operator
```

**Messaging operators** run as `claude` CLI sessions with a channel plugin attached. They get Gmail, Google Calendar, Google Drive, and Canva MCP tools without any integration code — authentication is handled by claude.ai's OAuth.

**Web UI operators** run via FastAPI calling the Anthropic API directly. The Python tooling layer provides the same integrations natively, making this path fully portable to a server without a claude CLI dependency.

---

## Operator Registry

Each operator lives in `agents/` as an isolated directory:

```
agents/
  dex/
    config.json       — id, name, model, channel, log path
    system.md         — persona, scope, allowed tools, response format
    skills/           — operator-specific skill overrides
```

The web UI loads all agents at startup and provides a management interface: view status, switch between conversation logs, update configuration.

---

## Persona & Appearance

Operators have defined identities — not just instructions. Each operator has a name, a personality, a consistent voice, and an appearance concept used across UI elements.

**Dex** (the reference deployment) is defined as: dry, precise, occasionally sarcastic, and completely unbothered. Minimal words. Exact numbers. Treats every task like it's beneath them but does it perfectly.

Visually: dark hair, angular face, one eye glowing blue (cybernetic), dark high-collar suit. Blue accent lighting. The kind of face that has already decided the answer before you finish the question.

Operator appearance is defined in `system.md` alongside behaviour. The same file drives both personality and the Nexus UI's visual identity for that operator.

---

## Skills System

Operators are extended through **skills** — modular instruction sets that load dynamically based on context. Each skill is a markdown file describing a specific capability: what it does, when to invoke it, and how to execute it step by step.

```
skills/
  cover-letter/       — tailored job application letters
  job-search/         — SEEK/LinkedIn lookup and filtering
  meal-planner/       — weekly meal prep and grocery lists
  surf-report/        — swell conditions via BoM and Swellnet
  calendar/           — Google Calendar queries and updates
  gmail/              — inbox triage and email drafting
  weather-alerts/     — fire danger and flood warnings
  coles-shop/         — automated grocery cart via browser
  unleashed-sentinel/ — inventory monitoring and reorder alerts
  tv-guide/           — personalised weekly TV picks via email
  ... (30+ skills)
```

Skills are plain markdown. No code changes required to add a new capability — write the instruction file, register the skill name in the operator's `system.md`, and the operator knows how to use it.

### Orchestrator

Multi-step workflows are defined as **systems** — pipeline definitions that chain skills together with checkpoints. The orchestrator reads the system file, invokes each skill in sequence, passes outputs forward, and pauses at defined checkpoints for human confirmation before continuing.

Available systems: job application pipeline, morning briefing, weekly review, garden check.

---

## Hooks

The platform extends Claude Code's hook system to wire operator behaviour to session events:

| Hook | Trigger | Action |
|------|---------|--------|
| `UserPromptSubmit` | User sends a message | Mirrors incoming message to Nexus web UI in real time |
| `PostToolUse` | Tool call completes | Logs tool calls; mirrors Telegram replies to Nexus UI |
| `Stop` | Session ends | Writes session log, updates current topic file, fires post-compaction recovery |
| `PreToolUse` | Before tool call | Enforces safety rules — blocks destructive operations without confirmation |

Hooks are Python scripts in `scripts/`. They run outside the AI context, making them reliable for logging and safety enforcement regardless of model behaviour.

---

## Unified Conversation Context

All channels share a single conversation history. A message sent on Telegram, a reply via Slack, a follow-up in the web UI — the operator sees all of it in order, tagged by source.

```jsonl
{"role": "user",    "source": "telegram", "ts": "...", "content": "..."}
{"role": "assistant","source": "telegram", "ts": "...", "content": "..."}
{"role": "user",    "source": "nexus",    "ts": "...", "content": "..."}
```

No context loss between channels. One brain, multiple entry points.

---

## Dashboards

The Nexus web UI includes live per-service dashboards alongside the chat interface. Each dashboard pulls data from its respective API and presents it in a compact, mobile-friendly format:

| Dashboard | Data source |
|-----------|-------------|
| Gmail | Unread count, recent threads, label overview |
| Slack | Channel activity, unread messages |
| Shopify | Orders, revenue, fulfilment status |
| Unleashed | Inventory levels, stock alerts, reorder queue |
| GitHub | Repo activity, open PRs, commit history |
| Stripe | Revenue, recent charges, subscription status |
| Spotify | Now playing, recently played, top tracks |
| Starlink | Signal quality, latency, uptime, obstruction map |
| Anthropic | API usage, token spend, model breakdown |

Dashboards are self-contained HTML pages served by the FastAPI backend, with data refreshed on demand or on a schedule.

---

## Schedules

Operators support scheduled tasks via the `CronCreate` tool. A scheduled job can:

- Run any skill or pipeline on a defined interval
- Fire alerts when conditions are met (e.g. stock goes RED in the sentinel)
- Send digests (weekly TV guide, morning briefing) at a configured time
- Trigger recovery pings after context compaction

Schedules are defined per-operator and persist across sessions.

---

## Channels

### Telegram
Primary mobile interface. Two-way messaging with the operator. Supports:
- Text, images, file attachments
- Voice messages (transcribed via Whisper API before processing)
- Emoji reactions for lightweight acknowledgement
- Reply threading for multi-message conversations

### Slack
Workspace integration. The operator can:
- Post to any channel
- Read channel history
- Create new channels
- Maintain structured channels (briefings, logs, backlog, projects)

### Nexus Web UI
Local or VPS-hosted PWA. The primary management interface:
- Full chat with the operator
- Dashboard panel (per-service views above)
- Operator registry and configuration
- Unified conversation log viewer across all channels
- Installable from browser — no app store required

---

## Voice Messages

Voice messages sent to any channel are transcribed before processing:

```
Telegram voice message → download → Whisper API (Groq or OpenAI) → transcript → operator
```

Transcription is confirmed to the user in one line before the response. Supports Groq (`whisper-large-v3`) and OpenAI (`whisper-1`).

---

## Integrations

| Integration | Method | Capabilities |
|-------------|--------|-------------|
| Gmail | MCP (CLI) / Google OAuth2 (web) | Read, search, draft, send, label |
| Google Calendar | MCP (CLI) / Google OAuth2 (web) | List, create, update, delete events |
| Google Drive | MCP (CLI) / Google OAuth2 (web) | Read, search, copy, create files |
| Slack | Slack Bolt (bot token) | Post, read, manage channels |
| Telegram | Bot API | Messages, files, voice, reactions |
| Canva | MCP (CLI) | Design creation and export |
| Shopify | REST Admin API | Orders, products, fulfilment |
| Unleashed | REST API + HMAC auth | Inventory, products, purchase orders |
| Stripe | REST API | Charges, subscriptions, revenue |
| Spotify | Web API | Playback state, history, library |
| Starlink | gRPC (local dish API) | Signal stats, latency, obstruction |
| GitHub | REST API | Repos, commits, issues, PRs |
| Whisper | Groq / OpenAI API | Voice transcription |
| Anthropic API | Python SDK | All AI inference (Sonnet, Opus) |

---

## Deployment

Self-hosted, per instance. No multi-tenancy, no shared data.

**Requirements:**
- Python 3.11+
- Claude Code CLI (for messaging channel operators)
- Channel credentials (Telegram bot token, Slack app, etc.)
- Google OAuth2 credentials (for Google Workspace integrations)

**Startup:**
```bash
# Web UI
python ui/server.py

# Telegram operator (CLI session)
claude --dangerously-skip-permissions --channels plugin:telegram@claude-plugins-official
```

Configuration lives in `CLAUDE.md` (operator behaviour), `agents/` (operator registry), and `.env` (credentials). Each instance is fully isolated — no telemetry, no remote access after handover.

---

## Live Example: Dex

Dex is the reference deployment — Ryan's personal AI operator, running daily since early 2026.

**What it handles:**

- Inbox triage and email drafting (Gmail)
- Calendar management — create, update, delete events (Google Calendar)
- Daily briefings: news, weather, surf report, schedule
- Job search and tailored cover letter writing
- Grocery automation (Coles — browser-based cart filling)
- Inventory monitoring: Unleashed Sentinel fires Telegram alerts when stock goes RED
- Personalised TV guide delivered weekly by email
- Session logging and weekly review pipeline
- Voice message transcription
- Fire danger and flood warnings (NSW RFS / BoM)
- Garden tracking (off-grid permaculture plot, Northern Rivers NSW)

Available on Telegram (primary) and the Nexus web UI. Context preserved across both. Responds in seconds. Occasionally sarcastic.

---

## Screenshots

*Coming soon — Nexus web UI, Telegram session, dashboard views, skills in action.*

---

## Tech Stack

Python · FastAPI · Claude Code CLI · Anthropic API (claude-sonnet-4, claude-opus-4) · MCP protocol · Google OAuth2 · Telegram Bot API · Slack Bolt · Whisper API (Groq / OpenAI) · PowerShell · HTML/CSS/JS (dashboard UIs)

---

## Status

Active development. The reference deployment (Dex) is in daily use. The platform is being hardened for multi-client deployment.
