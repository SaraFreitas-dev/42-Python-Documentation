# 🐧 Linux Fundamentals

> Understanding the foundations of Linux, the operating system at the heart of Born2beroot.

---

# Table of Contents

1. What is Linux?
2. The Linux Philosophy
3. What is the Kernel?
4. User Space vs Kernel Space
5. Linux Distributions
6. Debian
7. Rocky Linux
8. Why So Many Distributions?
9. The Linux Boot Process
10. BIOS and UEFI
11. Bootloaders
12. systemd
13. The Shell
14. Common Shells
15. Terminal vs Shell
16. Commands and Utilities
17. Processes
18. Process Lifecycle
19. Foreground vs Background Processes
20. Process IDs (PID)
21. Files and Directories
22. Linux Filesystem Hierarchy
23. Important Directories
24. Paths
25. Hidden Files
26. Environment Variables
27. Package Managers
28. Why Linux Dominates Servers
29. Mental Model

---

# 1️⃣ What is Linux?

Linux is an operating system.

An operating system sits between:

```text
Hardware
    ↓
Operating System
    ↓
Applications
```

Without an operating system, applications would need to communicate directly with hardware.

Linux provides a stable and secure layer that allows software to run efficiently.

---

# 2️⃣ The Linux Philosophy

Linux follows a simple philosophy:

```text
Do one thing and do it well
```

Many Linux tools are intentionally small and focused.

Examples:

- grep
- cat
- find
- sort
- chmod

Each solves a specific problem.

---

# 3️⃣ What is the Kernel?

The kernel is the core of Linux.

Think of it as the operating system's manager.

Responsibilities include:

- CPU scheduling
- memory management
- hardware communication
- process management
- filesystem access

---

# 4️⃣ User Space vs Kernel Space

Linux separates the system into two worlds.

## User Space

Applications run here.

Examples:

- Firefox
- VSCode
- Python

## Kernel Space

The kernel runs here.

Applications must ask permission before accessing hardware.

This separation improves security and stability.

---

# 5️⃣ Linux Distributions

Linux itself is only the kernel.

A distribution combines:

- Linux kernel
- package manager
- utilities
- configuration tools
- documentation

Examples:

- Debian
- Ubuntu
- Rocky Linux
- Fedora
- Arch Linux

---

# 6️⃣ Debian

Debian is one of the oldest and most respected Linux distributions.

Known for:

- stability
- reliability
- large repositories
- conservative updates

Many other distributions are based on Debian.

---

# 7️⃣ Rocky Linux

Rocky Linux focuses on enterprise environments.

Known for:

- Red Hat compatibility
- long-term support
- server usage

It is commonly found in professional infrastructures.

---

# 8️⃣ Why So Many Distributions?

Different users have different goals.

Some prioritize:

- stability
- performance
- security
- ease of use
- customization

This is why many Linux distributions exist.

---

# 9️⃣ The Linux Boot Process

A simplified boot sequence:

```text
Power On
    ↓
BIOS / UEFI
    ↓
Bootloader
    ↓
Kernel
    ↓
systemd
    ↓
Services
    ↓
Login
```

Understanding this sequence helps explain how Linux starts.

---

# 🔟 BIOS and UEFI

These systems initialize hardware before Linux starts.

Responsibilities:

- detect hardware
- perform checks
- locate boot devices

UEFI is the modern replacement for BIOS.

---

# 1️⃣1️⃣ Bootloaders

A bootloader loads the operating system.

One common example is:

```text
GRUB
```

Its job is to locate and start the Linux kernel.

---

# 1️⃣2️⃣ systemd

systemd is responsible for managing the system after the kernel starts.

It handles:

- service startup
- dependencies
- logging
- process supervision

Modern Linux distributions rely heavily on systemd.

---

# 1️⃣3️⃣ The Shell

The shell is the interface between the user and the operating system.

You type commands:

```bash
ls
pwd
cd
```

The shell interprets them and communicates with the system.

---

# 1️⃣4️⃣ Common Shells

Examples include:

- Bash
- Zsh
- Fish
- Dash

Bash remains the most widely known shell.

---

# 1️⃣5️⃣ Terminal vs Shell

These terms are often confused.

### Terminal

The application window.

### Shell

The program running inside that window.

Example:

```text
Terminal
    ↓
Bash
```

---

# 1️⃣6️⃣ Commands and Utilities

Linux provides many small tools.

Examples:

```bash
ls
pwd
cat
grep
find
```

Combining small tools is one of Linux's greatest strengths.

---

# 1️⃣7️⃣ Processes

A process is a running program.

Examples:

```text
Python Script
Browser
SSH Server
```

Every running application becomes a process.

---

# 1️⃣8️⃣ Process Lifecycle

A process typically follows:

```text
Created
   ↓
Running
   ↓
Waiting
   ↓
Finished
```

The kernel manages this lifecycle.

---

# 1️⃣9️⃣ Foreground vs Background Processes

### Foreground

Runs directly in the terminal.

### Background

Runs independently.

Examples include:

- servers
- monitoring services
- daemons

---

# 2️⃣0️⃣ Process IDs (PID)

Every process receives a unique identifier.

Example:

```text
PID 100
PID 250
PID 421
```

The kernel uses these IDs to manage processes.

---

# 2️⃣1️⃣ Files and Directories

Linux treats almost everything as a file.

Examples:

- documents
- devices
- logs
- configuration files

This creates a consistent environment.

---

# 2️⃣2️⃣ Linux Filesystem Hierarchy

Linux follows a standard structure.

```text
/
├── home
├── etc
├── usr
├── var
├── boot
└── tmp
```

Everything starts from the root directory.

---

# 2️⃣3️⃣ Important Directories

## /home

User files.

## /etc

Configuration files.

## /var

Logs and changing data.

## /boot

Boot-related files.

## /tmp

Temporary files.

---

# 2️⃣4️⃣ Paths

Linux uses paths to locate files.

Example:

```text
/home/student/document.txt
```

Paths can be:

- absolute
- relative

---

# 2️⃣5️⃣ Hidden Files

Files beginning with:

```text
.
```

are hidden by default.

Examples:

```text
.bashrc
.profile
.gitignore
```

---

# 2️⃣6️⃣ Environment Variables

Environment variables store configuration values.

Examples:

```text
PATH
HOME
USER
SHELL
```

Applications often rely on them.

---

# 2️⃣7️⃣ Package Managers

Package managers simplify software installation.

Examples:

- apt
- dnf
- yum

They manage:

- installation
- updates
- dependencies

---

# 2️⃣8️⃣ Why Linux Dominates Servers

Linux powers most servers because it offers:

✅ Stability

✅ Performance

✅ Security

✅ Flexibility

✅ Open-source development

Many cloud platforms rely heavily on Linux.

---

# 2️⃣9️⃣ Mental Model

Think of Linux as:

```text
The operating system city
```

The kernel is:

```text
The city manager
```

Processes are:

```text
The citizens
```

The filesystem is:

```text
The map
```

And the shell is:

```text
Your way of communicating with the city
```
