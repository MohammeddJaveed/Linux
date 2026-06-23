# 💾 Linux Disk & Storage Management — A Practical Guide

> A hands-on reference for viewing, partitioning, formatting, mounting, and managing storage in Linux.  
> Written as part of my personal Linux & DevOps upskilling journey.

---

## 📖 Why Disk Management Matters

Running out of disk space on a production server is one of the most avoidable — and most common — causes of system failures. Services crash, logs stop writing, databases corrupt.

Understanding how Linux handles storage gives you the power to:

- See exactly what's on your system and where space is going
- Add new disks and make them available to the OS
- Resize and restructure storage without data loss (using LVM)
- Control swap space to handle memory pressure

This guide walks through all of it — from reading disk info to a full new disk setup.

---

## 🗂️ Quick Command Index

### 🔍 Viewing Disk Information

| Command        | What it does                                          |
| -------------- | ----------------------------------------------------- |
| `lsblk`        | Tree view of all block devices and their mount points |
| `fdisk -l`     | Detailed partition table for all disks                |
| `blkid`        | Show UUID and filesystem type for each device         |
| `df -h`        | Disk space used/free per mounted filesystem           |
| `du -sh /path` | How much space a specific directory is using          |

### ✂️ Partition Management

| Command               | What it does                                     |
| --------------------- | ------------------------------------------------ |
| `fdisk /dev/sdX`      | Interactive partition editor (MBR disks)         |
| `parted /dev/sdX`     | Partition editor with GPT support (modern disks) |
| `mkfs.ext4 /dev/sdX1` | Format a partition as ext4                       |
| `mkfs.xfs /dev/sdX1`  | Format a partition as XFS                        |

### 📂 Mounting & Unmounting

| Command                    | What it does                            |
| -------------------------- | --------------------------------------- |
| `mount /dev/sdX1 /mnt`     | Mount a partition to a directory        |
| `umount /mnt`              | Unmount a partition                     |
| `mount -o remount,rw /mnt` | Remount a partition as read-write       |
| `mount -a`                 | Mount everything listed in `/etc/fstab` |

### 📦 Logical Volume Management (LVM)

| Command                                | What it does                                |
| -------------------------------------- | ------------------------------------------- |
| `pvcreate /dev/sdX`                    | Initialise a disk as a physical volume      |
| `vgcreate vg_name /dev/sdX`            | Create a volume group from physical volumes |
| `lvcreate -L 10G -n lv_name vg_name`   | Create a 10GB logical volume                |
| `lvextend -L +5G /dev/vg_name/lv_name` | Grow a logical volume by 5GB                |
| `resize2fs /dev/vg_name/lv_name`       | Resize the filesystem after extending       |

### 🔄 Swap Management

| Command            | What it does                         |
| ------------------ | ------------------------------------ |
| `mkswap /dev/sdX`  | Prepare a partition as swap space    |
| `swapon /dev/sdX`  | Enable swap                          |
| `swapoff /dev/sdX` | Disable swap                         |
| `swapon --show`    | Show active swap devices and usage   |
| `free -h`          | Check RAM and swap usage at a glance |

---

## 🔍 Viewing Disk Information

### `lsblk` — Your Starting Point

Before touching anything, always run `lsblk` first to understand what's on the system.

```bash
lsblk
```

Example output:

```
NAME    MAJ:MIN  RM   SIZE  RO  TYPE  MOUNTPOINT
sda       8:0     0   100G   0  disk
├─sda1    8:1     0    96G   0  part  /
└─sda2    8:2     0     4G   0  part  [SWAP]
sdb       8:16    0    20G   0  disk
```

**How to read this:**

| Column       | Meaning                                                                           |
| ------------ | --------------------------------------------------------------------------------- |
| `NAME`       | Device name (`sda` = first disk, `sdb` = second, `sda1` = first partition of sda) |
| `SIZE`       | Total size of the device                                                          |
| `TYPE`       | `disk` = physical drive, `part` = partition, `lvm` = logical volume               |
| `MOUNTPOINT` | Where it's accessible in the filesystem (blank = not mounted)                     |

In this example:

- `sda` is your existing system disk — already partitioned and mounted
- `sdb` is a new blank disk — no partitions yet, not mounted

---

### `fdisk -l` — Partition Table Details

```bash
sudo fdisk -l
```

Shows partition type, size, start/end sectors, and filesystem type for every disk on the system. Useful when `lsblk` doesn't give you enough detail.

---

### `blkid` — Find UUIDs

```bash
sudo blkid
```

Example output:

```
/dev/sda1: UUID="a1b2c3d4-..." TYPE="ext4"
/dev/sda2: UUID="e5f6g7h8-..." TYPE="swap"
```

