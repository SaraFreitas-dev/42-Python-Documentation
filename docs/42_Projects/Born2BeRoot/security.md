# 🔐 Security

> Understanding how Linux systems protect users, data and services.

---

# Table of Contents

1. What is Security?
2. Why Security Matters
3. Security Principles
4. Least Privilege
5. Defense in Depth
6. Authentication
7. Authorization
8. Authentication vs Authorization
9. Password Policies
10. PAM
11. PAM Workflow
12. UFW
13. Firewall Concepts
14. Common UFW Commands
15. AppArmor
16. Attack Surface
17. Hardening
18. Security Layers
19. Useful Commands
20. Mental Model

---

# 1️⃣ What is Security?

Security is the practice of protecting:

- users
- systems
- applications
- data

The goal is not:

```text
Perfect Protection
```

because no system is completely secure.

The real goal is:

```text
Reduce Risk
```

and limit damage when something goes wrong.

---

# 2️⃣ Why Security Matters

Imagine a Linux server connected to the internet.

Without security:

- anyone could attempt access
- services could be abused
- data could be stolen
- systems could be modified

Security exists to reduce those risks.

---

# 3️⃣ Security Principles

Most security systems are built around a few important concepts.

### Least Privilege

Give only the permissions that are required.

### Defense in Depth

Use multiple layers of protection.

### Access Control

Restrict who can access resources.

### Accountability

Actions should be traceable.

---

# 4️⃣ Least Privilege

One of the most important security principles.

Example:

```text
User only needs to read files
```

They should NOT receive:

- write permissions
- administrator privileges

---

Bad:

```text
Everyone = Administrator
```

Good:

```text
Each user receives only what is needed
```

---

Benefits:

✅ Smaller attack surface

✅ Fewer mistakes

✅ Better security

---

# 5️⃣ Defense in Depth

Never rely on a single protection layer.

Example:

```text
Password
    ↓
Permissions
    ↓
Firewall
    ↓
AppArmor
```

If one layer fails, others still provide protection.

---

# 6️⃣ Authentication

Authentication answers:

```text
Who are you?
```

Examples:

- Passwords
- SSH Keys
- Biometrics

Authentication verifies identity.

---

# 7️⃣ Authorization

Authorization answers:

```text
What are you allowed to do?
```

Examples:

- Read files
- Modify files
- Install software

Authorization determines permissions.

---

# 8️⃣ Authentication vs Authorization

These concepts are often confused.

Authentication:

```text
Identity Verification
```

Authorization:

```text
Permission Verification
```

Example:

```text
Login Successful
      ↓
Authentication
      ↓
Permission Check
      ↓
Authorization
```

---

# 9️⃣ Password Policies

Password policies define security rules.

Common requirements:

- Minimum length
- Complexity requirements
- Expiration periods
- Password history
- Account lockouts

---

Example policy:

```text
Minimum length: 10
Uppercase required
Lowercase required
Numbers required
```

---

Weak password:

```text
password123
```

Stronger password:

```text
Blue!Tiger84#Moon
```

---

# 🔟 PAM

PAM means:

```text
Pluggable Authentication Modules
```

PAM is the authentication framework used by Linux.

Instead of every application implementing authentication separately:

```text
SSH
sudo
login
   ↓
  PAM
```

all of them can share the same authentication rules.

---

Think of PAM as:

```text
The central authentication manager
```

for the system.

---

# 1️⃣1️⃣ PAM Workflow

Without PAM:

```text
SSH
  ↓
Own Authentication Logic

sudo
  ↓
Own Authentication Logic

login
  ↓
Own Authentication Logic
```

---

With PAM:

```text
SSH
sudo
login
   ↓
  PAM
```

Everything follows the same security policies.

---

PAM can enforce:

- Password complexity
- Password expiration
- Account restrictions
- Login limitations

---

Benefits:

✅ Centralized management

✅ Consistency

✅ Flexibility

---

# 1️⃣2️⃣ UFW

UFW means:

```text
Uncomplicated Firewall
```

UFW provides a simpler way to manage firewall rules.

---

Think of UFW as:

```text
A security guard
```

standing at the entrance of a building.

Every connection request is checked.

---

# 1️⃣3️⃣ Firewall Concepts

A firewall decides:

```text
Allow?
or
Block?
```

for incoming and outgoing traffic.

---

Without a firewall:

```text
Everything is exposed
```

---

With a firewall:

```text
Only approved traffic passes
```

---

Example:

```text
Internet
    ↓
Firewall
    ↓
Server
```

---

# 1️⃣4️⃣ Common UFW Commands

View firewall status:

```bash
sudo ufw status
```

---

Allow SSH:

```bash
sudo ufw allow 22
```

Meaning:

```text
Allow traffic to SSH
```

---

Deny traffic:

```bash
sudo ufw deny 80
```

Meaning:

```text
Block traffic to HTTP
```

---

Enable firewall:

```bash
sudo ufw enable
```

---

These commands are examples for learning concepts.

---

# 1️⃣5️⃣ AppArmor

AppArmor is a Mandatory Access Control system.

Its purpose is to restrict applications.

---

Example:

```text
Web Server
```

Allowed:

```text
/var/www
```

Not Allowed:

```text
/home/user
```

---

Even if the application becomes compromised, AppArmor can limit damage.

---

Think of AppArmor as:

```text
A set of invisible walls
```

around an application.

---

# 1️⃣6️⃣ Attack Surface

Attack Surface means:

```text
Everything an attacker can interact with
```

Examples:

- Open ports
- Running services
- User accounts
- Applications

---

Larger attack surface:

```text
More opportunities for attacks
```

---

Smaller attack surface:

```text
Fewer opportunities for attacks
```

---

# 1️⃣7️⃣ Hardening

Hardening means:

```text
Making a system more secure
```

Examples:

- Disable unnecessary services
- Restrict permissions
- Configure firewalls
- Use strong passwords
- Reduce exposed ports

---

Hardening is proactive security.

---

# 1️⃣8️⃣ Security Layers

A secure system usually combines:

```text
Passwords
      ↓
PAM
      ↓
Permissions
      ↓
Firewall
      ↓
AppArmor
```

No single mechanism should be trusted completely.

---

# 1️⃣9️⃣ Useful Commands

Current user:

```bash
whoami
```

---

User information:

```bash
id
```

---

File permissions:

```bash
ls -l
```

---

Firewall status:

```bash
sudo ufw status
```

---

Running services:

```bash
systemctl list-units --type=service
```

---

Listening ports:

```bash
ss -tuln
```

---

These commands help administrators inspect the security state of a system.

---

# 2️⃣0️⃣ Mental Model

Imagine a castle.

Authentication:

```text
Who is at the gate?
```

Authorization:

```text
Which rooms may they enter?
```

Firewall:

```text
Outer Wall
```

PAM:

```text
Identity Verification Process
```

AppArmor:

```text
Internal Restrictions
```

Least Privilege:

```text
Give only the keys that are needed
```

---

Final Mental Image

```text
Authentication
      ↓
Authorization
      ↓
Permissions
      ↓
Firewall
      ↓
AppArmor
      ↓
Security
```

Security is the result of many protective layers working together.
