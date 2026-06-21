# 🔐 Linux File Permissions — A Practical Guide

> A hands-on reference for understanding and managing file permissions in Linux.  
> Written as part of my personal Linux upskilling journey.

---

## 📖 Why File Permissions Matter

When you're working on a Linux system — whether it's a server, a VM, or your local machine — **every file and directory has access rules attached to it**.

These rules decide three things:

- **Who** can access a file
- **What** they can do with it (read, modify, or run it)
- **Who is denied** access entirely

Get this wrong and you either lock yourself out — or leave sensitive files wide open. Understanding permissions is a fundamental Linux skill.

---

## 👥 The Three Permission Levels

Every file in Linux has three groups of users associated with it:

| Level          | Who it applies to                                    |
| -------------- | ---------------------------------------------------- |
| **Owner (u)**  | The user who created or owns the file                |
| **Group (g)**  | Other users who belong to the same group as the file |
| **Others (o)** | Everyone else on the system                          |

---

## 🔑 The Three Permission Types

Within each level, there are three types of permission:

| Symbol | Numeric | What it allows                              |
| ------ | ------- | ------------------------------------------- |
| `r`    | `4`     | **Read** — view the file's contents         |
| `w`    | `2`     | **Write** — edit or delete the file         |
| `x`    | `1`     | **Execute** — run it as a script or program |
| `-`    | `0`     | **None** — no permission at all             |

---

## 🔍 Reading Permissions with `ls -l`

Run this to see a file's permissions:

```bash
ls -l myfile.sh
```

Example output:

```
-rwxr-xr-- 1 javeed developers 2048 Jun 10 09:00 myfile.sh
```

Here's how to decode it:

```
- rwx r-x r--
│  │   │   │
│  │   │   └── Others: read only
│  │   └────── Group: read + execute
│  └────────── Owner: read + write + execute
└───────────── File type (- = file, d = directory)
```

So in plain English:

- **javeed** (owner) can read, write, and execute the file
- **developers** (group) can read and execute it
- **Everyone else** can only read it

---

## ✏️ Changing Permissions with `chmod`

`chmod` is the command you use to set or modify permissions.

### Symbolic Mode (using letters)

Think of it as: **who** + **action** + **what**

```bash
# Add execute permission for the owner
chmod u+x script.sh

# Remove write permission from the group
chmod g-w notes.txt

# Give others read-only access
chmod o=r config.txt

# Set full access for owner, read+execute for group, nothing for others
chmod u=rwx,g=rx,o= deploy.sh
```

**Quick reference:**

| Symbol | Meaning                          |
| ------ | -------------------------------- |
| `u`    | Owner (user)                     |
| `g`    | Group                            |
| `o`    | Others                           |
| `a`    | All three                        |
| `+`    | Add permission                   |
| `-`    | Remove permission                |
| `=`    | Set exactly (overrides existing) |

---

### Numeric (Octal) Mode

Each permission type has a value. You **add them together** per level:

```
r = 4 | w = 2 | x = 1
```

So:

- `rwx` = 4+2+1 = **7**
- `r-x` = 4+0+1 = **5**
- `r--` = 4+0+0 = **4**
- `---` = 0+0+0 = **0**

```bash
# Owner: rwx (7), Group: r-x (5), Others: r-x (5)
chmod 755 script.sh

# Owner: rw- (6), Group: r-- (4), Others: r-- (4)
chmod 644 readme.txt

# Owner: rwx (7), Group: none (0), Others: none (0)
chmod 700 private.sh
```

**Common permission combos at a glance:**

| Octal | Who gets what                      | Typical use case                |
| ----- | ---------------------------------- | ------------------------------- |
| `777` | Everyone full access               | ⚠️ Avoid — very insecure        |
| `755` | Owner full, others read+execute    | Public scripts, web directories |
| `644` | Owner read+write, others read only | Config files, documents         |
| `700` | Owner full, no one else            | Private scripts                 |
| `600` | Owner read+write only              | SSH keys, credentials           |

---

## 👤 Changing Ownership with `chown`

`chown` lets you change who _owns_ a file and/or which group it belongs to.

```bash
# Change the owner
chown alice report.txt

# Change both owner and group
chown alice:engineering report.txt

# Change group only (notice the colon before group name)
chown :engineering report.txt

# Apply recursively to an entire directory
chown -R alice:engineering /var/www/project/
```

