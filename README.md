# MLGA — Make Laptop Great Again

Personal collection of Claude Code agents, prompts, and global policies. Tuned for Claude Opus 4.7.

## Layout

```
CLAUDE.md            global policies (root_cause, verification, research, comments, ...)
agents/              subagent definitions
prompts/             attachable prompt references and review guides
```

## Index

### Global

| File | Purpose | When |
|------|---------|------|
| `CLAUDE.md` | Universal policies loaded into every session. | Always (Claude Code reads it automatically). |

### Agents

| File | Spawn when | Tools |
|------|-----------|-------|
| `agents/library-researcher.md` | Before using a library/API not verified this session (new dep, version bump, unfamiliar signature). Returns findings inline. | Context7 MCP, Read, Grep, WebFetch |
| `agents/mj-rewrite.md` | Rewriting text into MJ's voice — direct, no fluff. Accepts a file path or raw text. | Edit, Write, Read |

### Prompt references (attach per task)

| File | Target | Attach when |
|------|--------|-------------|
| `prompts/prompting-guide.md` | Claude Opus 4.7 | Rewriting a prompt to be 4.7-shaped. The meta-guide. |
| `prompts/anthropic-prompt-design.md` | Claude Sonnet/Opus | Building a customer-support agent system prompt. |
| `prompts/codex-prompt-writing-guide.md` | OpenAI Codex (uses `AGENTS.md`, not Claude) | Writing a one-off Codex work order. |
| `prompts/gemini-prompt-design.md` | Google Gemini | Writing a Gemini prompt. Generic prompt-engineering reference otherwise. |
| `prompts/LOGICAL_REVIEW.md` | Claude Opus 4.7 | Reviewing LLM-written code for correctness. Pair with `QUALITY_REVIEW.md`. |
| `prompts/QUALITY_REVIEW.md` | Claude Opus 4.7 | Reviewing LLM-written code for idiom/syntax. Pair with `LOGICAL_REVIEW.md`. |

## Conventions

- Prompts and agents are written in XML where they will be attached to a Claude session — the model parses tags more reliably than markdown headers when content mixes instructions, examples, and context.
- Effort and adaptive-thinking parameters belong in the API call, not the prompt. See `prompts/prompting-guide.md` (`effort_and_adaptive_thinking`).
- Aggressive language ("CRITICAL", "MUST", caps) is avoided — Opus 4.7 overtriggers on it.
