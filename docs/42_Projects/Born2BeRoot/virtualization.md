# 🖥️ Virtualization

> Understanding Virtual Machines, Hypervisors and the foundations of modern infrastructure.

---

# Table of Contents

1. What is Virtualization?
2. Why Virtualization Exists
3. Physical Machines Before Virtualization
4. What is a Virtual Machine?
5. Hypervisors
6. Type 1 vs Type 2 Hypervisors
7. VirtualBox Overview
8. Host vs Guest Systems
9. Virtual Hardware
10. Virtual CPU
11. Virtual RAM
12. Virtual Storage
13. Virtual Network Interfaces
14. Virtual Disks
15. Snapshots
16. Resource Allocation
17. Advantages of Virtualization
18. Disadvantages of Virtualization
19. Real World Uses
20. Virtualization vs Containers
21. Why Born2beroot Uses Virtualization
22. Mental Model

---

# 1️⃣ What is Virtualization?

Virtualization is the process of creating a virtual version of a computer system.

Instead of running a single operating system directly on physical hardware, virtualization allows multiple isolated systems to run on the same machine.

---

Think of virtualization as:

```text
Creating a computer inside another computer
```

---

# 2️⃣ Why Virtualization Exists

Before virtualization became common, organizations typically used:

```text
1 Physical Server
        =
1 Operating System
```

This approach caused several problems:

- Expensive hardware
- Low resource usage
- Difficult maintenance
- Poor scalability

---

Many servers used only a small percentage of their available resources.

Example:

```text
CPU Usage: 10%
RAM Usage: 15%
Disk Usage: 20%
```

Most of the hardware remained unused.

---

Virtualization was created to solve this problem.

---

# 3️⃣ Physical Machines Before Virtualization

Traditional infrastructure looked like this:

```text
Server A → Database
Server B → Website
Server C → Mail Server
Server D → Backup System
```

Each service required its own machine.

---

Today, a single powerful server can host many virtual systems simultaneously.

---

# 4️⃣ What is a Virtual Machine?

A Virtual Machine (VM) behaves like a completely independent computer.

A VM has:

- CPU
- Memory
- Storage
- Network Interface
- Operating System

Even though these resources are virtual, the operating system treats them as real hardware.

---

Example:

```text
Physical Machine
      │
      ▼
 Hypervisor
      │
      ▼
 Virtual Machine
      │
      ▼
 Debian Linux
```

---

# 5️⃣ Hypervisors

A hypervisor is software responsible for managing virtual machines.

Its job is to:

- Create virtual machines
- Allocate resources
- Isolate systems
- Control access to hardware

Without a hypervisor, virtualization would not be possible.

---

Popular hypervisors:

- VirtualBox
- VMware
- Hyper-V
- KVM
- Xen

---

# 6️⃣ Type 1 vs Type 2 Hypervisors

There are two main categories.

---

## Type 1 Hypervisors

Run directly on hardware.

```text
Hardware
    │
Hypervisor
    │
Virtual Machines
```

Examples:

- VMware ESXi
- Hyper-V Server
- Xen

---

## Type 2 Hypervisors

Run inside another operating system.

```text
Hardware
    │
Host OS
    │
VirtualBox
    │
Virtual Machines
```

Examples:

- VirtualBox
- VMware Workstation

Born2beroot normally uses this model.

---

# 7️⃣ VirtualBox Overview

VirtualBox is one of the most popular desktop virtualization tools.

It allows users to:

- Create VMs
- Allocate resources
- Manage virtual disks
- Create snapshots
- Configure networking

---

VirtualBox acts as the bridge between the host machine and the guest operating system.

---

# 8️⃣ Host vs Guest Systems

This distinction is extremely important.

---

## Host

The host is:

```text
The real machine
```

Examples:

- Windows
- macOS
- Linux

---

## Guest

The guest is:

```text
The virtual machine
```

Examples:

- Debian
- Rocky Linux

