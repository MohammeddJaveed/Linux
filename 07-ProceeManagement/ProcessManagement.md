# ⚙️ Linux Process Management — A Practical Guide

> A hands-on reference for understanding, monitoring, and controlling processes in Linux.  
> Written as part of my personal Linux & DevOps upskilling journey.

---

## 📖 What is a Process?

Every time you run a program in Linux — whether it's a script, a web server, or a simple `ls` command — Linux creates a **process** for it.

Think of a process as Linux's way of tracking a running program. Each one gets:

- A unique **PID** (Process ID)
- A **PPID** (Parent Process ID — the process that launched it)
- An **owner** (the user who started it)
- A **priority** level that determines how much CPU time it gets

Understanding processes means understanding what's running on your system — and having the power to control it.

---

## 🗂️ Quick Command Index

### 👀 Viewing Processes

| Command             | What it does                                 |
| ------------------- | -------------------------------------------- |
| `ps aux`            | Show all running processes (every user)      |
| `ps -u username`    | Show processes for a specific user           |
| `ps -C processname` | Find a process by name                       |
| `pgrep processname` | Return the PID(s) of a named process         |
| `pidof processname` | Get the PID of a specific running program    |
| `top`               | Live interactive process monitor             |
| `htop`              | Friendlier version of top (needs installing) |

### 🔧 Managing Processes

| Command                | What it does                          |
| ---------------------- | ------------------------------------- |
| `kill PID`             | Gracefully stop a process by PID      |
| `kill -9 PID`          | Force kill — no cleanup, instant stop |
| `pkill processname`    | Stop a process by name                |
| `pkill -9 processname` | Force kill all instances by name      |
| `kill -STOP PID`       | Pause a process (keeps it in memory)  |
| `kill -CONT PID`       | Resume a paused process               |

### 🔄 Background & Foreground

| Command         | What it does                             |
| --------------- | ---------------------------------------- |
| `command &`     | Run a command in the background          |
| `jobs`          | List all background/suspended jobs       |
| `fg %jobnumber` | Bring a background job to the foreground |
| `bg %jobnumber` | Resume a suspended job in the background |
| `Ctrl + Z`      | Suspend the currently running process    |

### 📊 Priority Control

| Command               | What it does                                   |
| --------------------- | ---------------------------------------------- |
| `nice -n 10 command`  | Start a command with lower priority            |
| `nice -n -5 command`  | Start a command with higher priority (root)    |
| `renice -n 10 -p PID` | Lower the priority of a running process        |
| `renice -n -5 -p PID` | Raise the priority of a running process (root) |

### 🔌 Service / Daemon Management

| Command                               | What it does                  |
| ------------------------------------- | ----------------------------- |
| `systemctl list-units --type=service` | List all system services      |
| `systemctl start service-name`        | Start a service               |
| `systemctl stop service-name`         | Stop a service                |
| `systemctl restart service-name`      | Restart a service             |
| `systemctl enable service-name`       | Auto-start service on boot    |
| `systemctl status service-name`       | Check if a service is running |

---

## 👀 Viewing & Finding Processes

### `ps` — Snapshot of Running Processes

`ps` gives you a point-in-time list of processes. It doesn't update live.

```bash
# See everything running on the system
ps aux

# Show processes owned by a specific user
ps -u javeed

# Find a process by its name
ps -C nginx
```

**Breaking down `ps aux` output:**

```
USER    PID  %CPU  %MEM  VSZ   RSS  TTY  STAT  START  TIME  COMMAND
javeed  1234  0.5   1.2  5120  2048  pts/0  S  10:00  0:01  python3 app.py
```

| Column    | Meaning                                         |
| --------- | ----------------------------------------------- |
| `PID`     | Process ID                                      |
| `%CPU`    | CPU usage percentage                            |
| `%MEM`    | Memory usage percentage                         |
| `STAT`    | Process state (S=sleeping, R=running, Z=zombie) |
| `COMMAND` | The command that started the process            |

---

### `pgrep` and `pidof` — Find a Process PID Fast

