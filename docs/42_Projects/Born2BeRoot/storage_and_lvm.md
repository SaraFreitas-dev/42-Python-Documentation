# 💾 Storage and LVM

> Understanding how Linux stores, organizes and protects data.

---

# Table of Contents

1. Why Storage Matters
2. Physical Disks
3. Disk Layout
4. Partitions
5. Why Use Partitions?
6. Filesystems
7. Mount Points
8. The Linux Filesystem Tree
9. Common Mount Points
10. What Happens During Boot?
11. LVM Introduction
12. Why LVM Exists
13. Physical Volumes
14. Volume Groups
15. Logical Volumes
16. LVM Workflow
17. LVM Advantages
18. Encryption
19. Why Encrypt Data?
20. LUKS Concepts
21. Storage Commands
22. Mental Model

---

# 1️⃣ Why Storage Matters

Every operating system needs a place to store:

- files
- applications
- configuration
- logs
- user data

Without storage, all information would disappear when the computer shuts down.

---

# 2️⃣ Physical Disks

Storage begins with a physical device.

Examples:

- HDD (Hard Disk Drive)
- SSD (Solid State Drive)
- NVMe SSD

Linux sees these devices as storage resources that can be organized and divided.

---

# 3️⃣ Disk Layout

Think of a disk as a completely empty piece of land.

```text
Entire Disk
┌───────────────────┐
│                   │
│    Empty Space    │
│                   │
└───────────────────┘
```

Before it can be used efficiently, it is usually divided into sections.

---

# 4️⃣ Partitions

A partition is a logical division of a disk.

Example:

```text
Disk
├── Partition 1
├── Partition 2
└── Partition 3
```

Each partition behaves almost like an independent storage area.

---

# 5️⃣ Why Use Partitions?

Partitions provide:

✅ Organization

✅ Isolation

✅ Easier management

✅ Better security

---

Example:

```text
Partition 1 → System
Partition 2 → User Files
Partition 3 → Logs
```

If one partition fills up, the others remain unaffected.

---

# 6️⃣ Filesystems

A filesystem defines how data is stored and organized.

Without a filesystem:

```text
Disk = Raw Storage
```

Linux would not know how to manage files.

---

Common filesystems:

- ext4
- XFS
- Btrfs

---

Think of a filesystem as:

```text
The filing system of a library
```

It determines where everything is stored.

---

# 7️⃣ Mount Points

Linux does not assign drive letters like Windows.

Instead, storage is attached to directories.

This process is called:

```text
Mounting
```

---

Example:

```text
Storage Device
      ↓
Mount Point
      ↓
/home
```

---

# 8️⃣ The Linux Filesystem Tree

Linux has a single directory tree.

Everything starts at:

```text
/
```

called:

```text
Root Directory
```

---

Example:

```text
/
├── home
├── var
├── boot
└── etc
```

Additional storage is attached somewhere inside this tree.

---

# 9️⃣ Common Mount Points

## /home

User files.

---

## /var

Logs and changing data.

---

## /boot

Boot-related files.

---

## /

Root filesystem.

---

Example:

```text
Disk A
   ↓
/

Disk B
   ↓
/home
```

---

# 🔟 What Happens During Boot?

When Linux starts:

1. Detect storage
2. Load filesystems
3. Mount partitions
4. Make data available

Only after mounting can files be accessed.

---

# 1️⃣1️⃣ LVM Introduction

LVM means:

```text
Logical Volume Manager
```

This is one of the most important Born2beroot concepts.

Many students struggle with it because it adds an extra layer between the disk and the filesystem.

---

# 1️⃣2️⃣ Why LVM Exists

Traditional partitions are rigid.

Example:

```text
Partition = 20 GB
```

If it becomes full, resizing may be difficult.

---

LVM introduces flexibility.

Instead of thinking about fixed partitions:

```text
Think about storage pools
```

---

# 1️⃣3️⃣ Physical Volumes

A Physical Volume (PV) is storage managed by LVM.

Example:

```text
Disk
   ↓
Physical Volume
```

Think of a PV as raw storage that LVM can use.

---

# 1️⃣4️⃣ Volume Groups

A Volume Group (VG) combines storage.

Example:

```text
PV1 = 100 GB
PV2 = 100 GB

VG = 200 GB
```

Multiple disks can be combined into a single storage pool.

---

This is one of LVM's biggest advantages.

---

# 1️⃣5️⃣ Logical Volumes

Logical Volumes (LVs) are created from the Volume Group.

Example:

```text
Volume Group
     ↓
 ├── LV Home
 ├── LV Root
 └── LV Logs
```

To Linux, these behave similarly to partitions.

---

# 1️⃣6️⃣ LVM Workflow

The easiest way to remember LVM:

```text
Disk
  ↓
Physical Volume (PV)
  ↓
Volume Group (VG)
  ↓
Logical Volume (LV)
  ↓
Filesystem
```

---

Or visually:

```text
Disk
 │
 ▼
PV
 │
 ▼
VG
 │
 ▼
LV
 │
 ▼
ext4
```

---

# 1️⃣7️⃣ LVM Advantages

Benefits include:

✅ Easier resizing

✅ Flexible storage

✅ Better management

✅ Storage pooling

---

Example:

Traditional partition:

```text
20 GB
```

Needs more space?

Often difficult.

---

LVM:

```text
20 GB
      ↓
40 GB
```

Usually much easier to expand.

---

# 1️⃣8️⃣ Encryption

Encryption protects stored data.

Without the correct key:

```text
Data remains unreadable
```

---

Encryption converts:

```text
Readable Data
```

into:

```text
Protected Data
```

---

# 1️⃣9️⃣ Why Encrypt Data?

Imagine someone steals a hard drive.

Without encryption:

```text
Files may be readable
```

---

With encryption:

```text
Files appear meaningless
```

without the correct credentials.

---

Benefits:

✅ Privacy

✅ Security

✅ Theft protection

---

# 2️⃣0️⃣ LUKS Concepts

LUKS is commonly used for disk encryption on Linux.

Think of it as:

```text
A lock protecting storage
```

---

Example:

```text
Disk
   ↓
LUKS
   ↓
Filesystem
```

Before data can be accessed, the lock must be opened.

---

# 2️⃣1️⃣ Storage Commands

Display disks:

```bash
lsblk
```

---

Display mounted filesystems:

```bash
df -h
```

---

Display filesystem usage:

```bash
du -sh folder
```

---

Display partition information:

```bash
fdisk -l
```

---

Display mount information:

```bash
mount
```

---

Display LVM information:

```bash
pvs
vgs
lvs
```

---

These commands help administrators understand storage layout.

---

# 2️⃣2️⃣ Mental Model

Imagine a shopping center.

The disk is:

```text
The entire building
```

---

Partitions are:

```text
Individual stores
```

---

Filesystems are:

```text
The organization system
inside each store
```

---

LVM is:

```text
A manager capable of
expanding or shrinking stores
when needed
```

---

Encryption is:

```text
A lock on the building
```

---

Final Mental Image

```text
Disk
  ↓
Partition
  ↓
Filesystem
  ↓
Mount Point
```

Traditional storage.

---

```text
Disk
  ↓
PV
  ↓
VG
  ↓
LV
  ↓
Filesystem
```

LVM storage.

This extra layer is what gives LVM its flexibility.
