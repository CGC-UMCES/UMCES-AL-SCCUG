# 🧭 tmux Quick Reference (Essential Commands & Cheat Sheet)

tmux = **terminal multiplexer** — lets you split your terminal, keep sessions running, and detach/re-attach anytime.

> Default prefix = **`Ctrl+b`** (written as `C-b`).  
> Press prefix, release, then the following key(s).


---
## Install & Start (likely already installed but in case it is not)

```bash
# macOS (Homebrew)
brew install tmux

# Ubuntu/Debian (SCC is Ubuntu)
sudo apt-get update && sudo apt-get install -y tmux

# Windows
# Best to install Windows Subsystem for Linux (WSL)

```

---

## 🚀 Start & Manage Sessions

```bash
tmux new -s mysession         # start a new session
tmux attach                   # attach to last session
tmux attach -t mysession      # attach to named session
tmux ls                       # list sessions
tmux kill-session -t mysession  # kill one session
tmux kill-server              # kill all sessions
```

---

## 🪟 Windows (Tabs)

| Action | Keys | Description |
|--------|------|--------------|
| New window | `C-b c` | Create new window |
| Rename window | `C-b ,` | Rename current window |
| Next / Prev window | `C-b n` / `C-b p` | Cycle through |
| Select window list | `C-b w` | Interactive list to pick one |
| Go to window # | `C-b <0–9>` | Jump directly |
| Kill window | `C-b &` | Close current window |
| Cycle layouts | `C-b Space` | Toggle layouts (even, tiled, etc.) |

---

## 🔳 Panes (Splits)

| Action | Keys | Description |
|--------|------|--------------|
| Split vertical | `C-b %` | Left/right split |
| Split horizontal | `C-b "` | Top/bottom split |
| Focus next pane | `C-b o` | Move between panes |
| Toggle last two panes | `C-b ;` | Quick switch |
| Show pane numbers | `C-b q` | Display pane numbers |
| Zoom / unzoom pane | `C-b z` | Maximize or restore pane |
| Kill pane | `C-b x` | Close active pane |
| Resize pane | `C-b : resize-pane -L 5` etc. | Move border (L/R/U/D) |
| Equalize panes | `C-b Space` or `C-b : select-layout even-horizontal` | Make panes even |

---

## 📋 Copy & Paste

| Action | Keys | Description |
|--------|------|--------------|
| Enter copy mode | `C-b [` | Start scrolling/copy mode |
| Scroll / move | arrows, `PgUp/PgDn`, `g`, `G` | Navigate |
| Start selection | Space | Begin marking text |
| Copy selection | Enter | Copy to tmux buffer |
| Paste | `C-b ]` | Paste last copied buffer |

---

## ⚙️ Basic tmux Commands (from prompt)

Press `C-b :` then type:
```bash
kill-session
rename-window logs
resize-pane -R 10
select-layout even-vertical
source-file ~/.tmux.conf
```

---

## 💡 Minimal Cheat Sheet

> All assume prefix `C-b` first.

| Action | Key |
|--------|-----|
| Detach | `d` |
| New window | `c` |
| Rename window | `,` |
| Next / prev window | `n` / `p` |
| List windows | `w` |
| Cycle layouts | `Space` |
| Split vertical | `%` |
| Split horizontal | `"` |
| Move between panes | `o` |
| Show pane numbers | `q` |
| Zoom pane | `z` |
| Kill pane | `x` |
| Command prompt | `:` |
| Help (bindings) | `?` |
| Copy mode | `[` |
| Paste | `]` |

---

## 🧠 Quick Reference Summary

- **Detach / reattach:** `C-b d`, then `tmux attach`
- **Rebalance panes:** `C-b Space`
- **Window list:** `C-b w`
- **Reload config:** `C-b : source-file ~/.tmux.conf`
- **Kill all sessions:** `tmux kill-server`

---

**Happy multiplexing!**
