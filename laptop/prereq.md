# Laptop prerequisites

Fresh Mac setup for this workflow: terminal, window management, coding agents, token-saving proxy.

## 1. Homebrew (macOS)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Then put brew on PATH (Apple Silicon):

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Skip on Linux/Windows — use each tool’s native installer instead.

## 2. Ghostty

GPU terminal. Use zsh + a Nerd Font so Powerlevel10k glyphs render.

```bash
brew install --cask ghostty
brew install font-meslo-lg-nerd-font
```

Create `~/.config/ghostty/config` (and mirror under
`~/Library/Application Support/com.mitchellh.ghostty/config` if Ghostty only reads App Support):

```
# Powerlevel10k needs a Nerd Font for icons / powerline glyphs
font-family = MesloLGS NF
font-size = 14

# Use zsh so Powerlevel10k loads
command = /bin/zsh
```

If your login shell is still bash and `chsh` needs a password, force interactive bash into zsh:

```bash
# ~/.bash_profile and/or ~/.bashrc
if [[ $- == *i* ]] && [ -x /bin/zsh ]; then
  exec /bin/zsh
fi
```

Optional later: `chsh -s /bin/zsh` (requires your password).

## 3. Powerlevel10k

```bash
brew install powerlevel10k
```

In `~/.zshrc` (instant prompt near the top, theme after PATH):

```zsh
# Enable Powerlevel10k instant prompt (near top of ~/.zshrc)
if [[ -r "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh" ]]; then
  source "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh"
fi

# … PATH / brew shellenv …

source /opt/homebrew/share/powerlevel10k/powerlevel10k.zsh-theme
[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh
```

Configure once:

```bash
p10k configure
```

Recommended: **Rainbow** style, Meslo font, include `os_icon`, `prompt_char`, `time`, language versions as you like. Re-run `p10k configure` anytime, or edit `~/.p10k.zsh`.

## 4. Raycast

Launcher / Spotlight replacement.

```bash
brew install --cask raycast
```

Open Raycast, grant Accessibility/Screen Recording if prompted, set as default launcher if you want.

## 5. Rectangle

Keyboard window snapping (free Spectacle successor).

```bash
brew install --cask rectangle
```

Open Rectangle and grant Accessibility so shortcuts work.

## 6. Grok CLI (Grok Build)

```bash
curl -fsSL https://x.ai/cli/install.sh | bash
```

Sign in when prompted. Needs SuperGrok or X Premium Plus.

Docs: https://x.ai/cli

## 7. Claude CLI (Claude Code)

```bash
# Official installer (or brew if you prefer a package path you already use)
curl -fsSL https://claude.ai/install.sh | bash
# Alternative: npm i -g @anthropic-ai/claude-code
```

Sign in (`claude` / `claude auth`).

### Settings we use (`~/.claude/settings.json`)

Keep your own prefs; the important integration for this setup is the **RTK PreToolUse hook** (proxy Bash through RTK — no skill file):

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "rtk hook claude"
          }
        ]
      }
    ]
  }
}
```

Notes:

- Merge this into existing `hooks.PreToolUse` if you already have other hooks (e.g. claude-tabs). Do not wipe other matchers.
- **Do not** rely on `RTK.md` / `@RTK.md` in `CLAUDE.md`. Proxy-only: hook rewrites Bash; no RTK skill.
- Restart Claude Code after changing hooks.

Optional extras already used in this environment (not required for prereq): agent teams env, bypass permissions, plugins (`context7`, etc.). Add only if you want them.

## 8. Context7 CLI (`ctx7`)

Up-to-date library docs for agents.

```bash
npm install -g ctx7@latest
# or one-shot: npx ctx7@latest <command>
```

Node 18+.

```bash
ctx7 setup
# or non-interactive MCP mode for Claude:
ctx7 setup --mcp --claude
```

Prefer **MCP** (or the official Claude plugin) so the agent gets tools; CLI is available either way for manual `ctx7` queries.

Docs: https://context7.com/docs/clients/cli

## 9. RTK (proxy by default — no skill)

Rust Token Killer: compresses noisy CLI output so agents (and you) burn fewer tokens.

### Install

```bash
brew install rtk
# or: brew install rtk-ai/tap/rtk
# or: curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh
```

Verify:

```bash
rtk --version
```

### Proxy by default — agent (Claude)

**Do not install the RTK skill / awareness markdown.**

Either:

1. Manually add the PreToolUse hook above (`rtk hook claude`), **or**
2. Run `rtk init -g` then **remove** skill-style files if created:

```bash
# After init, strip skill/docs injection if present:
rm -f ~/.claude/RTK.md
# Remove any @RTK.md line from ~/.claude/CLAUDE.md / Claude.md
```

Goal: only the **hook** remains — transparent rewrite of Bash tool calls. No skill instructions for the model to “remember.”

Confirm:

```bash
rtk init --show   # or inspect ~/.claude/settings.json for "rtk hook claude"
```

### Proxy by default — interactive shell (zsh)

Append to `~/.zshrc` so normal commands are rewritten on Enter (`git status` → `rtk git status` when supported):

```zsh
# RTK transparent command proxy (interactive shell)
# Rewrites supported commands before they run.
# No skill/aliases — uses `rtk rewrite` as the source of truth.
# Bypass one command:  command git status   or   \git status
if (( $+commands[rtk] )) && [[ -o interactive ]]; then
  function rtk-accept-line() {
    if [[ -n "$BUFFER" && "$BUFFER" != rtk\ * && "$BUFFER" != \\* && "$BUFFER" != command\ * ]]; then
      local rewritten
      rewritten=$(rtk rewrite "$BUFFER" 2>/dev/null) || true
      if [[ -n "$rewritten" && "$rewritten" != "$BUFFER" ]]; then
        BUFFER="$rewritten"
      fi
    fi
    zle .accept-line
  }
  zle -N accept-line rtk-accept-line
fi
```

Reload: `source ~/.zshrc` or open a new Ghostty tab.

### Useful RTK commands

```bash
rtk rewrite 'git status'   # see what would run
rtk gain                   # token savings stats
rtk discover               # what commands RTK knows
```

Repo: https://github.com/rtk-ai/rtk

---

## Quick install block (macOS)

```bash
# Core
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
eval "$(/opt/homebrew/bin/brew shellenv)"

brew install --cask ghostty raycast rectangle
brew install font-meslo-lg-nerd-font powerlevel10k rtk

# Agents
curl -fsSL https://x.ai/cli/install.sh | bash
curl -fsSL https://claude.ai/install.sh | bash   # or your preferred Claude install path
npm install -g ctx7@latest

# Then: Ghostty font/shell config, p10k configure, Claude RTK hook (no skill),
# zsh accept-line RTK proxy, ctx7 setup --mcp
```

## Checklist

- [ ] Homebrew
- [ ] Ghostty + MesloLGS NF + `command = /bin/zsh`
- [ ] Powerlevel10k + `~/.p10k.zsh`
- [ ] Raycast
- [ ] Rectangle
- [ ] Grok CLI signed in
- [ ] Claude CLI signed in + `rtk hook claude` in settings
- [ ] ctx7 installed (+ setup)
- [ ] RTK installed, shell proxy on, **no** RTK skill / `RTK.md`