> 💡 UUIDs are what you use in `/etc/fstab` to make mounts persistent across reboots — device names like `/dev/sda1` can change, but UUIDs never do.

---

## ✂️ Partition Management

### Creating a Partition with `fdisk`

`fdisk` is an interactive tool. Here's the full flow for a new blank disk:

```bash
sudo fdisk /dev/sdb
```

Inside `fdisk`, use these keys:

| Key | Action                        |
| --- | ----------------------------- |
| `m` | Show help menu                |
| `p` | Print current partition table |
| `n` | Create a new partition        |
| `d` | Delete a partition            |
| `w` | Write changes and exit        |
| `q` | Quit without saving           |

**Typical flow:**

1. Press `n` → new partition
2. Press `p` → primary partition
3. Accept defaults for partition number and sectors (uses full disk)
4. Press `w` → write and exit

Verify it worked:

```bash
lsblk
```

> ⚠️ `fdisk` is best for **MBR** partition tables (disks under 2TB). For larger or newer disks use `parted` which supports **GPT**.

---

### Formatting a Partition

Once partitioned, the partition needs a **filesystem** before it can store data.

```bash
# ext4 — the standard choice for most Linux use cases
sudo mkfs.ext4 /dev/sdb1

# XFS — better for large files and high-performance workloads
sudo mkfs.xfs /dev/sdb1
```

**Which filesystem to choose?**

| Filesystem | Best for                                                  |
| ---------- | --------------------------------------------------------- |
| `ext4`     | General purpose — reliable, well-supported, great default |
| `xfs`      | Large files, databases, high I/O workloads                |
| `vfat`     | USB drives that need to work on Windows/macOS too         |

> ⚠️ Formatting **destroys all existing data** on the partition. Double-check the device name with `lsblk` before running `mkfs`.

---

## 📂 Mounting & Unmounting

**Mounting** is how Linux makes a partition accessible. You attach it to a directory — called a **mount point** — and everything stored on that partition appears at that path.

### Mount a Partition

```bash
# Create a mount point directory first
sudo mkdir /mnt/mydisk

# Mount the partition to it
sudo mount /dev/sdb1 /mnt/mydisk
```

Now `/mnt/mydisk` shows the contents of `/dev/sdb1`. Anything written to `/mnt/mydisk` is stored on that partition.

### Unmount a Partition

```bash
sudo umount /mnt/mydisk
```

> ⚠️ Always unmount before physically removing a disk. Unmounting while files are open or in use will fail — make sure no process is accessing the mount point first.

### Remount as Read-Write

```bash
sudo mount -o remount,rw /mnt/mydisk
```

Useful when a filesystem was mounted as read-only (often happens after an unclean shutdown).

---

### Making Mounts Persistent with `/etc/fstab`

By default, mounts don't survive a reboot. To make them permanent, add an entry to `/etc/fstab`:

```bash
# Get the UUID of your partition
sudo blkid /dev/sdb1
```

Add this line to `/etc/fstab`:

```
UUID=your-uuid-here   /mnt/mydisk   ext4   defaults   0   2
```

Test it without rebooting:

```bash
sudo mount -a
```

> 💡 Always use **UUID** in fstab, not device names — device names can change between reboots if you add/remove disks.

---

## 📦 Logical Volume Management (LVM)

LVM sits between your physical disks and the filesystem, giving you the ability to **resize volumes on the fly** without unmounting or repartitioning.

**The three layers of LVM:**

```
Physical Disks  →  Physical Volumes (PV)
                         ↓
              Volume Groups (VG)  ←── pool of storage
                         ↓
              Logical Volumes (LV)  ←── what you format and mount
```

### Setting Up LVM from Scratch

```bash
# Step 1: Initialise disks as Physical Volumes
sudo pvcreate /dev/sdb /dev/sdc

# Step 2: Create a Volume Group from those PVs
sudo vgcreate my_vg /dev/sdb /dev/sdc

# Step 3: Create a Logical Volume from the VG
sudo lvcreate -L 15G -n my_lv my_vg

# Step 4: Format the Logical Volume
sudo mkfs.ext4 /dev/my_vg/my_lv

# Step 5: Mount it
sudo mkdir /data
sudo mount /dev/my_vg/my_lv /data
```

### Extending a Logical Volume (Without Downtime)

This is where LVM really shines — you can grow a volume while it's live and mounted:

```bash
# Extend the logical volume by 5GB
sudo lvextend -L +5G /dev/my_vg/my_lv

# Resize the filesystem to use the new space
sudo resize2fs /dev/my_vg/my_lv

# Verify
df -h /data
```

