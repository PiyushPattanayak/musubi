# Chat App — Project Discussion & Roadmap

## Goal
Build and publish an **AI-first chat app** (think mini Discord/WhatsApp) in Python, starting from scratch.
Learn industry-standard technologies along the way through hands-on exercises.

## Background
- New to Python, familiar with Java and Gradle
- No frontend experience (no HTML, React, etc.) — frontend is a later focus
- Learning Python for interview prep (individual contributor roles)
- Teaching approach: exercise-based, step by step, Java/Gradle analogies where helpful
- Python concepts (classes, threading, etc.) will be taught from scratch as we encounter them

---

## Tech Stack (introduced gradually)

| Technology | Purpose | When |
|------------|---------|------|
| `socket`, `threading` | Networking foundation | Stage 1 |
| JSON files | User accounts & history (before DB) | Stage 1 |
| PostgreSQL | Persistent storage (users, messages, channels) | Stage 2 |
| Redis | Real-time pub/sub, presence, rate limiting | Stage 3 |
| Claude API | AI bot, summarizer, smart replies | Stage 4 |
| AWS S3 | Image & file storage | Stage 5 |
| Tkinter → Flask + WebSockets | Desktop GUI → Web client | Stage 6 |
| Docker, AWS EC2 | Deploy & publish | Stage 7 |

### Protocol progression
```
Stage 1-5  → raw TCP (learn the concept, keep it simple)
Stage 5.5  → gRPC / Protobuf (optional bonus — replace raw TCP with typed streaming)
Stage 6    → REST (login, history, uploads) + WebSockets (live messages) for browser client
```

- WebSockets are browser-only — browsers can't open raw TCP sockets
- gRPC is a nice-to-have after Postgres + Redis — higher priority to learn those first
- If we skip gRPC, the app still works perfectly going straight to Stage 6

### Infrastructure
- **Server** runs on AWS EC2 (free tier: t2.micro, 750 hrs/month for 12 months)
- **Client** is what users install/visit — evolves from a Python terminal script to a web app
- **Postgres + Redis** run on the same EC2 instance
- **Images/files** stored in AWS S3 (free tier: 5GB for 12 months)
- Start local, deploy to EC2 at end of Stage 1

---

## Full Roadmap

### Stage 1a — Basic Chat (anyone can connect and chat)
| # | Project | What you learn |
|---|---------|----------------|
| 1 | Server accepts one connection, echoes messages | Sockets, Python basics |
| 2 | Handle multiple connections at once | Threading |
| 3 | Broadcast — everyone sees every message | Shared state, lists |
| 4 | Usernames | Dicts, state per connection |

### Stage 1b — Channels + Invite Links
| # | Project | What you learn |
|---|---------|----------------|
| 5 | Default `#general` channel, messages belong to a channel | Data structures |
| 6 | `/create channelname` — create a new channel | String parsing, commands |
| 7 | `/invite` — generate an invite token (e.g. `x7k2p`) | Random, tokens |
| 8 | `/join x7k2p` — join a channel via token | Lookup tables |

### Stage 1c — User Accounts (file-based)
| # | Project | What you learn |
|---|---------|----------------|
| 9 | Register and login (saved to a JSON file) | File I/O, JSON |
| 10 | Message history saved to file + unread tracking | Appending, reading files, timestamps |
| 11 | Deploy to AWS EC2 | Linux, cloud basics |

**Unread message tracking (Exercise 10):**
Each user stores a `last_seen` timestamp per channel. On login, messages after that timestamp are marked as new.

File structure:
```json
{
  "piyush": {
    "password": "...",
    "channels": {
      "general": { "last_seen": "2026-06-09T10:30:00" }
    }
  }
}
```

Terminal display:
```
[general]
  piyush: hey everyone       (seen)
  alex: wassup               (seen)
  --- 3 new messages ---
  maya: anyone around?       (NEW)
  alex: hello?               (NEW)
  maya: guess not lol        (NEW)
```

This same `last_seen` concept moves to Postgres in Stage 2 as a column in `channel_members`.

### Stage 2 — PostgreSQL
Migrate everything from JSON files to a real database.

**Schema:**
```
users           → id, username, password_hash, avatar_url, created_at, last_seen_at
channels        → id, name, description, created_by, is_private, created_at
channel_members → user_id, channel_id, role (owner/member), joined_at
messages        → id, user_id, channel_id, content, created_at, edited_at
direct_messages → id, sender_id, receiver_id, content, created_at
```

