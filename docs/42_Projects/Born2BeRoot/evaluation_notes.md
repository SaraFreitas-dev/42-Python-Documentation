# 📝 Evaluation Notes

> Common Born2beroot evaluation questions and short conceptual answers.

---

# What is a Virtual Machine (VM)?

A Virtual Machine is a virtual computer running inside a physical computer.

It behaves like a real machine and has its own operating system, memory, storage and network interfaces.

---

# What is a Hypervisor?

A Hypervisor is software that creates and manages virtual machines.

It allocates resources and allows multiple VMs to run on the same physical machine.

---

# What is VirtualBox?

VirtualBox is a Type 2 Hypervisor used to create and manage virtual machines.

It runs on top of an existing operating system.

---

# What is the difference between Host and Guest?

The Host is the real operating system running on the physical machine.

The Guest is the operating system running inside the virtual machine.

---

# What is Linux?

Linux is an operating system built around the Linux kernel.

It manages hardware resources and allows applications to run.

---

# What is the Kernel?

The Kernel is the core of the operating system.

It manages:

- CPU
- Memory
- Hardware
- Processes
- Filesystems

---

# What is a Linux Distribution?

A Distribution combines:

- Linux Kernel
- Package Manager
- Utilities
- Configuration Tools

Examples include Debian and Rocky Linux.

---

# What is Root?

Root is the administrator account.

It has unrestricted access to the operating system.

---

# What is sudo?

sudo allows a user to execute commands with elevated privileges.

It provides temporary administrative access without logging directly as root.

---

# What is the difference between Root and sudo?

Root is the administrator account itself.

sudo is a mechanism that allows a user to temporarily use administrative privileges.

---

# What are Groups?

Groups allow multiple users to share permissions.

They simplify permission management.

---

# What is Ownership?

Ownership determines which user and group control a file or directory.

---

# What is chmod?

chmod is used to modify file and directory permissions.

---

# What is chown?

chown is used to change file ownership.

---

# What is an IP Address?

An IP Address identifies a device on a network.

It allows devices to communicate with each other.

---

# What is DNS?

DNS converts human-readable names into IP addresses.

Example:

google.com → IP Address

---

# What is a Port?

A Port identifies a specific network service running on a machine.

---

# What is SSH?

SSH (Secure Shell) allows secure remote access to another computer.

It encrypts communication between systems.

---

# What is TCP?

TCP is a network protocol focused on reliability and ordered delivery of data.

---

# What is UDP?

UDP is a network protocol focused on speed rather than guaranteed delivery.

---

# What is Authentication?

Authentication answers:

"Who are you?"

It verifies identity.

---

# What is Authorization?

Authorization answers:

"What are you allowed to do?"

It controls permissions.

---

# What is PAM?

PAM (Pluggable Authentication Modules) is Linux's authentication framework.

It centralizes authentication policies for services such as SSH, login and sudo.

---

# What is a Firewall?

A Firewall controls which network traffic is allowed or blocked.

It acts as a security barrier between systems.

---

# What is UFW?

UFW (Uncomplicated Firewall) is a simplified tool used to manage firewall rules.

---

# What is AppArmor?

AppArmor is a security framework that restricts what applications are allowed to do.

It limits damage if an application becomes compromised.

---

# What is Least Privilege?

Least Privilege means giving users and applications only the permissions they actually need.

---

# What is a Partition?

A Partition is a logical division of a disk.

It allows storage to be organized into separate sections.

---

# What is a Filesystem?

A Filesystem defines how files are stored and organized on a disk.

---

# What is a Mount Point?

A Mount Point is a directory where a filesystem becomes accessible.

---

# What is LVM?

LVM (Logical Volume Manager) is a storage management system that provides flexible and resizable storage.

---

# What are PV, VG and LV?

PV (Physical Volume):

Storage managed by LVM.

VG (Volume Group):

A pool of storage created from one or more Physical Volumes.

LV (Logical Volume):

Virtual storage created from a Volume Group.

---

# What is Encryption?

Encryption converts readable data into protected data.

Without the correct key, the data cannot be understood.

---

# Why Encrypt Data?

Encryption protects information if storage devices are lost, stolen or accessed without authorization.

---

# What is a Service?

A Service is a program running in the background that provides functionality to the system.

---

# What is a Daemon?

A Daemon is a background process that waits for work and usually starts automatically.

---

# What is systemd?

systemd is the service manager used by many Linux distributions.

It starts and manages services.

---

# What is systemctl?

systemctl is the command-line tool used to interact with systemd.

---

# What is Logging?

Logging is the process of recording system events.

---

# What is journald?

journald is the logging service used by systemd.

It collects system events and messages.

---

# What is journalctl?

journalctl is used to view and search logs managed by journald.

---

# What is cron?

cron is Linux's task scheduler.

It runs commands automatically at scheduled times.

---

# What is Monitoring?

Monitoring is the process of observing system health and performance.

Examples include CPU, memory, storage and network usage.
