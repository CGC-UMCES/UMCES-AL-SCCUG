# tmux Quick Reference & Mini-Tutorial

A fast, practical guide to **tmux** (terminal multiplexer). Use tmux to keep long‑running sessions alive, split your terminal into panes, and detach/reattach from anywhere.

> Default prefix in tmux is **`Ctrl+b`** (written as `C-b`). You press the prefix, release, then press the following key(s).

---

## Install & Start

```bash
# macOS (Homebrew)
brew install tmux

# Ubuntu/Debian
sudo apt-get update && sudo apt-get install -y tmux

# Start a new session
tmux new -s mysession

# Attach to an existing session (first one)
tmux attach

# Attach to a named session
tmux attach -t mysession
```

---

## Core Concepts

- **Session** → top-level container (persists even if your SSH drops)  
- **Window** → a “tab” inside a session (numbered: 0,1,2,…)  
- **Pane** → split regions inside a window  
- **Prefix** → default `C-b` (many folks remap to `C-a` in `.tmux.conf`)

---

## Everyday Essentials

### Session management (CLI)
```bash
tmux new -s NAME           # new session
tmux ls                    # list sessions
tmux attach -t NAME        # attach to session
tmux switch -t NAME        # switch client to session
tmux rename-session -t old NAME
tmux kill-session -t NAME
tmux kill-server           # kill all tmux
```

### Universal lifesavers (inside tmux, using keys)
- `C-b d` → **detach** (leave it running)
- `C-b ?` → **help** (list key bindings)
- `C-b :` → **command prompt** (enter any `tmux` command)
- `C-b t` → **big clock** (fun, also tests keys)

---

## Windows (tabs)

**Keys (after prefix `C-b`):**
- `c` → new window
- `,` → rename window
- `&` → kill window
- `p` / `n` → previous / next window
- `[0–9]` → select window by number
- `w` → choose window from a list
- `.` → move window to a new index

**Commands:**
```bash
tmux new-window
tmux list-windows
tmux rename-window -t :1 "build"
tmux select-window -t :3
tmux move-window -s :3 -t :1
tmux kill-window -t :2
```

---

## Panes (splits)

**Keys (after prefix `C-b`):**
- `%` → split **vertical** (left/right)
- `"` → split **horizontal** (top/bottom)
- `o` → focus next pane
- `;` → toggle between last two panes
- `x` → kill pane
- `!` → break pane into its own window
- `q` → show pane numbers (then press the number)
- `z` → zoom pane toggle (maximize/restore)
- Arrow keys → move focus (with `C-b` + arrow)
- `C-b` + `Alt` + Arrow → resize (many builds)
- `C-b` + `:` then `resize-pane -{L|R|U|D} 5` → precise resize

**Commands:**
```bash
tmux split-window -h                  # horizontal split (left/right)
tmux split-window -v                  # vertical split (top/bottom)
tmux select-pane -t :.1               # focus pane index 1
tmux swap-pane -s :.1 -t :.2          # swap panes
tmux resize-pane -L 10                # resize 10 cells left (R,U,D also)
```

---

## Copy Mode (scroll/search/copy)

- Enter copy mode: `C-b [`  
- Move: arrows, `PgUp/PgDn`, `g` (top), `G` (bottom)
- Search: `/` forward, `?` backward, `n`/`N` next/prev
- Start selection: press **Space**, move, then **Enter** to copy selection
- Paste: `C-b ]`

For **mouse wheel scroll** in copy mode with mouse integration, see config below.

---

## Synchronize Panes (run same input everywhere)

- Toggle with command: `:setw synchronize-panes on` (or `off`)
- Useful for running the same command across multiple panes in a window.

```bash
# Example
tmux setw -w synchronize-panes on
# type your commands...
tmux setw -w synchronize-panes off
```

---

## Session Sharing (SSH pair programming)

1. SSH into the same machine/user account.
2. One person creates a session: `tmux new -s pair`
3. Others attach: `tmux attach -t pair`  
   *(For read-only spectators)*: `tmux attach -t pair -r`

To share safely between different users, consider Unix groups + socket perms.

---

## A Nice Starter `~/.tmux.conf`

> Save this file as `~/.tmux.conf`, then reload with `tmux source-file ~/.tmux.conf` or restart tmux.

