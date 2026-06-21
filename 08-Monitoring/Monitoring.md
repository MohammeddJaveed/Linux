# 📊 Linux System Monitoring — A Practical Guide

> A hands-on reference for monitoring CPU, memory, disk, network, and logs in Linux.  
> Written as part of my personal Linux & DevOps upskilling journey.

---

## 📖 Why System Monitoring Matters

Imagine your web server slowing down at 2am — or a disk silently filling up until everything crashes. **System monitoring is how you catch these problems before they become disasters.**

In Linux, there's no single dashboard — instead, you have a powerful set of focused tools, each designed to give you a clear view of one part of your system. This guide covers all of them.

---

## 🗂️ Quick Command Index

### 🖥️ CPU & Memory

| Command      | What it does                                           |
| ------------ | ------------------------------------------------------ |
| `top`        | Live CPU, memory and process monitor                   |
| `htop`       | Friendlier live monitor with colour (needs installing) |
| `vmstat 1 5` | CPU, memory and I/O stats — updated every second       |
| `free -h`    | Quick snapshot of RAM and swap usage                   |
| `uptime`     | System load averages over 1, 5, and 15 minutes         |

### 💾 Disk

| Command                 | What it does                                           |
| ----------------------- | ------------------------------------------------------ |
| `df -h`                 | How much disk space is free on each mounted filesystem |
| `du -sh /path`          | How much space a specific directory is using           |
| `du -sh /* 2>/dev/null` | Top-level breakdown of what's filling your disk        |
| `iostat`                | Disk read/write activity and CPU usage                 |

### 🌐 Network

| Command                       | What it does                                   |
| ----------------------------- | ---------------------------------------------- |
| `ip a`                        | Show all network interfaces and IP addresses   |
| `ss -tulnp`                   | Active connections and listening ports         |
| `netstat -tulnp`              | Same as above (older systems)                  |
| `ping hostname`               | Test if a host is reachable                    |
| `traceroute hostname`         | Trace the network path to a destination        |
| `nslookup domain`             | DNS lookup — find what IP a domain resolves to |
| `curl -I https://example.com` | Check if a web endpoint is responding          |

### 📋 Logs

| Command                           | What it does                     |
| --------------------------------- | -------------------------------- |
| `tail -f /var/log/syslog`         | Follow system logs in real time  |
| `journalctl -f`                   | Follow systemd logs in real time |
| `journalctl -u service-name`      | Logs for one specific service    |
| `dmesg \| tail`                   | Most recent kernel messages      |
| `journalctl --since "1 hour ago"` | Logs from the last hour only     |

---

## 🖥️ CPU & Memory Monitoring

### `top` — Live System Monitor

```bash
top
```

`top` refreshes every few seconds and shows you everything that's running. Here's how to read it:

```
top - 10:32:01 up 3 days,  2:14,  1 user,  load average: 0.45, 0.38, 0.32
Tasks: 182 total,   1 running, 181 sleeping
%Cpu(s):  5.2 us,  1.3 sy,  0.0 ni, 92.8 id,  0.5 wa
MiB Mem :   7854.0 total,   2100.0 free,   3200.0 used,   2554.0 buff/cache
```

| Field          | What it means                                                                   |
| -------------- | ------------------------------------------------------------------------------- |
| `load average` | System load over last 1, 5, 15 mins. Above number of CPU cores = under pressure |
| `us`           | CPU time used by user processes                                                 |
| `sy`           | CPU time used by the kernel                                                     |
| `id`           | CPU idle percentage — higher is better                                          |
| `wa`           | CPU waiting on disk I/O — high values mean disk bottleneck                      |

**Useful keyboard shortcuts inside `top`:**

| Key | Action                             |
| --- | ---------------------------------- |
| `P` | Sort by CPU usage                  |
| `M` | Sort by memory usage               |
| `k` | Kill a process (prompts for PID)   |
| `r` | Renice a process (change priority) |
| `1` | Show individual CPU core stats     |
| `q` | Quit                               |

---

### `htop` — The Friendlier Alternative

```bash
# Install if needed
sudo apt install htop    # Debian/Ubuntu
sudo yum install htop    # CentOS/RHEL

htop
```

`htop` gives you everything `top` does, plus:

- Colour-coded CPU and memory bars (one per core)
- Mouse support — click any process to select it
- `F9` to kill, `F6` to sort, `F3` to search by name

> 💡 For quick checks on a server, `top` is always available. For proper investigation, `htop` is much easier to navigate.

---

### `vmstat` — CPU, Memory and I/O at a Glance

