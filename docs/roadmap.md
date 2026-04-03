# CliPilot — Roadmap

## MVP (Entry-Level)

A minimal web-based dashboard accessible from mobile browser. No native app, no Go daemon — just a Node.js server wrapping tmux on localhost.

| Step | What | Details |
|---|---|---|
| 1 | tmux wrapper server | Node.js + Express, list/create/kill tmux sessions via REST API |
| 2 | WebSocket stdout stream | Pipe `tmux capture-pane` output to browser via WebSocket |
| 3 | Mobile-friendly web UI | Simple responsive HTML/JS dashboard, works on phone browser |
| 4 | Send input | Text box to send commands to a session's stdin |

**Inspired by:** [muxplex](https://github.com/bkrabach/muxplex), [webmux](https://github.com/nooesc/webmux), [claude-code-monitor](https://github.com/onikan27/claude-code-monitor)

## Production

Full mobile app + Go agent daemon across multiple machines.

| Phase | Milestone | Description |
|---|---|---|
| 1 | **Agent Daemon** | Go binary: tmux wrapper + REST API + `/api/health` |
| 2 | **Host List** | Mobile screen to manually add hosts by IP, show online/offline |
| 3 | **Session List** | Fetch and display sessions from agent API |
| 4 | **Create / Kill** | Specify folder + CLI type to create session; kill existing ones |
| 5 | **Live Terminal** | WebSocket stdout tail view on mobile |
| 6 | **Voice Control** | Push-to-talk voice input + optional TTS readback |
| 7 | **Auto-Discovery** | Tailscale + mDNS discovery (last priority) |

## Related Open Source Projects

| Project | Description |
|---|---|
| [muxplex](https://github.com/bkrabach/muxplex) | Web-based tmux session dashboard, mobile-friendly, live session grid |
| [webmux](https://github.com/nooesc/webmux) | Web tmux viewer with xterm.js, Vue 3, Tailwind CSS |
| [claude-code-monitor](https://github.com/onikan27/claude-code-monitor) | Real-time dashboard with CLI + mobile web UI and QR code access |
| [ccmanager](https://github.com/kbwo/ccmanager) | Multi-agent session manager for Claude Code, Gemini CLI, Codex CLI |
| [claude-code-dashboard](https://github.com/Stargx/claude-code-dashboard) | Lightweight localhost dashboard for multi-session monitoring |
| [claudecodeui](https://github.com/siteboon/claudecodeui) | Open-source web UI for managing Claude Code sessions remotely |