**Channel access model:**
- Public channels: anyone can join
- Private channels: join only via invite link (token)
- `role` in `channel_members`: owner can invite/delete, member can chat

### Stage 3 — Redis
| Focus | Details |
|-------|---------|
| Real-time messaging | Replace raw TCP broadcast with Redis Pub/Sub |
| Online presence | Who's online, who's typing (Redis key expiry / TTL) |
| Rate limiting | Prevent spam (Redis counters) |

### Stage 4 — AI Features (Claude API)
| Feature | Details |
|---------|---------|
| `@ai` bot | Mention AI in any channel, it responds |
| Summarizer | Summarize a channel's recent messages |
| Smart replies | Suggest replies to messages |

### Stage 5 — Images & Files (AWS S3)
| Feature | Details |
|---------|---------|
| File transfer | Send files over the connection |
| Image sharing | Upload to S3, share the link |
| Image understanding | Claude vision API describes shared images |

Add to Postgres schema:
```
attachments → id, message_id, file_url (S3), file_type, file_size, created_at
```

### Stage 6 — Real UI
| # | Client | Tech |
|---|--------|------|
| 1 | Desktop app | Tkinter (GUI, event loops) |
| 2 | Web client | Flask (HTTP REST API) + WebSockets (live messages) |

### Stage 7 — Publish
- Docker
- Domain name
- Production configuration

---

## Repo Structure (Monorepo)

```
musubi/                      ← GitHub repo root (Apache 2.0)
├── server/                  ← Python backend (building now)
│   ├── pyproject.toml
│   ├── server.py
│   └── client.py
├── web/                     ← web app (Stage 6, future)
├── mobile/                  ← iOS + Android via React Native (future)
├── proto/                   ← shared Protobuf definitions (Stage 5.5, optional)
└── DISCUSSION.md
```

Add `web/` and `mobile/` only when we get there — no need to create them now.

## Architecture Summary

```
┌─────────────────────────────────┐
│         AWS EC2 Server          │
│                                 │
│  server.py                      │
│  ├── PostgreSQL (users, msgs)   │
│  ├── Redis (real-time, presence)│
│  └── S3 client (image uploads)  │
└────────────────┬────────────────┘
                 │ TCP / WebSocket
     ┌───────────┴───────────┐
     │                       │
 client.py              browser
 (terminal)            (web app)
```

---

## Key Decisions
- **Invite links** instead of explicit member adds — anyone with the token can join a private channel
- **File I/O first, then Postgres** — so the migration feels meaningful, not arbitrary
- **Raw TCP first, WebSockets later** — WebSockets are browser-only; learn the foundation first
- **AWS throughout** — EC2, RDS-compatible Postgres on EC2, S3, all on free tier
- **`uv` for virtual environments** — modern standard, handles venv + packages + lockfile in one tool
- **Python over Java for interviews** — Python is shorter/cleaner for coding interviews, widely used across target companies
- **Apache 2.0 license** — better than MIT for a startup; explicit patent grant protects you as contributors join
- **App name: Musubi** — Japanese for "to connect/tie together," short and memorable
- **AI layer on by default** — AI listens to all chats/channels/DMs and surfaces action items, reminders, calendar events organically. Users can opt individual channels or DMs out
- **Plain classes → Dataclasses → Pydantic** — learn each in order so the upgrade feels meaningful
  - Stage 1: plain classes (learn `__init__`, `self`, class vs instance variables)
  - Stage 2: refactor to dataclasses (see what boilerplate disappears)
  - Stage 6: Pydantic for REST API models (adds runtime validation)
- **Type hints from the start** — modern Python practice, better for interviews
  - Always on: function parameters, return types, class attributes
  - Optional: local variables (only when type isn't obvious from context)
  - `ClassVar` only needed in dataclasses to distinguish class vs instance variables

## Python Environment Setup

Use **`uv`** (not pip or conda). It's the modern standard — handles virtual environments, package installation, and dependency locking all in one tool.

```
Without venv:                    With venv (via uv):
─────────────────                ──────────────────────────
packages install globally        packages install locally
all projects share versions      each project has its own versions
upgrading one breaks another     projects are fully isolated

Like Java's classpath mess   →   Like Maven/Gradle per-project deps
```

### Setup commands (when we start):
```bash
# Install uv (one time)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create project
uv init chat-server
cd chat-server

# Add a package (instead of pip install)
uv add <package-name>

# Run a file inside the venv
uv run server.py
```

Dependencies are saved to `pyproject.toml` (like `pom.xml` in Maven).