```bash
# Update every 1 second, show 5 readings
vmstat 1 5
```

Example output:

```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 210000  12000 380000    0    0    10    25  300  500  5  1 93  1  0
```

**Key columns to watch:**

| Column         | What it means                                         |
| -------------- | ----------------------------------------------------- |
| `r`            | Processes waiting for CPU time                        |
| `b`            | Processes blocked waiting on I/O                      |
| `si / so`      | Swap in / swap out — non-zero means you're low on RAM |
| `wa`           | CPU waiting on disk — high values = disk bottleneck   |
| `us / sy / id` | CPU: user / system / idle                             |

> ⚠️ If `si` and `so` are consistently non-zero, your system is swapping heavily — you need more RAM or you have a memory leak somewhere.

---

### `free` — Quick RAM and Swap Check

```bash
free -h
```

Example output:

```
              total        used        free      shared  buff/cache   available
Mem:           7.7G        3.1G        2.0G        200M        2.5G        4.2G
Swap:          2.0G          0B        2.0G
```

| Column       | What it means                                                      |
| ------------ | ------------------------------------------------------------------ |
| `total`      | Total installed RAM                                                |
| `used`       | RAM currently in use                                               |
| `buff/cache` | RAM used by the kernel for caching — can be freed if needed        |
| `available`  | RAM actually available for new processes (more useful than `free`) |

> 💡 Don't panic if `free` looks small — Linux intentionally uses spare RAM as disk cache. **Look at `available` for the real picture.**

---

## 💾 Disk Monitoring

### `df` — Disk Space by Filesystem

```bash
df -h
```

Example output:

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   32G   15G  69% /
/dev/sdb1       200G   80G  112G  42% /data
tmpfs           3.9G     0  3.9G   0% /dev/shm
```

> ⚠️ Watch the `Use%` column. At **85%+** start investigating. At **95%+** things start failing — logs stop writing, databases crash, services go down.

---

### `du` — What's Eating Your Disk Space

```bash
# How much is this directory using?
du -sh /var/log

