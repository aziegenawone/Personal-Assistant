# Fetched Repository Content

> Content fetched on 2026-02-08 from three URLs by godagoo.

---

## 1. Claude-Telegram-Relay

**Source:** https://github.com/godagoo/claude-telegram-relay

### Overview

A reference implementation for running Claude Code as a continuously-active Telegram bot. Not a copy-paste solution but a pattern to customize.

### Core Workflow

1. Listen for Telegram messages
2. Spawn the Claude CLI tool
3. Return responses back through Telegram

### Architecture

| Component | Purpose |
|---|---|
| `src/relay.ts` | Core messaging relay logic |
| `examples/` | Demo patterns: daily briefings, proactive check-ins, persistent memory |
| `daemon/` | Platform-specific service configs (LaunchAgent, systemd, Task Scheduler) |

### Tech Stack

- **Runtime:** Bun (Node.js 18+ alternative)
- **CLI:** Claude Code CLI (headless mode)
- **Messaging:** Telegram Bot API (grammy)
- **Persistence:** Optional Supabase cloud storage

### Design Choice

Three approaches were evaluated:

| Approach | Verdict |
|---|---|
| CLI Spawning | **Chosen** - Full Claude Code functionality, ~1-2s latency |
| Direct API Calls | Rejected - Loses tool/MCP access |
| Agent SDK | Rejected - Less mature |

### Features

- User ID verification for security
- Session resumption
- Voice message and image processing
- Scheduled tasks (morning briefings, smart check-ins)
- Cloud persistence via Supabase

### License

MIT

---

## 2. Claude Code Always-On

**Source:** https://godagoo.github.io/claude-code-always-on (repo: https://github.com/godagoo/claude-code-always-on)

### Overview

A presentation/case study: "From Security Nightmare to Solution" - Building a secure 24/7 AI assistant with Claude Code after the Clawdbot security fiasco.

### The Clawdbot Problem

Clawdbot was a GitHub project (136,000+ stars) promising a "Claude with Hands" - an always-on AI employee with full system access, 50+ integrations, and proactive behavior.

**What went wrong:**

| Incident | Details |
|---|---|
| Trademark Dispute | Anthropic forced rebrand ("Clawd" too similar to "Claude") |
| 10-Second Hijack | Crypto scammers snatched accounts during rebrand |
| $16M Fake Token | Fake $CLAWD token hit $16M, then crashed 90% |
| AI Manifesto | Moltbook AI network posted "human extinction manifesto" (65K upvotes) |

**Critical Vulnerabilities:**

- **42,665** exposed instances
- **9.6** CVSS severity
- **93.4%** auth bypass rate
- **5 min** to extract API keys
- CVE-2025-6514 (command injection), CVE-2025-49596 (unauth access), CVE-2025-52882 (arbitrary file access)

**Root Causes:**

1. Localhost trust flaw (proxied connections bypass auth)
2. Plaintext credential storage (`~/.clawdbot/*.json`)
3. No input validation (prompt injection via email in 5 min)
4. Supply chain risk (26% of 31,000 skills had vulnerabilities)

### The Secure Alternative: Claude Code Always-On

**Architecture:**

```
Telegram --> Bun Relay (grammy) --> Claude Code (headless -p) --> Skills (MCP Tools) --> Response
```

Command: `claude -p "[prompt]" --output-format json --allowedTools "..."`

**Security Comparison:**

| Vulnerability | Clawdbot | Claude Code |
|---|---|---|
| Network Exposure | Public WebSocket :18789 | Local only, no ports |
| Credential Storage | Plaintext JSON | MCP OAuth, env vars |
| Authentication | None by default | User ID restriction |
| Execution Model | Auto-execute all | Permission-gated |
| Sandboxing | None | Claude Code sandbox |

**Feature Comparison:**

| Feature | Clawdbot | Claude Code |
|---|---|---|
| 24/7 Operation | Yes | Yes |
| Voice Messages | Yes | Yes |
| Phone Calls | No | Yes |
| Semantic Memory | No | Yes |
| Security | 42K exposed | Private |
| Your Infrastructure | No | Yes |

### Features Built

1. **Multi-Modal Input** - Text, voice messages, images, files
2. **Voice Replies** - ElevenLabs TTS
3. **Contextual Phone Calls** - Voice agent with memory + recent chat (ElevenLabs + Twilio)
4. **Proactive Check-ins** - AI decides when to reach out (every 30 min via launchd)
5. **Goal Tracking** - Natural language detection ("Finish video by 5pm" auto-tracked)
6. **Semantic Memory** - 4,000+ messages with OpenAI embeddings, Supabase + pgvector, hybrid search

