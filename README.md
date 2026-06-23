# Linux Learning Notes

This repository contains practical Linux notes and command references for learning Linux administration, troubleshooting, and DevOps foundations.

The content is organized by topic, with each numbered folder focusing on one area of Linux.

## Contents

| Section | Topic | What it covers |
| ------- | ----- | -------------- |
| [01-Introduction](01-Introduction/) | Linux introduction | Why Linux matters, basic structure, distributions, setup, and package managers |
| [02-Structure](02-Structure/Structure.md) | Linux filesystem structure | Important directories and how the Linux filesystem is organized |
| [03-userManagement](03-userManagement/userManagement.md) | User management | Users, groups, account commands, and basic administration |
| [04-FileManagement](04-FileManagement/FileManagement.md) | File management | Creating, viewing, copying, moving, deleting, and editing files |
| [05-Shortcuts](05-Shortcuts/shortcuts.md) | Terminal shortcuts | Useful shell shortcuts for faster command-line work |
| [06-FilePermissions](06-FilePermissions/File_Permissions.md) | File permissions | Ownership, permission bits, chmod, chown, and access control basics |
| [07-ProceeManagement](07-ProceeManagement/ProcessManagement.md) | Process management | Viewing, managing, and stopping Linux processes |
| [08-Monitoring](08-Monitoring/Monitoring.md) | System monitoring | CPU, memory, disk, network, and log monitoring commands |
| [09-DiskManagement](09-DiskManagement/DiskManagement.md) | Disk management | Disks, partitions, filesystems, mounts, LVM, and swap |
| [10-Networking](10-Networking/Networking.md) | Networking | IP addresses, routes, DNS, connectivity tests, ports, firewalls, and troubleshooting |

## Suggested Learning Order

1. Start with [01-Introduction](01-Introduction/) to understand what Linux is and how it is commonly used.
2. Read [02-Structure](02-Structure/Structure.md) to understand where files and configuration live.
3. Practice [04-FileManagement](04-FileManagement/FileManagement.md), [05-Shortcuts](05-Shortcuts/shortcuts.md), and [06-FilePermissions](06-FilePermissions/File_Permissions.md).
4. Move into administration topics: users, processes, monitoring, disks, and networking.
5. Revisit the command indexes when troubleshooting real systems.

## Repository Goal

The goal of this repo is to act as a beginner-friendly Linux reference that is still practical enough for real terminal use.

Each guide focuses on:

- Common commands
- What each command does
- Example usage
- Important warnings
- Practical troubleshooting habits

## Notes

Some commands require `sudo` because they inspect or change system-level settings. Always double-check command targets before running destructive commands, especially disk, permission, firewall, and process-management commands.
