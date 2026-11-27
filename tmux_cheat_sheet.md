# tmux Terminal Multiplexer Essentials

**Introduction:**
`tmux` (Terminal Multiplexer) is an indispensable tool for anyone working extensively in the command line, especially when managing remote servers or complex local development environments. It allows you to create and manage multiple terminal sessions, each with its own windows and panes, all within a single terminal window. This means you can detach from a session and re-attach later, even from a different machine, without losing your work.

**Why `tmux`?**
- **Persistence:** Keep processes running on a remote server even after your SSH connection drops.
- **Organization:** Manage multiple projects or tasks in isolated sessions.
- **Multitasking:** Split your terminal screen into multiple panes to monitor logs, edit files, and run commands simultaneously.
- **Productivity:** Navigate quickly between different contexts without opening new terminal windows.

---

## **Core Concepts & Essential Commands**

### **1. Sessions**
A `tmux` session is a persistent environment that can host multiple windows and panes. You can detach from a session and re-attach later.

- **`tmux new -s <session_name>`**: Create a new session with a specific name.
  *Example:* `tmux new -s homelab_management`
- **`tmux attach -t <session_name>`**: Attach to an existing named session.
  *Example:* `tmux attach -t homelab_management`
- **`tmux ls`** (or `tmux list-sessions`): List all active `tmux` sessions.
- **`tmux kill-session -t <session_name>`**: Terminate a specific session.
  *Example:* `tmux kill-session -t homelab_management`
- **`tmux kill-server`**: Kill all `tmux` sessions and the `tmux` server. Use with caution.

### **2. Prefix Key (Default: `Ctrl-b`)**
Most `tmux` commands are invoked by pressing a "prefix key" followed by another key. The default prefix is `Ctrl-b`. Throughout this guide, "Prefix" refers to `Ctrl-b`.

- **Prefix + `d`**: Detach from the current session. The session continues to run in the background.

### **3. Windows**
Within a `tmux` session, you can have multiple windows, each acting like a full terminal tab.

- **Prefix + `c`**: Create a new window.
- **Prefix + `w`**: List all windows in the current session (window selector).
- **Prefix + `n`**: Move to the next window.
- **Prefix + `p`**: Move to the previous window.
- **Prefix + `<window_number>`**: Go to a specific window by its number (e.g., Prefix + `0` for the first window).
- **Prefix + `,`**: Rename the current window.

### **4. Panes**
Each `tmux` window can be split into multiple independent panes, allowing you to view and interact with several terminal outputs simultaneously.

- **Prefix + `"`**: Split the current pane horizontally (new pane below).
- **Prefix + `%`**: Split the current pane vertically (new pane to the right).
- **Prefix + `<arrow_key>`**: Navigate between panes (e.g., Prefix + `Left Arrow` to move to the pane on the left).
- **Prefix + `z`**: Toggle zoom for the current pane (makes it full screen).
- **Prefix + `x`**: Kill the current pane.
- **Prefix + `[`**: Enter copy mode (allows scrolling and copying text).
  - While in copy mode: `Space` to start selection, `Enter` to copy.
- **Prefix + `]`**: Paste text copied from `tmux` buffer.

---

## **Basic `.tmux.conf` Configuration**

You can customize `tmux` by creating a `.tmux.conf` file in your home directory.

**Example `.tmux.conf` snippets:**

```bash
# Set prefix to Ctrl-a (more convenient for some)
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# Reload config with Prefix + r
bind r source-file ~/.tmux.conf \; display-message "tmux.conf reloaded!"

# Enable mouse support (scrolling, clicking to select panes/windows)
set -g mouse on

# Start window numbering from 1 instead of 0
set -g base-index 1
set-window-option -g pane-base-index 1

# Automatically set window titles
set-window-option -g automatic-rename on
set-option -g set-titles on

# Improve pane navigation with Alt + arrow keys (requires terminal support)
bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D
```

---

**Practical Workflow Example:**

1.  **Start a new session for a project:**
    `tmux new -s my_project`
2.  **Split the window vertically for code editing and a shell:**
    `Prefix` + `%`
3.  **Split the first pane horizontally for logs:**
    `Prefix` + `<arrow_key>` to select the first pane, then `Prefix` + `"`
4.  **Navigate between panes:**
    `Prefix` + `<arrow_key>`
5.  **Rename a window for clarity:**
    `Prefix` + `,` then type `Dev Environment`
6.  **Detach and leave:**
    `Prefix` + `d`
7.  **Later, re-attach:**
    `tmux attach -t my_project`

Mastering `tmux` transforms your terminal experience, making you more efficient and resilient in your daily sysadmin and development tasks.