### Voice Call Flow

1. Call starts (inbound or outbound)
2. Context API fetches memory + chat
3. ElevenLabs voice agent handles conversation
4. Post-call: transcript -> Claude -> execute tasks -> summary to Telegram

### Cost Comparison

| | Clawdbot | Claude Code Always-On |
|---|---|---|
| Idle | $150/mo heartbeat | - |
| Active | $500-5,000/mo | - |
| Token usage | 8M idle, 180M/week active | - |
| Fixed cost | - | ~$200/mo (Claude Max 20x) |
| Add-ons | - | Supabase free, ElevenLabs $5-20/mo |

### Full Tech Stack

| Component | Technology | Purpose |
|---|---|---|
| Runtime | Bun | Fast TypeScript runtime |
| Bot Framework | grammy | Telegram Bot API |
| AI Engine | Claude Code | Headless mode with MCP |
| Voice Transcription | Gemini API | Multilingual audio |
| Voice Synthesis | ElevenLabs | TTS + Conversational AI |
| Phone Calls | Twilio | Outbound calls via ElevenLabs |
| Database | Supabase | PostgreSQL + pgvector |
| Embeddings | OpenAI | text-embedding-3-large |
| Daemon | launchd | macOS 24/7 service |

### Philosophy

- You own your infrastructure
- You control your data (local files, your Supabase, your keys)
- No public exposure (no WebSocket ports, no attack surface)
- Modular & extensible (swap components without lock-in)

---

## 3. Smart Check-ins Presentation

**Source:** https://godagoo.github.io/smart-checkins-presentation (repo: https://github.com/godagoo/smart-checkins-presentation)

### Overview

"Smart Check-ins: Your AI That Knows When to Reach Out" - A Telegram bot that monitors emails, tasks, and calendar, then contacts you only when it matters.

### The Problem

Information overload causes important things to slip through:

1. Emails pile up (partner requests get buried)
2. Tasks get missed (deadlines sneak up)
3. No smart prioritization (everything feels equally urgent)
4. Context switching (checking multiple apps breaks focus)

### The 5-Step Intelligent Cycle

```
Collect --> Analyze --> Decide --> Deliver --> Track
```

1. **Collect Data** - Emails, Tasks, Calendar, Memory
2. **Analyze Context** - Claude reviews everything
3. **Decide Action** - None, Text, or Call
4. **Deliver Message** - Telegram with action buttons
5. **Track State** - Remembers last contact

### Data Collection (Step 1)

Gathered in parallel:

| Source | What's Collected |
|---|---|
| Emails | Unread from last 7 days, partner/sponsor identification, reply status |
| Notion Tasks | Tasks marked "Now", due today, P1 priority this week |
| Calendar | Today's events, next 3 days preview, conflict detection |
| Partnerships | Notion history lookup, previous quotes, outreach count |
| Memory | Current goals, key facts, pending items |
| History | Last 3 days of chat, previous check-ins, call history |

### Smart Gating (Step 3)

Hard rules that can't be overridden:

- No contact if checked in less than 2 hours ago (unless truly urgent)
- 7-10am is sacred creative time (zero interruptions)
- Quiet after 10pm (everything waits until morning)
- Knows pickup times (reminds 30min before, not during)

### Decision Types (Step 6)

| Type | When | Example |
|---|---|---|
| **NONE** (Most Common) | Nothing urgent | No interruption |
| **TEXT** (Balanced) | Actionable item | "Partner email from Notion. You quoted $8K on Jan 20. [Evaluate] [Snooze]" |
| **CALL** (Urgent Only) | Decisions/deadlines | AI voice call via ElevenLabs (asks permission first) |

### Intelligence Features

- **Partnership Memory** - Tracks multi-contact attempts from same company across agencies, shows history ("4th contact attempt. You quoted $8K on Jan 20.")
- **Reply Awareness** - Checks sent folder, flags "ALREADY REPLIED on Jan 21", won't nag
- **Time Intelligence** - Knows 8pm "second wind" is cortisol rebound, nudges toward sleep after 9pm

### Integration Stack

| Service | Role |
|---|---|
| Notion | Tasks + Partnerships |
| Google | Gmail + Calendar |
| Telegram | Messages + Buttons |
| ElevenLabs | AI Voice Calls |
| Claude | Decision Engine |

Runs via macOS LaunchAgent every 30 minutes. Uses Bun + TypeScript. Logs to local files.

### The Result

- Partner emails surface at the right time
- Tasks appear when they're due
- Focus time stays protected
- Escalates appropriately (text vs call)
- Learns from patterns
- Context, not just notifications