```bash
# Returns the PID of all processes named "nginx"
pgrep nginx

# Returns the PID of a specific running program
pidof firefox
```

> 💡 `pgrep` is more flexible (supports partial names and patterns), while `pidof` matches exact program names.

---

## 🔧 Managing & Controlling Processes

### Killing Processes

Linux sends **signals** to processes to control them. `kill` is the command, but it doesn't just kill — it sends signals.

```bash
# Send the default SIGTERM signal — politely asks the process to stop
kill 1234

# SIGKILL — forces immediate termination, no cleanup
kill -9 1234

# Kill by process name instead of PID
pkill nginx

# Force kill all instances of a process by name
pkill -9 nginx
```

**Common signals:**

| Signal    | Number | What it does                               |
| --------- | ------ | ------------------------------------------ |
| `SIGTERM` | `15`   | Politely ask the process to quit (default) |
| `SIGKILL` | `9`    | Force kill — cannot be ignored or caught   |
| `SIGSTOP` | -      | Pause the process                          |
| `SIGCONT` | -      | Resume a paused process                    |

> ⚠️ Always try `kill PID` (SIGTERM) first. Use `-9` only if the process won't respond — it skips cleanup and can cause data loss.

---

### Pausing and Resuming Processes

Sometimes you want to temporarily pause a process rather than kill it:

```bash
# Pause a running process (it stays in memory, just frozen)
kill -STOP 1234

# Resume it exactly where it left off
kill -CONT 1234
```

---

## 🔄 Background & Foreground Jobs

Running long tasks? You don't need to wait — push them to the background.

```bash
# Start a command directly in the background
python3 long_script.py &

# While something is running, press Ctrl+Z to suspend it
# Then push it to the background
bg %1

# See all background/suspended jobs in this terminal session
jobs
```

Output of `jobs`:

```
[1]  Running    python3 long_script.py &
[2]+ Stopped    vim notes.txt
```

```bash
# Bring job #2 back to the foreground
fg %2

# Bring job #1 back to the foreground
fg %1
```

> 💡 Background jobs are tied to your terminal session. If you close the terminal, they stop. Use `nohup command &` to keep them running after logout.

---

## 📊 Monitoring with `top` and `htop`

### `top` — Live Process Monitor

```bash
top
```

Key things you see:

- **PID, USER, %CPU, %MEM** — what's using resources and who owns it
- **NI (Nice value)** — process priority (-20 = highest, +19 = lowest)
- **STAT** — current process state

**Keyboard shortcuts inside `top`:**

| Key | Action                             |
| --- | ---------------------------------- |
| `k` | Kill a process (prompts for PID)   |
| `r` | Renice a process (change priority) |
| `M` | Sort by memory usage               |
| `P` | Sort by CPU usage                  |
| `q` | Quit                               |

---

### `htop` — The Friendlier Alternative

```bash
# Install first if not available
sudo apt install htop   # Debian/Ubuntu
sudo yum install htop   # CentOS/RHEL

htop
```

`htop` gives you:

- Colour-coded CPU and memory bars
- Mouse support — click to select and kill processes
- Easier navigation without memorising key shortcuts

> 💡 For day-to-day process monitoring, `htop` is faster to work with. For servers where you can't install packages, `top` is always available.

---

## ⚡ Process Priority with `nice` and `renice`

Linux assigns each process a **nice value** from `-20` (highest priority, gets the most CPU) to `+19` (lowest priority, gets the least).

The name "nice" makes sense: a process with a high nice value is being _nice_ to other processes by stepping aside.

```bash
# Start a heavy task at low priority so it doesn't slow down other things
nice -n 15 ./backup-script.sh

# Start something at high priority (needs sudo for negative values)
sudo nice -n -10 ./critical-task.sh

# Change priority of a process that's already running
renice -n 10 -p 1234    # Lower its priority
sudo renice -n -5 -p 1234   # Raise its priority (root required)
```

To see priorities live, look at the **NI column** in `top` or `htop`.

---

## 🔌 Daemon & Service Management with `systemctl`

