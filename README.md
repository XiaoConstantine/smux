# smux

One-command tmux setup with terminal automation for AI agents.

- **For you** — keyboard-driven tmux config with Option-key bindings, mouse support, and pane labels
- **For agents** — `tmux-bridge` CLI lets any agent read, type, and send keys to any pane
- **Agent-to-agent** — Claude Code can prompt Codex in the next pane, and Codex replies back. Any agent that can run bash can participate.

```bash
tmux-bridge msgsend codex "review src/auth.ts"  # send message with header + Enter
tmux-bridge send worker "y"                      # type + Enter in one command
tmux-bridge list --json                          # machine-readable pane list
tmux-bridge batch-msgsend "start task" codex cc amp  # fan-out to multiple agents
```

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/XiaoConstantine/smux/main/install.sh | bash
```

This installs:
- **tmux** if not already installed (via Homebrew, apt, dnf, pacman, or apk)
- **tmux.conf** with Option-key bindings, mouse support, pane labels, and a minimal status bar
- **tmux-bridge** CLI for cross-pane agent communication

Everything lives in `~/.smux/`.

## Keybindings

All keybindings use **Option (Alt)** with no prefix required.

### Panes

| Key | Action |
|---|---|
| `Option+i/k/j/l` | Navigate up/down/left/right (no wrap) |
| `Option+n` | New pane (split + auto-tile) |
| `Option+w` | Close pane |
| `Option+o` | Cycle layouts |
| `Option+g` | Mark pane |
| `Option+y` | Swap with marked pane |

### Windows

| Key | Action |
|---|---|
| `Option+m` | New window |
| `Option+u` | Next window |
| `Option+h` | Previous window |

### Scrolling

| Key | Action |
|---|---|
| `Option+Tab` | Toggle scroll mode |
| `i/k` | Scroll up/down |
| `Shift+I/K` | Half-page up/down |
| `q` or `Escape` | Exit scroll mode |

### Mouse

- Click to select panes
- Drag to select text (auto-copies to clipboard)
- Scroll wheel to scroll

## tmux-bridge

A CLI for cross-pane communication. Any tool that can run bash can use it — Claude Code, Codex, Gemini CLI, or a plain shell script.

### Commands

| Command | Description |
|---|---|
| `tmux-bridge list [--json\|--tsv]` | Show all panes (table, JSON, or TSV) |
| `tmux-bridge send <target> <text>` | Type text and press Enter |
| `tmux-bridge msgsend <target> <text>` | Send message with sender info and press Enter |
| `tmux-bridge batch-send <text> <t1> [t2...]` | Send text to multiple targets |
| `tmux-bridge batch-msgsend <text> <t1> [t2...]` | Send message to multiple targets |
| `tmux-bridge type <target> <text>` | Type text (no Enter) |
| `tmux-bridge message <target> <text>` | Type with sender info (no Enter) |
| `tmux-bridge read <target> [lines]` | Read last N lines from a pane |
| `tmux-bridge keys <target> <key>...` | Send keys (Enter, Escape, C-c, etc.) |
| `tmux-bridge name <target> <label>` | Label a pane for easy addressing |
| `tmux-bridge resolve <label>` | Look up a pane by label |
| `tmux-bridge id` | Print this pane's ID |

### Environment Variables

| Variable | Description |
|---|---|
| `TMUX_BRIDGE_SOCKET` | Override tmux server socket path (skips auto-detection) |
| `TMUX_BRIDGE_HEADER_CACHE` | Pre-built message header (skips 2 tmux display-message calls) |

### Performance Tips

For high-frequency callers (e.g., orchestrators running many bridge calls):

```bash
# Skip socket auto-detection on every call
export TMUX_BRIDGE_SOCKET="${TMUX%%,*}"

# Cache the message header to avoid 2 tmux calls per msgsend
export TMUX_BRIDGE_HEADER_CACHE="$(tmux-bridge msgsend --dry-run 2>/dev/null || true)"

# Use pane IDs (%N) directly instead of labels to skip resolve
tmux-bridge msgsend %5 "hello"    # faster than: tmux-bridge msgsend codex "hello"

# Fan-out to multiple agents in one process
tmux-bridge batch-msgsend "start task" codex cc amp
```

See the [smux skill](skills/smux/SKILL.md) for full documentation on agent-to-agent workflows.

## Update

```bash
smux update
```

## Uninstall

```bash
smux uninstall
```

## AI Agent Skills

Install the smux skill to teach your agents how to use tmux-bridge:

```bash
npx skills add XiaoConstantine/smux
```

Works with Claude Code, Codex, Cursor, Copilot, and [40+ other agents](https://skills.sh).

## Requirements

- macOS (requires [Homebrew](https://brew.sh)) or Linux
- tmux 3.2+ (installed automatically)
