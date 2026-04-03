# TermPilot — Mobile CLI Session Manager

A mobile app to discover, create, monitor, and kill CLI coding sessions (Claude Code, ChatGPT CLI, Codex, etc.) on local and remote machines — like a team manager dashboard for your terminal windows.

## Core Features

### 1. Session Discovery
Scan local network + Tailscale network to discover running CLI processes (`claude`, `chatgpt`, `codex`, `bash`, `zsh`) on all machines. Each machine runs a lightweight agent daemon.

### 2. Session Lifecycle
Create / attach / detach / kill any session from mobile. Session creation requires specifying a working directory (folder picker, remote machines use agent API for directory listing). Uses `tmux` or `screen` for session persistence.

### 3. Live Feed
Real-time stdout tail for each session via WebSocket stream. A mini read-only terminal on mobile with support for sending simple commands.

### 4. Host Management
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
| 6 | **Auto-Discovery** | Tailscale + mDNS discovery (last priority) |
