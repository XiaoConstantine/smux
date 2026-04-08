---
name: smux
description: Control tmux panes and communicate between AI agents. Use this skill whenever the user mentions tmux panes, cross-pane communication, sending messages to other agents, reading other panes, managing tmux sessions, or interacting with processes running in tmux. Includes tmux-bridge CLI for agent-to-agent messaging and raw tmux commands for direct session control.
metadata:
  { "openclaw": { "emoji": "🖥️", "os": ["darwin", "linux"], "requires": { "bins": ["tmux", "tmux-bridge"] } } }
---

# smux

Tmux pane control and cross-pane agent communication. Use `tmux-bridge` (the high-level CLI) for all cross-pane interactions. Fall back to raw tmux commands only when you need low-level control.

## tmux-bridge — Cross-Pane Communication

A CLI that lets any AI agent interact with any other tmux pane. Works via plain bash. Prefer the fast-path commands: `send` types text and presses Enter, `msgsend` adds sender info and presses Enter. The low-level commands remain available: `type` types text without Enter, `keys` sends special keys, `read` captures pane content.

### DO NOT WAIT OR POLL

Other panes have agents that will reply to you via tmux-bridge. Their reply appears directly in YOUR pane as a `[tmux-bridge from:...]` message. Do not sleep, poll, read the target pane for a response, or loop. Type your message, press Enter, and move on.

The ONLY time you read a target pane is:
- **Before** using low-level `type`, `message`, or `keys` commands (enforced by the read guard)
- **After typing** to verify your text landed before pressing Enter
- When interacting with a **non-agent pane** (plain shell, running process)

### Read Guard

The CLI enforces read-before-act for the low-level `type`, `message`, and `keys` commands. Fast-path `send` and `msgsend` are single-shot helpers and do not require a separate `read`.

1. `tmux-bridge read <target>` marks the pane as "read"
2. `tmux-bridge type/message/keys <target>` checks for that mark — errors if you haven't read
3. After a successful low-level action, the mark is cleared — you must read again before the next low-level interaction

```
$ tmux-bridge type codex "hello"
error: must read the pane before interacting. Run: tmux-bridge read codex
```

### Command Reference

| Command | Description | Example |
|---|---|---|
| `tmux-bridge list` | Show all panes with target, pid, command, size, label | `tmux-bridge list` |
| `tmux-bridge list --json` | JSON array of pane objects | `tmux-bridge list --json` |
| `tmux-bridge list --tsv` | Tab-separated pane listing with header | `tmux-bridge list --tsv` |
| `tmux-bridge send <target> <text>` | Type text and press Enter | `tmux-bridge send worker "y"` |
| `tmux-bridge msgsend <target> <text>` | Fast agent-to-agent send with auto sender info and Enter | `tmux-bridge msgsend codex "review src/auth.ts"` |
| `tmux-bridge batch-send <text> <t1> [t2...]` | Send text to multiple targets in one invocation | `tmux-bridge batch-send "start" codex cc amp` |
| `tmux-bridge batch-msgsend <text> <t1> [t2...]` | Send message with header to multiple targets | `tmux-bridge batch-msgsend "begin task" codex cc amp` |
| `tmux-bridge type <target> <text>` | Type text without pressing Enter | `tmux-bridge type codex "hello"` |
| `tmux-bridge message <target> <text>` | Type text with auto sender info and reply target | `tmux-bridge message codex "review src/auth.ts"` |
| `tmux-bridge read <target> [lines]` | Read last N lines (default 50) | `tmux-bridge read codex 100` |
| `tmux-bridge keys <target> <key>...` | Send special keys | `tmux-bridge keys codex Enter` |
| `tmux-bridge name <target> <label>` | Label a pane (visible in tmux border) | `tmux-bridge name %3 codex` |
| `tmux-bridge resolve <label>` | Print pane target for a label | `tmux-bridge resolve codex` |
| `tmux-bridge id` | Print this pane's ID | `tmux-bridge id` |

### Target Resolution

Targets can be:
- **tmux native**: `session:window.pane` (e.g. `shared:0.1`), pane ID (`%3`), or window index (`0`)
- **label**: Any string set via `tmux-bridge name` — resolved automatically

When you run `tmux-bridge` inside tmux, label lookup prefers panes in your current tmux session. If a label is ambiguous, use the pane ID (`%N`) or a fully qualified tmux target instead of relying on the label.

### Fast Path

For normal one-shot submissions, prefer the single-command helpers:

**Sending a message to an agent:**
```bash
tmux-bridge msgsend codex 'Please review src/auth.ts'
# STOP. Do NOT read codex for a reply. The agent replies into YOUR pane.
```