> 💡 **Tip:** You typically need `sudo` to change ownership of files you don't own.

---

## 👥 Changing Group with `chgrp`

`chgrp` is a focused version of `chown` — it only changes the group.

```bash
# Change the group of a single file
chgrp devops server.conf

# Change recursively
chgrp -R devops /etc/configs/
```

---

## ⚙️ Special Permissions

Beyond the standard `rwx`, Linux has three special permission flags worth knowing.

### SetUID (`s` on owner execute bit)

When set on an **executable**, it runs as the _file's owner_, not the person who ran it.

```bash
chmod u+s /usr/local/bin/mytool
```

Real-world example: `/usr/bin/passwd` uses SetUID so regular users can update their own passwords (which requires writing to a root-owned file).

---

### SetGID (`s` on group execute bit)

- On a **file**: it runs with the file's group permissions
- On a **directory**: new files created inside automatically inherit the directory's group

```bash
# Set on a file
chmod g+s shared-tool

# Set on a shared team directory
chmod g+s /var/shared/team/
```

Useful for shared team folders where you want all files to belong to the same group regardless of who created them.

---

### Sticky Bit (`t` on others execute bit)

When set on a **directory**, only the file's owner (or root) can delete it — even if others have write access to the directory.

```bash
chmod +t /tmp/shared/
```

The classic example is `/tmp` — everyone can write files there, but you can only delete your own files.

---

## 🛡️ Default Permissions with `umask`

Every time you create a new file or directory, the system applies a **umask** — a mask that _removes_ certain permissions from the defaults.

Default maximums:

- Files: `666` (rw-rw-rw-)
- Directories: `777` (rwxrwxrwx)

The `umask` value is **subtracted** from these:

```bash
# Check your current umask
umask

# Common output: 0022
```

With `umask 022`:

- Files get: `666 - 022` = `644` (rw-r--r--)
- Directories get: `777 - 022` = `755` (rwxr-xr-x)

```bash
# Set a stricter umask (only owner gets access)
umask 077
# Files: 600, Directories: 700
```

> 💡 Set umask in your `~/.bashrc` or `~/.zshrc` to make it persistent.

---

## 🔎 Viewing Special Permissions with `ls -l`

```bash
ls -l /usr/bin/passwd
ls -ld /tmp
ls -l /var/shared/team/
```

| Output       | What it means                                         |
| ------------ | ----------------------------------------------------- |
| `-rwsr-xr-x` | SetUID is active (`s` in owner execute position)      |
| `-rwxr-sr-x` | SetGID is active (`s` in group execute position)      |
| `drwxrwxrwt` | Sticky bit is active (`t` in others execute position) |

---

## 📋 Quick Reference Cheatsheet

```bash
# View permissions
ls -l filename
ls -ld directory/

# chmod — symbolic
chmod u+x file          # Add execute for owner
chmod g-w file          # Remove write for group
chmod o=r file          # Set read-only for others
chmod a+r file          # Add read for everyone

# chmod — numeric
chmod 755 file          # rwxr-xr-x
chmod 644 file          # rw-r--r--
chmod 600 file          # rw-------
chmod 777 file          # rwxrwxrwx (avoid)

# chown
chown user file
chown user:group file
chown -R user:group dir/

# chgrp
chgrp group file
chgrp -R group dir/

# Special bits
chmod u+s file          # SetUID
chmod g+s file/dir      # SetGID
chmod +t dir            # Sticky bit

# umask
umask                   # View current
umask 022               # Set (755 dirs / 644 files)
umask 077               # Strict (700 dirs / 600 files)
```

---

## 🚀 Key Takeaways

- Every file has **owner / group / others** permissions
- Permissions are **read (4), write (2), execute (1)** — add them for octal mode
- Use `chmod` to change permissions, `chown` to change ownership, `chgrp` for group
- **SetUID / SetGID / Sticky bit** are advanced flags for special use cases
- `umask` controls the default permissions applied to newly created files
- Principle of least privilege: **give only the access that's needed, nothing more**

---

_Part of my Linux & DevOps learning journey — documented publicly on GitHub._  
_Feel free to use this as a reference or starting point for your own notes._
