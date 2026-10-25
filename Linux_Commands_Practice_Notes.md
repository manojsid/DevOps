# Linux Commands — Interview Practice Notes

A quick-reference guide for DevOps interviews. Commands are grouped by purpose, with explanations and practical examples.

## 1. System information and resource usage

| Command | What it does | Useful notes |
|---|---|---|
| `free -h` | Displays RAM and swap usage in human-readable units. | Check **available** memory rather than only **free** memory; Linux uses RAM for cache. |
| `uname -a` | Shows kernel name, version, architecture, and other kernel details. | `uname -r` shows only the kernel release. |
| `cat /etc/os-release` | Shows Linux distribution information (name and version). | Useful to distinguish Ubuntu, RHEL, Amazon Linux, etc. |
| `df -h` | Shows filesystem capacity, used space, and available space. | `df -i` checks inode usage when space looks available but file creation fails. |
| `uptime` | Shows how long the system has been running, logged-in user count, and load averages. | Load averages are for 1, 5, and 15 minutes; they are **not** CPU percentages. |

**Practice:**
```bash
free -h
uname -a
cat /etc/os-release
df -h
df -i
uptime
```

## 2. Viewing files

| Command | What it does |
|---|---|
| `cat filename` | Prints file contents to the terminal. |
| `less filename` | Opens a scrollable viewer; move forward and backward, search with `/`, quit with `q`. |
| `more filename` | Pages through file contents, primarily forward. |

**Practice:**
```bash
cat /etc/os-release
less /var/log/syslog    # Common on Ubuntu; may not exist on other distros
```

## 3. Processes and performance

| Command | What it does | Useful notes |
|---|---|---|
| `top` | Displays live processes, CPU usage, memory usage, and system load. | Press `P` to sort by CPU or `M` to sort by memory. |
| `ps aux` | Lists processes with user, PID, CPU%, MEM%, and command. | Snapshot, unlike `top`'s live display. |
| `kill PID` | Sends **SIGTERM (15)** by default. | Requests graceful termination; a process can handle or ignore it. |
| `kill -9 PID` | Sends **SIGKILL (9)**. | Forces termination without application cleanup; use as a last resort. |

**Practice:**
```bash
ps aux | grep nginx
kill -15 1234
kill -9 1234   # Only if graceful termination fails
```

> **Interview tip:** SIGTERM gives a process the chance to finish work and clean up; it does not guarantee that the process will do so. SIGKILL cannot be caught or ignored by a normal userspace process.

## 4. Logs and services

| Command | What it does |
|---|---|
| `journalctl` | Reads logs collected by **systemd-journald**, including services and, where available, kernel messages. |
| `journalctl -k` | Displays kernel messages in the journal. |
| `journalctl -u nginx` | Shows logs for the `nginx` systemd unit. |
| `journalctl -u nginx -f` | Follows new logs for that unit. |
| `systemctl status nginx` | Checks the service state and recent logs. |
| `systemctl start nginx` | Starts the service. |
| `systemctl stop nginx` | Stops the service. |
| `systemctl restart nginx` | Restarts the service. |
| `systemctl enable nginx` | Configures the service to start automatically at boot. |

**Practice:**
```bash
systemctl status nginx
journalctl -u nginx --since '30 minutes ago'
journalctl -k -n 50
```

## 5. Linux file permissions

| Symbol | On a file | On a directory |
|---|---|---|
| `r` (read) | Read file contents. | List directory entries (subject to other permissions). |
| `w` (write) | Modify file contents. | Create, delete, or rename entries **when combined with execute permission**. |
| `x` (execute) | Execute a program/script (subject to other requirements). | Traverse or access entries in the directory. |

### Understanding `-rwxr-xr-x`

```text
-   rwx   r-x   r-x
│    │     │     └── Others: read and execute
│    │     └──────── Group: read and execute
│    └────────────── Owner: read, write, and execute
└─────────────────── File type: - means regular file
```

Other common file types: `d` = directory; `l` = symbolic link.

### SSH private-key permissions

```bash
chmod 400 myfile.pem
ssh -i /path/to/myfile.pem ec2-user@HOSTNAME
```

`chmod 400` gives **only the owner read permission**. It does **not** make the key publicly accessible; it restricts access. The SSH username depends on the AMI (for example, `ec2-user` on Amazon Linux and `ubuntu` on many Ubuntu AMIs).

## 6. Networking and ports

| Command | What it does | Useful notes |
|---|---|---|
| `ss -tuln` | Lists listening TCP (`-t`) and UDP (`-u`) sockets numerically (`-n`). | Add `-p` (often with `sudo`) to see owning processes. |
| `netstat` | Legacy tool for network connections, routes, and interface statistics. | `ss` is generally preferred on modern Linux. |

**Practice:**
```bash
ss -tuln
sudo ss -tulnp
ss -tn state established
```

---

## Quick interview drills

1. **Memory looks full. Is that necessarily a problem?** No. Check `free -h` and focus on **available** RAM, swap use, and signs of memory pressure.
2. **Disk has free GB but writes fail. Why?** Inodes may be exhausted. Compare `df -h` with `df -i`.
3. **What is the difference between `kill` and `kill -9`?** Default `kill` sends SIGTERM; `kill -9` sends uncatchable SIGKILL.
4. **What is the difference between `journalctl` and `systemctl`?** `journalctl` queries journal logs; `systemctl` manages and inspects systemd units.
5. **Why use `chmod 400` for an SSH key?** It restricts private-key read access to the owner.
6. **How do you find what is listening on port 8080?** `sudo ss -tulnp | grep ':8080'`.
7. **What does load average mean?** Average number of runnable or uninterruptible tasks over 1, 5, and 15 minutes, not direct CPU utilization.

*Revision tip: Run the commands on a test Linux VM and explain each output aloud.*