---

Example:

```text
Windows Host
      │
VirtualBox
      │
Debian Guest
```

---

# 9️⃣ Virtual Hardware

Every VM receives virtual hardware.

Examples:

- Virtual CPU
- Virtual RAM
- Virtual Disk
- Virtual Network Card

The guest operating system believes this hardware is real.

---

# 🔟 Virtual CPU

The virtual CPU represents processing power assigned to the VM.

Example:

```text
Physical CPU = 8 cores

VM 1 = 2 cores
VM 2 = 2 cores
VM 3 = 2 cores
VM 4 = 2 cores
```

The hypervisor distributes resources between machines.

---

# 1️⃣1️⃣ Virtual RAM

Memory is also shared.

Example:

```text
Physical RAM = 16 GB

VM A = 4 GB
VM B = 4 GB
VM C = 4 GB
```

Allocating too much memory may negatively impact the host system.

---

# 1️⃣2️⃣ Virtual Storage

A VM stores data inside virtual disks.

The guest operating system sees:

```text
A normal hard drive
```

even though the disk is actually a file on the host machine.

---

# 1️⃣3️⃣ Virtual Network Interfaces

Virtual machines require network connectivity.

Virtual adapters allow:

- Internet access
- Communication between VMs
- Communication with the host

---

# 1️⃣4️⃣ Virtual Disks

Virtual disks are files representing storage devices.

Common formats:

- VDI
- VHD
- VMDK

The guest system treats these files as physical disks.

---

# 1️⃣5️⃣ Snapshots

Snapshots are one of the most useful virtualization features.

A snapshot records the state of a VM at a specific moment.

---

Example:

```text
Clean Installation
      ↓
Snapshot
      ↓
Experiment
      ↓
Problem Occurs
      ↓
Restore Snapshot
```

---

Snapshots make testing much safer.

---

# 1️⃣6️⃣ Resource Allocation

Every VM consumes resources.

Administrators must decide:

- CPU allocation
- Memory allocation
- Storage allocation
- Network configuration

Poor allocation can reduce performance.

---

# 1️⃣7️⃣ Advantages of Virtualization

Benefits include:

✅ Isolation

✅ Easy testing

✅ Portability

✅ Lower hardware costs

✅ Better resource utilization

✅ Safer experimentation

---

# 1️⃣8️⃣ Disadvantages of Virtualization

Potential drawbacks:

❌ Resource overhead

❌ Increased complexity

❌ Additional management layers

❌ Hardware limitations

---

Despite these drawbacks, the benefits usually outweigh the costs.

---

# 1️⃣9️⃣ Real World Uses

Virtualization is used almost everywhere.

Examples:

- Cloud providers
- Development environments
- Testing platforms
- Corporate servers
- Educational environments

---

Companies often run hundreds or thousands of virtual machines.

---

# 2️⃣0️⃣ Virtualization vs Containers

Students often confuse these concepts.

---

## Virtual Machines

Include:

```text
Operating System
Kernel
Applications
```

---

## Containers

Share the host kernel.

They are lighter and faster but provide a different type of isolation.

Examples:

- Docker
- Podman

---

# 2️⃣1️⃣ Why Born2beroot Uses Virtualization

Virtualization provides:

- Safety
- Isolation
- Reproducibility
- Easy recovery

Students can experiment without risking their primary operating system.

---

If something breaks:

```text
Delete VM
Create New VM
Continue Learning
```

---

# 2️⃣2️⃣ Mental Model

Think of a Virtual Machine as:

```text
A complete computer
inside another computer
```

---

Think of the Hypervisor as:

```text
The manager
of those computers
```

---

# Final Mental Image

```text
Physical Computer
        │
        ▼
   VirtualBox
        │
        ▼
   Debian VM
        │
        ▼
 Linux Administration
```

Virtualization creates a safe environment where systems can be studied, configured and maintained without affecting the real machine.