# Break down the top-level of your root filesystem
du -sh /* 2>/dev/null

# Find the top 10 largest directories under /var
du -h /var 2>/dev/null | sort -rh | head -10
```

> 💡 `du` and `df` sometimes disagree. If `df` shows a disk is nearly full but `du` doesn't account for it, a deleted file is probably still being held open by a running process. Restart that process to release the space.

---

### `iostat` — Disk Read/Write Activity

```bash
# Install if needed
sudo apt install sysstat

# Show disk stats, update every 1 second
iostat -dx 1
```

Key columns:

| Column  | What it means                                    |
| ------- | ------------------------------------------------ |
| `%util` | How busy the disk is (close to 100% = saturated) |
| `r/s`   | Read operations per second                       |
| `w/s`   | Write operations per second                      |
| `await` | Average wait time for I/O requests (ms)          |

> ⚠️ If `%util` is consistently near 100% and `await` is high, your disk is a bottleneck — consider SSD or splitting data across volumes.

---

## 🌐 Network Monitoring

### `ip a` — View Network Interfaces

```bash
ip a
```

Example output:

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 192.168.1.50/24 brd 192.168.1.255 scope global eth0
```

This shows your interface names, IP addresses, and whether each interface is `UP` or `DOWN`.

> 💡 `ifconfig` does the same thing but is deprecated. Use `ip a` on modern systems.

---

### `ss` — Active Connections and Open Ports

```bash
# Show all listening TCP/UDP ports with process names
ss -tulnp
```

Example output:

```
Netid  State   Recv-Q  Send-Q  Local Address:Port  Process
tcp    LISTEN  0       128     0.0.0.0:22           sshd
tcp    LISTEN  0       511     0.0.0.0:80           nginx
tcp    LISTEN  0       128     127.0.0.1:5432       postgres
```

**Flag breakdown:**

| Flag | Meaning                               |
| ---- | ------------------------------------- |
| `-t` | TCP connections                       |
| `-u` | UDP connections                       |
| `-l` | Listening sockets only                |
| `-n` | Show port numbers (not service names) |
| `-p` | Show which process owns each socket   |

> 💡 This is invaluable for security checks — run it to see exactly what ports your server has open and which process is listening on each one.

---

### `ping` and `traceroute` — Connectivity Testing

```bash
# Basic connectivity check
ping google.com

# Limit to 4 packets
ping -c 4 google.com

# Trace the full network path (shows every hop)
traceroute google.com
```

`traceroute` is useful when `ping` works but something downstream is slow — it pinpoints exactly where the delay is in the route.

---

### `nslookup` — DNS Resolution Check

```bash
nslookup example.com
```

Example output:

```
Server:    8.8.8.8
Address:   8.8.8.8#53

Name:   example.com
Address: 93.184.216.34
```

> 💡 If your app can't reach a hostname, `nslookup` tells you whether it's a DNS problem (name not resolving) or a network problem (name resolves but host unreachable).

---

## 📋 Log Monitoring

Logs are where Linux tells you exactly what went wrong. Learning to read them in real time is one of the most valuable skills you can build.

### Live Log Tailing

```bash
# Follow the main system log in real time (Debian/Ubuntu)
tail -f /var/log/syslog

# Follow systemd's journal in real time (most modern distros)
journalctl -f
```

### Filtering Logs for a Specific Service

```bash
# See only logs from nginx
journalctl -u nginx

# Follow nginx logs live
journalctl -u nginx -f

# See what happened in the last hour
journalctl --since "1 hour ago"

# See logs between two timestamps
journalctl --since "2025-06-10 09:00" --until "2025-06-10 10:00"
```

### Kernel Messages

```bash
# View the most recent kernel messages
dmesg | tail

# Follow kernel messages live
dmesg -w

# Filter kernel messages for disk-related events
dmesg | grep -i "error\|disk\|sda"
```

> 💡 `dmesg` is the first place to look when hardware behaves unexpectedly — disk errors, USB devices, memory issues, and driver problems all show up here.

---

## 🔍 Putting It All Together — Troubleshooting Workflow

When something seems wrong, here's the order I work through it:

```
1. Is the system generally healthy?
   → uptime       (check load averages)
   → top / htop   (what's using CPU/memory?)

2. Is it a memory problem?
   → free -h      (is RAM full? is it swapping?)
   → vmstat 1 5   (is swap activity high?)

3. Is it a disk problem?
   → df -h        (is any filesystem full?)
   → iostat -dx 1 (is the disk saturated?)
   → du -sh /* 2>/dev/null  (what's using the space?)

4. Is it a network problem?
   → ping google.com      (basic connectivity)
   → ss -tulnp            (is the right port open?)
   → nslookup domain      (is DNS resolving?)
   → traceroute hostname  (where is the slowdown?)

5. What does the system say happened?
   → journalctl -f                    (live system logs)
   → journalctl -u service-name       (specific service logs)
   → dmesg | tail                     (kernel messages)
```

---

## 📋 Full Cheatsheet

```bash
# ─── CPU & MEMORY ──────────────────────────────────────
top                              # Live monitor
htop                             # Friendly live monitor
vmstat 1 5                       # CPU/memory/IO stats
free -h                          # RAM and swap snapshot
uptime                           # Load averages

# ─── DISK ──────────────────────────────────────────────
df -h                            # Disk space per filesystem
du -sh /path                     # Size of a directory
du -h /var 2>/dev/null | sort -rh | head -10  # Largest dirs
iostat -dx 1                     # Disk I/O activity

# ─── NETWORK ───────────────────────────────────────────
ip a                             # Network interfaces and IPs
ss -tulnp                        # Open ports and connections
netstat -tulnp                   # Same (older systems)
ping -c 4 google.com             # Connectivity check
traceroute google.com            # Network path trace
nslookup example.com             # DNS lookup
curl -I https://example.com      # HTTP response check

# ─── LOGS ──────────────────────────────────────────────
tail -f /var/log/syslog          # Live system log
journalctl -f                    # Live systemd journal
journalctl -u nginx -f           # Live logs for one service
journalctl --since "1 hour ago"  # Last hour of logs
dmesg | tail                     # Recent kernel messages
dmesg -w                         # Follow kernel messages live
dmesg | grep -i error            # Kernel errors only
```

---

## 🚀 Key Takeaways

- **CPU:** Use `top`/`htop` for live monitoring. Watch load average relative to your CPU core count
- **Memory:** Use `free -h` — look at `available`, not `free`. Non-zero swap activity is a warning sign
- **Disk:** `df -h` for space, `du` to find what's using it, `iostat` to spot I/O bottlenecks
- **Network:** `ss -tulnp` to see open ports, `ping`/`traceroute` to diagnose connectivity
- **Logs:** `journalctl` is your best friend on modern systemd systems — filter by service and time
- When troubleshooting, **work through layers** — system → memory → disk → network → logs

---

_Part of my Linux & DevOps upskilling journey — documented publicly on GitHub._  
_Feel free to use this as a reference or a starting point for your own notes._
