# CliPilot — Architecture (Production)

## System Overview

```
┌──────────────┐         WebSocket / HTTPS
│  Mobile App  │ <-------------------------------> ┌──────────────┐
│ (React Native│                                   │ Agent Daemon  │  <- one per machine
│   Expo)      │         Tailscale / LAN           │ (Go binary)   │
└──────────────┘                                   └──────────────┘
                                                      │
                                                      ├─ tmux API (create/attach/kill)
                                                      ├─ process scanner (ps aux)
                                                      ├─ stdout streamer (WebSocket)
                                                      └─ folder browser (ls API)
```

## Components

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

## Production Repo Structure

```
clipilot/
├── agent/                  <- Go daemon (single binary, brew/curl install)
│   ├── main.go
│   ├── tmux.go             <- tmux session management
│   ├── scanner.go          <- process scanning
│   ├── stream.go           <- WebSocket stdout streaming
│   └── folders.go          <- remote directory browsing
├── app/                    <- React Native (Expo)
│   ├── screens/
│   │   ├── HostList/
│   │   ├── SessionList/
│   │   ├── SessionDetail/
│   │   ├── CreateSession/
│   │   └── LiveTerminal/
│   └── services/
│       ├── discovery.ts    <- Tailscale + mDNS
│       └── agentClient.ts  <- REST + WS client
├── docs/
├── install.sh              <- agent one-liner install script
└── README.md
```