A **daemon** is a background process that runs continuously without user interaction — things like web servers, SSH, databases, and cron schedulers.

`systemctl` is how you manage them on modern Linux systems (systemd-based distros like Ubuntu, Debian, CentOS).

```bash
# See all currently running services
systemctl list-units --type=service

# Check the status of a specific service
systemctl status nginx

# Start / Stop / Restart a service
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx

# Enable a service to start automatically on boot
sudo systemctl enable nginx

# Disable auto-start on boot
sudo systemctl disable nginx
```

**Example output of `systemctl status nginx`:**

```
● nginx.service - A high performance web server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled)
   Active: active (running) since Mon 2025-06-10 09:00:00 UTC
```

- `active (running)` — service is up and running ✅
- `inactive (dead)` — service is stopped
- `failed` — service crashed or didn't start properly ❌

---

## 🧠 Process States Explained

When you look at the `STAT` column in `ps` or `top`, here's what the letters mean:

| State      | Letter | What it means                                |
| ---------- | ------ | -------------------------------------------- |
| Running    | `R`    | Actively using the CPU right now             |
| Sleeping   | `S`    | Waiting for something (I/O, event)           |
| Stopped    | `T`    | Paused with SIGSTOP                          |
| Zombie     | `Z`    | Finished but parent hasn't cleaned it up yet |
| Disk Sleep | `D`    | Waiting on disk I/O — cannot be interrupted  |

> ⚠️ **Zombie processes** aren't dangerous — they take no CPU or memory. But if you see hundreds of them, it usually means a parent process has a bug and isn't collecting its children's exit statuses.

---

## 📋 Full Cheatsheet

```bash
# ─── VIEWING ───────────────────────────────────────────
ps aux                          # All processes
ps -u username                  # Processes by user
ps -C processname               # Process by name
pgrep nginx                     # Get PID by name
pidof firefox                   # Get PID of exact program

# ─── KILLING ───────────────────────────────────────────
kill PID                        # Graceful stop (SIGTERM)
kill -9 PID                     # Force kill (SIGKILL)
pkill processname               # Kill by name
pkill -9 processname            # Force kill by name

# ─── PAUSE / RESUME ────────────────────────────────────
kill -STOP PID                  # Pause a process
kill -CONT PID                  # Resume a paused process

# ─── BACKGROUND / FOREGROUND ───────────────────────────
command &                       # Run in background
jobs                            # List background jobs
fg %1                           # Bring job 1 to foreground
bg %1                           # Resume job 1 in background
Ctrl+Z                          # Suspend current process
nohup command &                 # Run and survive terminal close

# ─── PRIORITY ──────────────────────────────────────────
nice -n 10 command              # Start at lower priority
sudo nice -n -5 command         # Start at higher priority
renice -n 10 -p PID             # Lower running process priority
sudo renice -n -5 -p PID        # Raise running process priority

# ─── MONITORING ────────────────────────────────────────
top                             # Live monitor
htop                            # Friendly live monitor

# ─── SERVICES ──────────────────────────────────────────
systemctl list-units --type=service
systemctl status service-name
sudo systemctl start service-name
sudo systemctl stop service-name
sudo systemctl restart service-name
sudo systemctl enable service-name
sudo systemctl disable service-name
```

---

## 🚀 Key Takeaways

- Every running program is a **process** with a unique PID and owner
- Use `ps`, `pgrep`, or `pidof` to **find** processes; use `top` or `htop` to **monitor** them live
- `kill` sends signals — always try `SIGTERM` (graceful) before `SIGKILL` (force)
- `nice` and `renice` control **CPU priority** — positive = lower priority, negative = higher (needs root)
- Use `&`, `jobs`, `fg`, and `bg` to manage **background tasks** in your terminal
- **Daemons** are background services managed with `systemctl`
- Zombie processes are harmless — but many of them signal a bug in a parent process

---

_Part of my Linux & DevOps upskilling journey — documented publicly on GitHub._  
_Feel free to use this as a reference or a starting point for your own notes._