> 💡 This is the main reason LVM is used in production servers — resizing storage without taking services offline.

---

## 🔄 Swap Management

Swap is disk space Linux uses as overflow when RAM is full. It's slower than RAM but prevents out-of-memory crashes.

```bash
# Check current swap usage
swapon --show
free -h

# Create a swap partition (on an existing partition)
sudo mkswap /dev/sdX

# Enable it
sudo swapon /dev/sdX

# Disable it
sudo swapoff /dev/sdX
```

**Creating a swapfile instead (no partition needed):**

```bash
# Create a 2GB swapfile
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make it permanent in /etc/fstab
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

> 💡 Swapfiles are more flexible than swap partitions — you can resize or remove them without repartitioning. Preferred on modern systems.

---

## 🛠️ Real-World Scenario — Full New Disk Setup

Here's the complete workflow when you get a brand new disk (`/dev/sdb`) and want to make it usable:

```bash
# 1. Confirm the disk is visible
lsblk

# 2. Partition the disk
sudo fdisk /dev/sdb
#    → n (new partition) → p (primary) → accept defaults → w (write)

# 3. Verify the partition was created
lsblk

# 4. Format the partition
sudo mkfs.ext4 /dev/sdb1

# 5. Create a mount point
sudo mkdir /data

# 6. Mount it
sudo mount /dev/sdb1 /data

# 7. Verify it's mounted and showing available space
df -h /data

# 8. Make it persistent (survives reboots)
sudo blkid /dev/sdb1          # copy the UUID
sudo nano /etc/fstab
# add: UUID=<your-uuid>  /data  ext4  defaults  0  2

# 9. Test fstab entry
sudo mount -a
```

---

## 🔍 When to Use What — Decision Guide

| Situation                                | What to use                |
| ---------------------------------------- | -------------------------- |
| Just want to see what disks exist        | `lsblk`                    |
| New disk, need to partition it first     | `fdisk` or `parted`        |
| Partition exists, just need to access it | `mount`                    |
| Need to resize storage without downtime  | **LVM**                    |
| Disk over 2TB or using UEFI              | `parted` (GPT support)     |
| Need mount to survive a reboot           | `/etc/fstab` with UUID     |
| Low on RAM, need overflow space          | Swap partition or swapfile |

---

## 📋 Full Cheatsheet

```bash
# ─── VIEWING ───────────────────────────────────────────
lsblk                            # Tree view of all disks
sudo fdisk -l                    # Full partition table details
sudo blkid                       # UUIDs and filesystem types
df -h                            # Space usage per filesystem
du -sh /path                     # Size of a directory

# ─── PARTITIONING ──────────────────────────────────────
sudo fdisk /dev/sdX              # Interactive partition editor (MBR)
sudo parted /dev/sdX             # Partition editor (GPT/large disks)

# ─── FORMATTING ────────────────────────────────────────
sudo mkfs.ext4 /dev/sdX1        # Format as ext4
sudo mkfs.xfs /dev/sdX1         # Format as XFS

# ─── MOUNTING ──────────────────────────────────────────
sudo mkdir /mnt/point            # Create mount point
sudo mount /dev/sdX1 /mnt/point  # Mount partition
sudo umount /mnt/point           # Unmount partition
sudo mount -a                    # Mount everything in fstab
sudo mount -o remount,rw /mnt    # Remount as read-write

# ─── LVM ───────────────────────────────────────────────
sudo pvcreate /dev/sdX           # Create physical volume
sudo vgcreate vg_name /dev/sdX   # Create volume group
sudo lvcreate -L 10G -n lv_name vg_name   # Create logical volume
sudo lvextend -L +5G /dev/vg/lv  # Extend logical volume
sudo resize2fs /dev/vg/lv        # Resize filesystem after extend

# ─── SWAP ──────────────────────────────────────────────
swapon --show                    # Show active swap
sudo mkswap /dev/sdX             # Prepare swap partition
sudo swapon /dev/sdX             # Enable swap
sudo swapoff /dev/sdX            # Disable swap
```

---

## 🚀 Key Takeaways

- Always start with `lsblk` — understand what's there before making any changes
- The full new disk workflow is: **partition → format → mount → fstab**
- Use **UUID in fstab**, never device names — they can change between reboots
- **LVM** is the production-grade solution for flexible, resizable storage
- **Swapfiles** are more flexible than swap partitions on modern systems
- `fdisk` for MBR/smaller disks — `parted` for GPT/disks over 2TB
- Formatting destroys data — always double-check the device name with `lsblk` first

---

_Part of my Linux & DevOps upskilling journey — documented publicly on GitHub._  
_Feel free to use this as a reference or a starting point for your own notes._
