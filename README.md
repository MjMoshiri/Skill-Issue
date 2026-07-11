# Skill Issue

Personal agents, skills, prompts — and the laptop setup that makes them useful.

## Laptop prerequisites

Stuff I need on a Mac before the rest of this repo is worth much. Full install steps live in [`laptop/prereq.md`](laptop/prereq.md).

| Tool | Why |
|------|-----|
| **Homebrew** | Package manager (macOS) |
| **Ghostty** | Terminal (zsh + Meslo Nerd Font for the prompt) |
| **Powerlevel10k** | Fast, pretty zsh prompt |
| **Raycast** | Launcher |
| **Rectangle** | Keyboard window snapping |
| **Grok CLI** | Grok Build coding agent |
| **Claude CLI** | Claude Code, with RTK hook in settings |
| **ctx7** | Context7 CLI / docs for agents |
| **RTK** | Token-saving CLI proxy by default (hook + shell) — **no skill** |

Quick path: follow the checklist at the bottom of [`laptop/prereq.md`](laptop/prereq.md).

## Repo layout

| Path | Contents |
|------|----------|
| [`laptop/`](laptop/) | Machine setup (`prereq.md`) |
| [`agents/`](agents/) | Agent definitions |
| [`prompts/`](prompts/) | Prompting guides and review prompts |