**Approving a prompt (non-agent pane):**
```bash
tmux-bridge send worker "y"
tmux-bridge read worker 20
```

**Fan-out to multiple agents:**
```bash
tmux-bridge batch-msgsend 'Start working on the task' codex cc amp
```

Use the low-level **read → act → read** flow only when you need to inspect the pane before typing, verify the typed text before pressing Enter, or send keys other than Enter.

### Messaging Convention

The `message` and `msgsend` commands auto-prepend sender info:

```
[tmux-bridge from:claude pane:%4 at:3:0.0 - load the smux skill to reply] Please review src/auth.ts
```

The receiver gets: who sent it (`from`), the exact pane to reply to (`pane`), and the session/window location (`at`). When you see this header, reply using tmux-bridge to the pane ID from the header.

### Agent-to-Agent Workflow

```bash
# 1. Label yourself
tmux-bridge name "$(tmux-bridge id)" claude

# 2. Discover other panes
tmux-bridge list

# 3. Send a message
tmux-bridge msgsend codex 'Please review the changes in src/auth.ts'
```

### Example Conversation

**Agent A (claude) sends:**
```bash
tmux-bridge msgsend codex 'What is the test coverage for src/auth.ts?'
```

**Agent B (codex) sees in their prompt:**
```
[tmux-bridge from:claude pane:%4 at:3:0.0 - load the smux skill to reply] What is the test coverage for src/auth.ts?
```

**Agent B replies using the pane ID from the header:**
```bash
tmux-bridge msgsend %4 '87% line coverage. Missing the OAuth refresh token path (lines 142-168).'
```

### Environment Variables

| Variable | Description |
|---|---|
| `TMUX_BRIDGE_SOCKET` | Override tmux server socket path (skips auto-detection) |
| `TMUX_BRIDGE_HEADER_CACHE` | Pre-built message header (skips 2 tmux display-message calls) |

---

## Raw tmux Commands

Use these when you need direct tmux control beyond what tmux-bridge provides — session management, window navigation, creating panes, or low-level scripting.

### Capture Output

```bash
tmux capture-pane -t shared -p | tail -20    # Last 20 lines
tmux capture-pane -t shared -p -S -          # Entire scrollback
tmux capture-pane -t shared:0.0 -p           # Specific pane
```

### Send Keys

```bash
tmux send-keys -t shared -l -- "text here"   # Type text (literal mode)
tmux send-keys -t shared Enter               # Press Enter
tmux send-keys -t shared Escape              # Press Escape
tmux send-keys -t shared C-c                 # Ctrl+C
tmux send-keys -t shared C-d                 # Ctrl+D (EOF)
```

For interactive TUIs, split text and Enter into separate sends:
```bash
tmux send-keys -t shared -l -- "Please apply the patch"
sleep 0.1
tmux send-keys -t shared Enter
```

### Panes and Windows

```bash
# Create panes (prefer over new windows)
tmux split-window -h -t SESSION              # Horizontal split
tmux split-window -v -t SESSION              # Vertical split
tmux select-layout -t SESSION tiled          # Re-balance

# Navigate
tmux select-window -t shared:0
tmux select-pane -t shared:0.1
tmux list-windows -t shared
```

### Session Management

```bash
tmux list-sessions
tmux new-session -d -s newsession
tmux kill-session -t sessionname
tmux rename-session -t old new
```

### Claude Code Patterns

```bash
# Check if session needs input
tmux capture-pane -t worker-3 -p | tail -10 | grep -E "❯|Yes.*No|proceed|permission"

# Approve a prompt
tmux send-keys -t worker-3 'y' Enter

# Check all sessions
for s in shared worker-2 worker-3 worker-4; do
  echo "=== $s ==="
  tmux capture-pane -t $s -p 2>/dev/null | tail -5
done
```

## Tips

- Prefer `send` and `msgsend` for normal one-shot submissions
- **Read guard is enforced** for low-level `type`/`message`/`keys`
- **Every low-level action clears the read mark** — after `type`, read again before `keys`
- **Never wait or poll** — agent panes reply via tmux-bridge into YOUR pane
- **Label panes early** — easier than using `%N` IDs
- **`type` uses literal mode** — special characters are typed as-is
- **`read` defaults to 50 lines** — pass a higher number for more context
- **Non-agent panes** are the exception — you DO need to read them to see output
- Export `TMUX_BRIDGE_SOCKET="${TMUX%%,*}"` inside tmux to skip socket auto-detection on each call
- Use `capture-pane -p` to print to stdout (essential for scripting)
- Target format: `session:window.pane` (e.g., `shared:0.0`)
