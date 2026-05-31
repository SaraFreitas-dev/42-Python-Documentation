# ⚙️ Services and Monitoring

> Understanding how Linux runs background services, records events and monitors system health.

---

# Table of Contents

1. What is a Service?
2. Why Services Exist
3. What is a Daemon?
4. Services vs Applications
5. What is systemd?
6. Why systemd Exists
7. Service Lifecycle
8. systemctl
9. Common systemctl Commands
10. What is Logging?
11. Why Logs Matter
12. journald
13. journalctl
14. Common journalctl Commands
15. Cron
16. Why Automation Matters
17. Monitoring
18. Resource Monitoring
19. Monitoring Tools
20. Common Monitoring Commands
21. Troubleshooting Mindset
22. Mental Model

---

# 1️⃣ What is a Service?

A service is a program that runs in the background.

Unlike normal applications, services usually operate without direct user interaction.

Examples:

- SSH Server
- Database Server
- Web Server

---

Think of a service as:

```text
A worker operating behind the scenes
```

---

# 2️⃣ Why Services Exist

Many tasks must continue running even when nobody is actively using the system.

Examples:

```text
Website Hosting
Remote Access
Database Management
```

These functions are provided by services.

---

# 3️⃣ What is a Daemon?

In Linux, a background process is often called a:

```text
Daemon
```

---

Examples:

```text
sshd
cron
systemd-journald
```

---

A daemon usually starts automatically and waits for work.

---

Think of a daemon as:

```text
A service always on standby
```

---

# 4️⃣ Services vs Applications

Applications:

```text
User opens them
```

Examples:

- Browser
- VSCode
- Calculator

---

Services:

```text
Start automatically
```

and continue running in the background.

---

Example:

```text
Browser → User launches

SSH Service → Runs continuously
```

---

# 5️⃣ What is systemd?

systemd is one of the most important Linux components.

After Linux boots, systemd becomes responsible for managing the system.

---

Responsibilities include:

- Starting services
- Stopping services
- Monitoring services
- Managing dependencies
- Collecting logs

---

Think of systemd as:

```text
The operating system manager
```

---

# 6️⃣ Why systemd Exists

Before systemd, Linux distributions used different startup systems.

Managing services was often inconsistent.

systemd introduced:

✅ Standardization

✅ Faster startup

✅ Better service management

✅ Centralized logging

---

# 7️⃣ Service Lifecycle

A service typically follows:

```text
Stopped
   ↓
Started
   ↓
Running
   ↓
Stopped
```

---

Some services restart automatically if they fail.

---

# 8️⃣ systemctl

systemctl is the command-line tool used to communicate with systemd.

Think of it as:

```text
The remote control for services
```

---

# 9️⃣ Common systemctl Commands

Display service status:

```bash
systemctl status ssh
```

---

Start a service:

```bash
sudo systemctl start ssh
```

---

Stop a service:

```bash
sudo systemctl stop ssh
```

---

Restart a service:

```bash
sudo systemctl restart ssh
```

---

Enable automatic startup:

```bash
sudo systemctl enable ssh
```

---

List running services:

```bash
systemctl list-units --type=service
```

---

These examples are provided to explain concepts.

---

# 🔟 What is Logging?

Logs are records of events occurring on a system.

Think of logs as:

```text
A system diary
```

---

Logs help answer questions such as:

- What happened?
- When did it happen?
- Why did it happen?

---

# 1️⃣1️⃣ Why Logs Matter

Without logs:

```text
Troubleshooting becomes guesswork
```

---

Logs help identify:

- errors
- crashes
- failed logins
- service failures
- security incidents

---

# 1️⃣2️⃣ journald

journald is the logging service provided by systemd.

It collects events from across the operating system.

---

Examples:

- Service messages
- Startup events
- Errors
- Warnings

---

Think of journald as:

```text
The central log collector
```

---

# 1️⃣3️⃣ journalctl

journalctl is used to read logs managed by journald.

---

Think of it as:

```text
A search tool for system events
```

---

# 1️⃣4️⃣ Common journalctl Commands

Display recent logs:

```bash
journalctl
```

---

Display logs for a service:

```bash
journalctl -u ssh
```

---

Display latest entries:

```bash
journalctl -n 50
```

---

Display logs from current boot:

```bash
journalctl -b
```

---

These commands help administrators investigate problems.

---

# 1️⃣5️⃣ Cron

Cron is Linux's scheduling system.

It allows commands and scripts to run automatically.

---

Examples:

- Backups
- Monitoring
- Reports
- Maintenance tasks

---

Think of cron as:

```text
A task scheduler
```

---

# 1️⃣6️⃣ Why Automation Matters

Many administrative tasks repeat regularly.

Without automation:

```text
Humans must remember everything
```

---

Automation improves:

- consistency
- reliability
- efficiency

---

# 1️⃣7️⃣ Monitoring

Monitoring means observing system health.

Administrators monitor:

- CPU usage
- Memory usage
- Disk usage
- Network activity
- Running services

---

Monitoring answers:

```text
Is the system healthy?
```

---

# 1️⃣8️⃣ Resource Monitoring

Important resources include:

### CPU

Processing power.

### Memory

RAM usage.

### Storage

Disk capacity.

### Network

Traffic and connectivity.

---

Problems often appear first in monitoring data.

---

# 1️⃣9️⃣ Monitoring Tools

Linux provides many monitoring tools.

Examples:

- top
- htop
- free
- df
- uptime

---

Each focuses on a different aspect of system health.

---

# 2️⃣0️⃣ Common Monitoring Commands

View running processes:

```bash
top
```

---

Display memory usage:

```bash
free -h
```

---

Display disk usage:

```bash
df -h
```

---

Display system uptime:

```bash
uptime
```

---

Display process information:

```bash
ps aux
```

---

These commands provide insight into system performance.

---

# 2️⃣1️⃣ Troubleshooting Mindset

When something breaks, administrators usually ask:

- Is the service running?
- What do the logs say?
- Is the system overloaded?
- Did anything change recently?

---

Common troubleshooting flow:

```text
Problem
   ↓
Check Service
   ↓
Check Logs
   ↓
Check Resources
   ↓
Find Cause
```

---

Good troubleshooting relies on evidence rather than guesses.

---

# 2️⃣2️⃣ Mental Model

Imagine a city.

---

Services are:

```text
Workers
```

---

Daemons are:

```text
Workers always on duty
```

---

systemd is:

```text
The city manager
```

---

journald is:

```text
The city recorder
```

---

journalctl is:

```text
The archive search system
```

---

cron is:

```text
The scheduler
```

---

Monitoring tools are:

```text
Security cameras
```

---

Final Mental Image

```text
systemd
    ↓
Services
    ↓
Logs
    ↓
Monitoring
    ↓
Troubleshooting
```

Understanding these concepts is essential for maintaining healthy Linux systems.
