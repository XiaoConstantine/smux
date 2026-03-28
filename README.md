# smux

A tmux setup that gives you and your AI agents the same interface to the terminal. One command installs tmux with a keyboard-driven config, and **tmux-bridge** — a CLI that lets any agent read, type, and send keys to any tmux pane.

This means agents can use your terminal the way you do. They can spin up panes, run commands, read output, and interact with other running processes. More importantly, agents can talk to *each other* — Claude Code can type a message into a pane running Codex, and Codex can reply back. Any agent that can run bash can participate.

**How it works:** tmux-bridge gives agents atomic terminal operations — `read` a pane's output, `type` text into it, `keys` to press Enter or Ctrl+C. Agents label their panes (`tmux-bridge name %3 claude`) and address each other by name. A built-in read guard forces agents to look before they act, preventing blind input. Messages are framed with `[tmux-bridge from:claude]` so the receiving agent knows who sent it and where to reply.

```bash
# Agent A reads a pane, types a message, verifies, and sends
tmux-bridge read codex 20
tmux-bridge type codex '[tmux-bridge from:claude] Review src/auth.ts for security issues'
tmux-bridge read codex 20
tmux-bridge keys codex Enter
# Codex receives the message and replies back into Claude's pane
```

https://github.com/user-attachments/assets/9d5463ba-5972-4bbd-a07e-b585f1178011

## Install

```bash
curl -fsSL shawnpana.com/smux/install.sh | bash
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

| Command | Description |
|---|---|
| `tmux-bridge list` | Show all panes with target, process, label |
| `tmux-bridge read <target> [lines]` | Read last N lines from a pane |
| `tmux-bridge type <target> <text>` | Type text into a pane (no Enter) |
| `tmux-bridge keys <target> <key>...` | Send keys (Enter, Escape, C-c, etc.) |
| `tmux-bridge name <target> <label>` | Label a pane for easy addressing |
| `tmux-bridge resolve <label>` | Look up a pane by label |
| `tmux-bridge id` | Print this pane's ID |

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
npx skills add ShawnPana/smux
```

Works with Claude Code, Codex, Cursor, Copilot, and [40+ other agents](https://skills.sh).

## Requirements

- macOS (requires [Homebrew](https://brew.sh)) or Linux
- tmux 3.2+ (installed automatically)
