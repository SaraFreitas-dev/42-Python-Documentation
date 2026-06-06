# 👤 Users and Permissions

> Understanding how Linux manages users, groups, ownership and access control.

---

# Table of Contents

1. Why Users Exist
2. Users
3. Groups
4. Root User
5. Sudo
6. Authentication vs Authorization
7. Ownership
8. Permission Model
9. Read, Write and Execute
10. Viewing Permissions
11. chmod
12. Numeric Permissions
13. Symbolic Permissions
14. chown
15. Useful User Commands
16. Useful Group Commands
17. Security Principles
18. Mental Model

---

# 1️⃣ Why Users Exist

Linux was designed as a multi-user operating system.

Without users and permissions:

```text
Everyone could access everything
```

which would be a major security risk.

Users allow Linux to identify:

- who is performing an action
- who owns a file
- who can access a resource

---

# 2️⃣ Users

A user represents an identity inside Linux.

Examples:

```text
sara
student
admin
```

Users can:

- own files
- run processes
- belong to groups
- access resources

Every action performed on Linux is associated with a user.

---

# Viewing Current User

```bash
whoami
```

Displays the current logged-in user.

Example:

```text
sara
```

---

# Viewing User Information

```bash
id
```

Displays:

- User ID (UID)
- Group ID (GID)
- Group memberships

Example:

```text
uid=1000(sara)
gid=1000(sara)
groups=1000(sara),27(sudo)
```

---

# 3️⃣ Groups

Groups allow permissions to be assigned to multiple users simultaneously.

Instead of managing:

```text
Sara
John
Maria
```

individually, Linux can use:

```text
developers
```

and assign permissions to the group.

---

# Viewing Groups

```bash
groups
```

Displays groups associated with the current user.

---

# Creating Groups

```bash
groupadd developers
```

Creates a new group.

---

# Adding Users to Groups

```bash
usermod -aG developers sara
```

Adds a user to a group.

---

# Why Groups Matter

Groups simplify permission management.

Instead of assigning permissions individually, permissions can be assigned once to a group.

---

# 4️⃣ Root User

Root is the administrator account.

Root has unrestricted access.

Root can:

- install software
- create users
- remove users
- modify system configuration
- change permissions

---

Think of root as:

```text
The owner of the entire system
```

---

# 5️⃣ Sudo

Instead of logging directly as root, Linux commonly uses:

```bash
sudo command
```

Example:

```bash
sudo apt update
```

---

Sudo allows:

```text
Temporary administrator privileges
```

Benefits:

- accountability
- logging
- better security

---

# 6️⃣ Authentication vs Authorization

These concepts are different.

---

## Authentication

Answers:

```text
Who are you?
```

Examples:

- Passwords
- SSH Keys

---

## Authorization

Answers:

```text
What are you allowed to do?
```

Examples:

- Read files
- Modify files
- Install software

---

Authentication identifies.

Authorization grants permissions.

---

# 7️⃣ Ownership

Every file has:

- an owner
- a group owner

Example:

```text
Owner: sara
Group: developers
```

Ownership determines who controls the resource.

---

# Viewing Ownership

```bash
ls -l
```

Example:

```text
-rw-r--r-- 1 sara developers file.txt
```

---

# 8️⃣ Permission Model

Linux uses three permission categories.

```text
Owner
Group
Others
```

Each category receives different permissions.

---

# 9️⃣ Read, Write and Execute

Linux permissions are built around three actions.

---

## Read (r)

Allows viewing content.

Examples:

- open files
- view directories

---

## Write (w)

Allows modifications.

Examples:

- edit files
- create files
- delete files

---

## Execute (x)

Allows execution.

Examples:

- run programs
- run shell scripts

---

# 🔟 Viewing Permissions

Example:

```text
-rwxr-xr--
```

Breakdown:

```text
rwx | r-x | r--
```

Owner | Group | Others

---

Meaning:

Owner:

- Read
- Write
- Execute

Group:

- Read
- Execute

Others:

- Read

---

# 1️⃣1️⃣ chmod

chmod means:

```text
Change Mode
```

Used to modify permissions.

---

Example:

```bash
chmod 755 script.sh
```

Changes file permissions.

---

Making a script executable:

```bash
chmod +x script.sh
```

One of the most common uses of chmod.

---

Removing write permission:

```bash
chmod -w file.txt
```

---

# 1️⃣2️⃣ Numeric Permissions

Linux can represent permissions numerically.

Common values:

```text
755
744
700
644
```

---

Relationship:

```text
r = 4
w = 2
x = 1
```

Examples:

```text
7 = rwx
6 = rw-
5 = r-x
4 = r--
```

---

Example:

```bash
chmod 644 file.txt
```

---

# 1️⃣3️⃣ Symbolic Permissions

Instead of numbers, permissions can be modified symbolically.

Examples:

```bash
chmod +x script.sh
chmod -w file.txt
chmod g+w file.txt
```

---

Symbolic mode focuses on:

```text
Adding or removing permissions
```

---

# 1️⃣4️⃣ chown

chown means:

```text
Change Owner
```

Used to transfer ownership.

---

Example:

```bash
chown sara file.txt
```

Changes the owner.

---

Changing owner and group:

```bash
chown sara:developers file.txt
```

---

Ownership and permissions work together.

---

# 1️⃣5️⃣ Useful User Commands

Display current user:

```bash
whoami
```

---

Display user information:

```bash
id
```

---

Create a user:

```bash
useradd username
```

---

Delete a user:

```bash
userdel username
```

---

# 1️⃣6️⃣ Useful Group Commands

Display groups:

```bash
groups
```

---

Create group:

```bash
groupadd developers
```

---

Modify group membership:

```bash
usermod -aG developers sara
```

---

Delete group:

```bash
groupdel developers
```

---

# 1️⃣7️⃣ Security Principles

Linux permissions follow important security concepts.

---

## Least Privilege

Give users only the permissions they need.

---

## Separation of Responsibilities

Different users perform different tasks.

---

## Access Control

Protect resources according to their importance.

---

Permissions are one of Linux's primary security mechanisms.

---

# 1️⃣8️⃣ Mental Model

Imagine an office building.

---

Users are:

```text
Employees
```

---

Groups are:

```text
Departments
```

---

Files are:

```text
Documents
```

---

Permissions are:

```text
Access badges
```

---

Root is:

```text
The building owner
```

---

Final Mental Image

```text
Users
    ↓
Groups
    ↓
Permissions
    ↓
Security
```

Linux uses this hierarchy to control who can access what.
