# CliPilot — Mobile CLI Session Manager

## Problem Statement

AI-powered CLI tools like Claude Code, ChatGPT CLI, and Codex are turning every terminal window into an autonomous worker — writing code, running tests, and deploying fixes with minimal human input. Power users now run multiple sessions across personal machines and remote servers simultaneously. But there is no unified way to oversee them. You are stuck at your desk, switching between terminal tabs, SSH-ing into machines, and losing track of which session is doing what. If you step away from your computer, those virtual workers run blind with no visibility, no control, and no voice.

## Goal

**Short-term:** Build a mobile app that lets CLI users manage all their terminal sessions — across personal computers and remote servers — from their phone. Discover running sessions, create new ones, monitor live output, send commands (by text or voice), and kill sessions that go off track. Each CLI session is treated as a **virtual worker** that you can supervise on the go.

**Long-term:** Evolve into a **virtual workforce dashboard** — a single pane of glass where each CLI session is a worker with a name, a task, a status, and a performance history. Assign work, review output, get alerts, and manage your fleet of AI coding agents the same way a manager oversees a team.

## How CliPilot Differs from Fully Autonomous AI Agents

Think of it like autonomous driving levels:

| Level | Analogy | Example | Human Role |
|---|---|---|---|
| **L2** | Hands on wheel | Traditional terminal + manual commands | Fully manual |
| **L2++ — CliPilot** | Hands off, eyes on road | AI CLI sessions with human oversight via mobile | Supervise, intervene, redirect |
| **L3** | Fully autonomous | OpenClaw, Devin, fully autonomous agents | Fire and forget |

**Fully autonomous agents (L3)** like OpenClaw dispatch tasks and expect deliverables — no process visibility, no mid-course correction. This works for well-defined, low-risk tasks but carries real risk: the agent may go off track, burn tokens, or deliver something unusable with no chance to intervene.

**CliPilot operates at L2++** — the AI does the heavy lifting, but the human stays in the loop. You monitor live output, provide feedback mid-task, redirect when something goes wrong, and approve before critical actions. For tasks that require multiple rounds of human judgment and iteration, this supervised approach produces more reliable deliverables than full autonomy.

**Both models have a market.** L3 excels at batch, repeatable, well-scoped work. L2++ excels at complex, exploratory, high-stakes work where the cost of a bad deliverable is high. CliPilot starts with L2++ because most real-world AI coding work today demands human context that no prompt can fully capture.

**Long-term, both need management.** Even fully autonomous L3 agents need oversight — tracking what they are working on, reviewing deliverables, catching failures, and reallocating work. The management problem doesn't disappear with more autonomy; it changes shape. CliPilot's vision is to become the unified dashboard for managing AI workers at every autonomy level — whether they need constant supervision or just periodic check-ins.

## Previous Work

This project builds on [claude-code-plugin](https://github.com/chenz16/claude-code-plugin) — a browser-based dashboard for monitoring and managing Claude Code CLI sessions. CliPilot takes the concept mobile-first and expands it to support multiple CLI tools across multiple machines.

## Core Features

### 1. Session Discovery
Scan local network + Tailscale network to discover running CLI processes (`claude`, `chatgpt`, `codex`, `bash`, `zsh`) on all machines. Each machine runs a lightweight agent daemon.

### 2. Session Lifecycle
Create / attach / detach / kill any session from mobile. Session creation requires specifying a working directory (folder picker, remote machines use agent API for directory listing). Uses `tmux` or `screen` for session persistence.

### 3. Live Feed
Real-time stdout tail for each session via WebSocket stream. A mini read-only terminal on mobile with support for sending simple commands.

### 4. Voice Control
Push-to-talk voice input to send commands to active sessions. Uses on-device speech recognition (`expo-speech-recognition` wrapping iOS `SFSpeechRecognizer`) for low-latency, free STT. Optional TTS (`expo-speech`) to read back terminal output. Falls back to OpenAI Whisper API for higher accuracy or multilingual needs.

### 5. Host Management
Add/remove machines. Supports Tailscale auto-discovery (`tailscale status --json`) and local mDNS/Bonjour scanning. Each host shows online/offline status, OS, and resource usage.

## Architecture

```
┌──────────────┐         WebSocket / HTTPS
│  Mobile App  │ ◄──────────────────────────────►  ┌──────────────┐
│ (React Native│                                   │ Agent Daemon  │  ← one per machine
│   Expo)      │         Tailscale / LAN           │ (Go binary)   │
└──────────────┘                                   └──────────────┘
                                                      │
                                                      ├─ tmux API (create/attach/kill)
                                                      ├─ process scanner (ps aux)
                                                      ├─ stdout streamer (WebSocket)
                                                      └─ folder browser (ls API)
```

### Components

| Component | Tech | Responsibility |
|---|---|---|
| **Mobile App** | React Native (Expo) | UI, session management, host list, live terminal view |
| **Agent Daemon** | Go single binary | Runs on each workstation, exposes REST + WebSocket API, manages tmux sessions |
| **Discovery** | Tailscale API + mDNS | Auto-discover agents on LAN and Tailscale network |

## Agent Daemon API

```
GET    /api/health                — host status, OS, cpu, mem
GET    /api/sessions              — list all CLI sessions (tmux ls + process info)
POST   /api/sessions              — create session { type, folder, cmd }
GET    /api/sessions/:id          — session detail + resource usage
DELETE /api/sessions/:id          — kill session
GET    /api/sessions/:id/stream   — WebSocket, tail stdout
POST   /api/sessions/:id/input   — send command to session stdin
GET    /api/folders?path=...      — browse directories for folder picker
```

## Repo Structure

```
clipilot/
├── agent/                  ← Go daemon (single binary, brew/curl install)
│   ├── main.go
│   ├── tmux.go             ← tmux session management
│   ├── scanner.go          ← process scanning
│   ├── stream.go           ← WebSocket stdout streaming
│   └── folders.go          ← remote directory browsing
├── app/                    ← React Native (Expo)
│   ├── screens/
│   │   ├── HostList/
│   │   ├── SessionList/
│   │   ├── SessionDetail/
│   │   ├── CreateSession/
│   │   └── LiveTerminal/
│   └── services/
│       ├── discovery.ts    ← Tailscale + mDNS
│       └── agentClient.ts  ← REST + WS client
├── install.sh              ← agent one-liner install script
└── README.md
```

## MVP Roadmap

| Phase | Milestone | Description |
|---|---|---|
| 1 | **Agent Daemon** | `tmux` wrapper + REST API + `/api/health` endpoint |
| 2 | **Host List** | Mobile screen to manually add hosts by IP, show online/offline |
| 3 | **Session List** | Fetch and display sessions from agent API |
| 4 | **Create / Kill** | Specify folder + CLI type to create session; kill existing ones |
| 5 | **Live Terminal** | WebSocket stdout tail view on mobile |
| 6 | **Voice Control** | Push-to-talk voice input + optional TTS readback |
| 7 | **Auto-Discovery** | Tailscale + mDNS discovery (last priority) |
