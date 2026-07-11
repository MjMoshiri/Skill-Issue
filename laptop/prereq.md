# Laptop prerequisites

What I want. Agents can figure out installs; below is the desired shape.

## Must have

- Homebrew (macOS)
- Ghostty
  - font: `MesloLGS NF`
  - font-size: 14
  - shell: zsh (`command = /bin/zsh`)
- Powerlevel10k — see [settings](#powerlevel10k)
- Raycast
- Rectangle
- Grok CLI
- Claude CLI — see [RTK proxy](#rtk-proxy-by-default)
- ctx7 (Context7 CLI)
- RTK — proxy by default, **no skill** — see [RTK proxy](#rtk-proxy-by-default)

## Powerlevel10k

Rainbow / powerline style (colorful segment backgrounds). Nerd Font complete icons.

**Left**

- line 1: `os_icon` · `dir` · `vcs`
- line 2: `prompt_char` (`❯`)

**Right** (show what’s useful; keep at least these on)

- `status` · `command_execution_time` · `background_jobs`
- lang/env when relevant: `node_version` · `go_version` · `rust_version` · `package` · `virtualenv` / `pyenv` etc.
- `time` — format `%H:%M:%S`

**Behavior**

- multiline with solid gap: `POWERLEVEL9K_MULTILINE_FIRST_PROMPT_GAP_CHAR='─'`
- `POWERLEVEL9K_PROMPT_ADD_NEWLINE=true`
- `POWERLEVEL9K_TRANSIENT_PROMPT=always` (collapse old prompts in scrollback)
- `POWERLEVEL9K_INSTANT_PROMPT=quiet`
- `POWERLEVEL9K_MODE=nerdfont-complete`
- `POWERLEVEL9K_ICON_PADDING=none`

Config lives in `~/.p10k.zsh` after `p10k configure` (or copy an existing one and tweak).

## RTK proxy (by default)

Transparent rewrite only. **Do not** install RTK skill / `RTK.md` / `@RTK.md` in Claude config.

### Agent (Claude Code)

In `~/.claude/settings.json`, PreToolUse on Bash:

```json
{
  "matcher": "Bash",
  "hooks": [
    {
      "type": "command",
      "command": "rtk hook claude"
    }
  ]
}
```

Merge with any other hooks; don’t wipe them.

### Interactive shell (zsh)

On Enter, rewrite supported commands via `rtk rewrite` (e.g. `git status` → `rtk git status`). Bypass: `command …` or `\…`.

```zsh
# RTK transparent command proxy (interactive shell)
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

If `rtk init -g` creates skill/docs files, delete them and keep only the hook + shell proxy.