```tmux
##### Prefix & basics #####
# Remap prefix to Ctrl-a (popular)
unbind C-b
set -g prefix C-a
bind C-a send-prefix

# Use | and - for splits (muscle memory from other tools)
bind | split-window -h
bind - split-window -v
unbind '"'
unbind %

# Easy reload
bind r source-file ~/.tmux.conf \; display-message "Reloaded ~/.tmux.conf"

##### Quality of life #####
set -g mouse on                   # mouse to select panes, resize, scroll
set -g history-limit 100000       # bigger scrollback
setw -g mode-keys vi              # vi-style keys in copy mode
set -g clock-mode-style 24        # 24h clock

# Faster key repeat (adjust to taste)
set -s escape-time 10

##### Status bar #####
set -g status-interval 5
set -g status-left "#[bold]#S #[default]"
set -g status-right "#(whoami) | %Y-%m-%d %H:%M "

##### Visuals #####
set -g pane-border-status top
set -g pane-border-format " #[bold]#P: #{pane_current_command} "
set -g status-bg colour236
set -g status-fg white

##### Copy to system clipboard (pick your platform) #####
# macOS (requires reattach-to-user-namespace on some setups; often not needed now)
bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "pbcopy"

# Linux with xclip
# bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "xclip -selection clipboard"

# Linux with wl-copy (Wayland)
# bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "wl-copy"
```

---

## Handy One-Liners & Workflows

```bash
# Create a new session in one shot
tmux new -s dev -n editor

# Split and run commands immediately
tmux new -s build \  \; split-window -v -p 30 'htop' \  \; split-window -h -p 50 'tail -f logs/app.log' \  \; select-pane -t 0 \  \; send-keys 'nvim .' C-m

# Rename session, window, pane
tmux rename-session -t 0 "main"
tmux rename-window  "server"
tmux select-pane -T "api"

# Save layout (dump) & restore later
tmux list-windows -F '#I #W #{window_layout}'
tmux select-layout even-horizontal
```

---

## Plugin Manager (Optional but Recommended)

Use **TPM** (Tmux Plugin Manager):

```bash
# Install TPM (once)
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

Add to `~/.tmux.conf`:
```tmux
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
# set -g @plugin 'tmux-plugins/tmux-resurrect'   # save/restore sessions
# set -g @plugin 'tmux-plugins/tmux-continuum'   # auto-save
run '~/.tmux/plugins/tpm/tpm'
```

Then press `prefix` + `I` (capital i) to install.

---

## Troubleshooting

- **Keys not working over SSH?** Make sure `$TERM` is sane (e.g., `xterm-256color`) and your SSH client supports the keys.  
- **Clipboard not copying to OS?** Use `copy-pipe` bindings and ensure `pbcopy` (macOS), `xclip`/`wl-copy` (Linux) is installed.  
- **Mouse wheel doesn’t scroll?** `set -g mouse on` and use copy mode (`C-b [`).  
- **Detached but can’t find session?** `tmux ls` then `tmux attach -t NAME`.  
- **Weird characters / colors?** Prefer truecolor terminals; set `export TERM=xterm-256color` in your shell rc.

---

## Cheat Sheet (Keys)

> All assume the prefix (default `C-b`) first.

| Action | Keys |
|---|---|
| Detach | `d` |
| New window | `c` |
| Rename window | `,` |
| Kill window | `&` |
| Next/prev window | `n` / `p` |
| Pick from list | `w` |
| Split vertical (left/right) | `%` |
| Split horizontal (top/bottom) | `"` |
| Toggle last two panes | `;` |
| Focus next pane | `o` |
| Show pane numbers | `q` (then number) |
| Zoom pane | `z` |
| Kill pane | `x` |
| Command prompt | `:` |
| Help (keys) | `?` |
| Copy mode | `[` |
| Paste buffer | `]` |

---

## Minimal Muscle-Memory Map (print me)

- **Detach:** `C-b d`  
- **New window / split:** `C-b c`, `C-b %`, `C-b "`  
- **Move around panes:** `C-b` + arrows, `C-b o`, `C-b q` + number  
- **Copy:** `C-b [`, then select, `Enter`; **Paste:** `C-b ]`  
- **Rename:** windows `C-b ,` ; panes `tmux select-pane -T "name"`  
- **List sessions:** `tmux ls`; **Attach:** `tmux attach -t NAME`

---

## Version Check

```bash
tmux -V
```

If you’re on an older tmux (< 3.x), some key names/features may differ slightly.

---

**Happy multiplexing!** If you want this guide with your own preferred keybindings or a custom starter `~/.tmux.conf`, just edit and commit this file.
