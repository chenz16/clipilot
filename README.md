# CliPilot — Mobile CLI Session Manager

Manage your AI coding sessions (Claude Code, ChatGPT CLI, Codex, etc.) across personal machines and remote servers — from your phone. Each CLI session is a virtual worker you can supervise on the go.

**L2++ autonomy** — the AI does the heavy lifting, you stay in the loop. Monitor, intervene, redirect. Unlike fully autonomous agents (L3) that fire and forget, CliPilot keeps human judgment where it matters most.

## Quick Start (MVP)

The MVP is a lightweight Node.js server wrapping tmux, accessible from your phone's browser.

```bash
cd mvp
npm install
npm start
# Open http://<your-ip>:3000 on your phone
```

## Features

- **Session list** — see all running CLI sessions across machines
- **Create / Kill** — start new sessions with a folder + CLI type, kill ones that go off track
- **Live output** — real-time stdout stream via WebSocket
- **Send commands** — text input to session stdin
- **Voice control** — push-to-talk voice commands (production)
- **Host management** — add machines, auto-discover via Tailscale/mDNS (production)

## Previous Work

Built on [claude-code-plugin](https://github.com/chenz16/claude-code-plugin) — a browser-based dashboard for Claude Code sessions.

## Docs

- [Requirements & Vision](docs/requirements.md) — problem statement, goals, autonomy levels
- [Architecture](docs/architecture.md) — system design, API spec, repo structure
- [Roadmap](docs/roadmap.md) — MVP steps, production phases, related projects
