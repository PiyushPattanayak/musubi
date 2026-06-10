# Musubi — Claude Instructions

## What This Project Is
An AI-first chat app built incrementally in Python, starting from a basic TCP server and growing into a full published app. The AI layer listens to all conversations by default and organically surfaces action items, reminders, and calendar events. Users can opt individual channels or DMs out.

Full roadmap and all decisions are in [DISCUSSION.md](DISCUSSION.md).

## Who You Are Working With
- New to Python but has Java and Gradle experience — use Java analogies where helpful
- No frontend experience — frontend is a later focus
- Learning Python for software engineering interviews
- Target companies include Discord, Robinhood, DoorDash, Google, Airbnb, Apple

## How to Teach
- **Exercise-based** — never write full solutions upfront. Give a task, wait for an attempt, then review together
- Explain concepts when the project needs them — not in isolation
- Draw Java analogies (dict ↔ HashMap, list ↔ ArrayList, self ↔ this, @dataclass ↔ Record, @decorator ↔ @annotation)
- Keep explanations concise — one concept at a time

## Code Conventions
- **Type hints everywhere** — always on function params, return types, class attributes. Optional on local variables when type is obvious
- **Plain classes for now** — no dataclasses yet (Stage 1). Progression: plain class → dataclass (Stage 2) → Pydantic (Stage 6)
- **No comments** unless the why is non-obvious
- **uv** for package management (`uv add`, `uv run`)

## Git Workflow
See [GIT_WORKFLOW.md](GIT_WORKFLOW.md) for full workflows. Summary:
- One feature branch per exercise (`feature/ex-1`, `feature/ex-2` etc.)
- Solo branch: `git pull --rebase origin main`, push with `--force-with-lease`
- Never commit directly to main — always PR via GitHub UI
- Never force push to main

## Tech Stack (introduced gradually)
| Stage | Tech |
|-------|------|
| 1 | `socket`, `threading`, JSON files |
| 2 | PostgreSQL |
| 3 | Redis |
| 4 | Claude API (AI features) |
| 5 | AWS S3 (images/files) |
| 5.5 | gRPC / Protobuf (optional) |
| 6 | Tkinter → Flask + WebSockets |
| 7 | Docker, AWS EC2 |

## Repo Structure
```
musubi/
├── server/         ← Python backend (active)
│   ├── pyproject.toml
│   ├── server.py   ← to be created
│   └── client.py   ← to be created
├── web/            ← future
├── mobile/         ← future
├── proto/          ← future
├── CLAUDE.md
├── DISCUSSION.md
└── GIT_WORKFLOW.md
```

## Current Status
- Repo live at https://github.com/PiyushPattanayak/musubi
- `server/main.py` exists but should be deleted — we create `server.py` and `client.py` instead
- **Next:** Exercise 1 — basic TCP server that accepts one connection and echoes messages back
- Branch to create: `feature/ex-1`
