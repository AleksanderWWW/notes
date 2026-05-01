# Linux Filesystem Hierarchy (FHS) Explained

This guide provides a systematic overview of the standard Linux directory structure.



## The Root Directory (`/`)
Every single file and directory in Linux starts at the root. Even if you mount a separate physical hard drive, it must be attached (mounted) somewhere under `/`.

### Primary System Directories

| Directory | Purpose | Key Content |
| :--- | :--- | :--- |
| `/bin` | **Essential Binaries** | Commands needed in single-user mode (e.g., `ls`, `cp`, `bash`). |
| `/sbin` | **System Binaries** | Essential tools for the root user/system maintenance (e.g., `iptables`, `reboot`). |
| `/etc` | **System Configuration** | Host-specific configuration files (e.g., `fstab`, `passwd`, `sysctl.conf`). |
| `/lib` | **Essential Libraries** | Shared library files required by the binaries in `/bin` and `/sbin`. |
| `/boot` | **Boot Loader** | Files needed to boot the system (Kernel images, Initrd, GRUB config). |

---

## Variable and Temporary Data

### `/var` (Variable Files)
This directory contains data that is expected to grow and change frequently. It is often placed on a separate partition to prevent log files from filling up the root disk.
*   `/var/log`: System and application logs.
*   `/var/lib`: State information (e.g., the contents of your Postgres database).
*   `/var/mail`: User mailboxes.
*   `/var/spool`: Print queues and cron jobs.

### `/tmp` (Temporary Files)
Used by applications to store temporary data. On many modern systems (like Ubuntu), this is actually a `tmpfs` (a virtual filesystem running in RAM), meaning it is wiped on every reboot.

You can try that by running

```bash
echo "test" > /tmp/test
sudo systemctl reboot
```

then once the machine is back online run

```bash
cat /tmp/test
```

You should see

```terminaloutput
cat: /tmp/test: No such file or directory
```

---

## User and Resource Directories

### `/home` (User Data)
Personal directories for regular users. This is where your documents, SSH keys (`.ssh/`), and shell configs (`.bashrc`) live.

### `/root` (Root Home)
The home directory for the `root` user. It is located here instead of `/home/root` to ensure it is available even if the `/home` partition fails to mount.

### `/usr` (User Resources)
Historically stood for "Unix System Resources." It contains the bulk of the system’s software.
*   `/usr/bin`: Non-essential binaries for users (e.g., `python`, `git`, `vim`).
*   `/usr/local`: Software you install manually (e.g., compiled from source) to keep it separate from system-managed packages.

---

## Virtual and Device Filesystems

These are "pseudo-filesystems"—they don't exist on your hard drive; they are interfaces provided by the kernel.

| Directory | Purpose | Example |
| :--- | :--- | :--- |
| `/dev` | **Device Files** | Hardware interfaces (e.g., `/dev/sda` for a disk, `/dev/null`). |
| `/proc` | **Process Info** | Kernel and process information (e.g., `/proc/cpuinfo`). |
| `/sys` | **System Info** | Modern interface for kernel/hardware parameters (e.g., battery/thermal info). |

---

## Optional and Mount Points

*   `/opt`: Add-on application software packages (e.g., Google Chrome or large proprietary suites).
*   `/mnt`: Temporary mount point for filesystems (e.g., mounting an external drive).
*   `/media`: Automatic mount point for removable media (CDs, USB sticks).

---

## Example for software engineers

Let's say you are hosting your application on a Ubuntu server.
This is the setup you may want to follow:

*   **The Binary:** Should go in `/usr/local/bin/`.
*   **The Config:** Should go in `/etc/my-app/`.
*   **The Logs:** Should go in `/var/log/my-app/`.
*   **The Service File:** Should go in `/etc/systemd/system/`
