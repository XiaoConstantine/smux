# tmux + Claude Code Config

Tmux configuration and Claude Code settings, including a hook that turns your tmux pane red when Claude Code needs approval.

## Setup

Clone the repo and create symlinks:

```bash
git clone https://github.com/ShawnPana/tmux-config.git ~/Projects/tmux-config

# Tmux config
ln -sf ~/Projects/tmux-config/.tmux.conf ~/.config/tmux/tmux.conf

# Claude Code settings
ln -sf ~/Projects/tmux-config/claude-settings.json ~/.claude/settings.json

# Pane alert script
mkdir -p ~/.config/tmux
ln -sf ~/Projects/tmux-config/scripts/claude-pane-alert.sh ~/.config/tmux/claude-pane-alert.sh
```

Add the `rename` helper to your `.zshrc`:

```bash
rename() {
  if [ $# -eq 2 ]; then
    tmux rename-window -t "$1" "$2"
  elif [ $# -eq 1 ]; then
    tmux rename-window "$1"
  fi
}
```

Reload tmux config:

```bash
tmux source-file ~/.config/tmux/tmux.conf
```

Restart any running Claude Code sessions to pick up the new hooks.

## Usage

All keybindings use **Option (Alt)** with no prefix required.

### Panes

| Key | Action |
|---|---|
| `Option+i/k/j/l` | Navigate up/down/left/right (no wrap) |
| `Option+n` | New pane (split + auto-tile) |
| `Option+w` | Kill current pane |
| `Option+o` | Cycle through layouts |
| `Option+g` | Mark current pane |
| `Option+y` | Swap marked pane with current |

### Windows

| Key | Action |
|---|---|
| `Option+m` | New window |
| `Option+u` | Next window |
| `Option+h` | Previous window |

Rename windows from the shell:

```bash
rename myname        # rename current window
rename 0 myname      # rename window by index
```

(Requires the `rename` function in your `.zshrc` — see Setup)

### Scrolling

| Key | Action |
|---|---|
| `Option+Tab` | Toggle scroll mode |
| `i/k` | Scroll up/down |
| `Shift+i/k` | Half-page up/down |
| `q` or `Escape` | Exit scroll mode |

Mouse scroll also works and auto-enters copy mode.

### Mouse

- Click to select panes
- Drag to select text (auto-copies to clipboard on release)
- Scroll wheel to scroll

### Claude Code pane alert

When Claude Code shows a permission prompt, the tmux pane background turns red. It resets when you approve or when Claude finishes responding. Only the relevant pane is affected.

## How the pane alert works

Three hooks in `claude-settings.json`:

| Event | Action |
|---|---|
| `PermissionRequest` | Pane turns red |
| `PostToolUse` | Pane resets (approved, tool ran) |
| `Stop` | Pane resets (fallback) |

The script uses `$TMUX_PANE` to target only the correct pane — no interference with other panes.
