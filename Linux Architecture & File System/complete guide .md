# 🚀 Mastering Linux Architecture & File System (Module 3)


> 📌 **Data from trusted sources:** Official docs, Red Hat, Ubuntu, Linux Foundation, GeeksforGeeks, YouTube: The Linux Foundation, TechWorld with Nana, NetworkChuck, DorianDotSlash

-----

## 🎯 Module Overview

This module dives into the core components of the Linux operating system, from its fundamental architecture to its file system structure and security mechanisms. Understanding these concepts is essential for any Linux user, system administrator, or DevOps engineer.

### 1\. 🧬 What is Linux Architecture?

Linux architecture is a layered structure that defines how the different components of the operating system interact. The key layers are the **Kernel**, the **Shell**, and the **User Space**.

  * **Kernel:** The core of the OS. It directly interacts with the hardware, managing resources like the CPU, memory, and I/O devices.
  * **Shell:** A command-line interpreter that allows users to interact with the kernel. It takes user commands and translates them into kernel instructions. Examples include Bash, Zsh, and Fish.
  * **User Space:** The area where user applications and programs run. This space is isolated from the kernel to prevent system instability.

**Monolithic vs. Microkernel:**

  * **Monolithic Kernel:** Linux uses a monolithic kernel, where most OS services (device drivers, file systems, networking) run within the kernel space. This design generally offers better performance but can be less modular and more complex to maintain.
  * **Microkernel:** In contrast, a microkernel keeps only essential services in the kernel, with other services running in user space. This offers greater modularity and stability but can introduce performance overhead due to inter-process communication.

**Process & Memory Management:**

  * **Process Management:** The kernel is responsible for creating, scheduling, and terminating processes. Each process has its own address space, and the kernel ensures that processes don't interfere with each other.
  * **Memory Management:** The kernel allocates and manages memory for processes, ensuring efficient use of system resources. It uses techniques like virtual memory and swapping to handle memory demands.

**Visual Representation:**

```ascii
+---------------------------+
|        User Space         |  <-- Applications, Shells, Utilities
+---------------------------+
             | System Calls
+---------------------------+
|          Kernel           |  <-- Core OS: Process, Memory, I/O Management
+---------------------------+
             | Hardware Interaction
+---------------------------+
|         Hardware          |  <-- CPU, RAM, Disk, Network
+---------------------------+
```

**Real Kernel Use Case (I/O):**

When a user application needs to read data from a file, it makes a system call. The kernel receives this request, interacts with the disk driver to retrieve the data, and then passes the data back to the application.

**Source:** [Linux Kernel Documentation](https://www.kernel.org/) and [GeeksforGeeks: Linux Kernel](https://www.geeksforgeeks.org/linux-kernel/)

-----

### 2\. 🕹️ Linux Boot Process

The Linux boot process is the sequence of steps that starts when you turn on your computer and ends when you see the login prompt.

1.  **BIOS/UEFI:** The Basic Input/Output System (BIOS) or Unified Extensible Firmware Interface (UEFI) initializes the hardware and performs a power-on self-test (POST). UEFI is the modern replacement for BIOS, offering faster boot times and more features.
2.  **GRUB:** The Grand Unified Bootloader (GRUB) is loaded from the boot sector. It presents a menu allowing you to choose which operating system or kernel to boot.
3.  **Kernel:** The Linux kernel is loaded into memory and starts executing. It initializes devices and mounts the root file system.
4.  **Init/Systemd:** The `init` process (or `systemd` on modern systems) is the first process to run after the kernel. It starts other system services and processes. `systemd` is a more advanced init system that manages services in parallel, improving boot times.
5.  **Target:** `systemd` uses the concept of "targets" to define system states. The `graphical.target` starts the graphical login screen, while `multi-user.target` starts a command-line login.
6.  **Login:** The login process allows users to authenticate and access the system.

**Boot Sequence Flowchart:**

```mermaid
graph LR
    A[Power On] --> B(BIOS/UEFI);
    B --> C{GRUB};
    C --> D[Kernel];
    D --> E{{systemd/init}};
    E --> F[Target (e.g., graphical.target)];
    F --> G(Login Prompt);
```

**Real Boot Logs:**

  * `dmesg`: Displays kernel-related messages, including hardware detection and driver initialization.

    ```bash
    dmesg | less
    ```

  * `journalctl`: Shows systemd journal logs, providing detailed information about the boot process and service startup.

    ```bash
    journalctl -b | less
    ```

**YouTube Source:** [TechWorld with Nana - Linux Boot Process Explained](https://www.youtube.com/watch?v=8mjJHIXQMcA)

-----

### 3\. 🗂️ Linux Filesystem Hierarchy (FHS)

The Filesystem Hierarchy Standard (FHS) defines the directory structure and directory names in Linux. It provides a consistent way to organize files and directories.

**Key Directories:**

  * `/`: The root directory. All other directories are subdirectories of the root.
  * `/root`: The home directory for the root user.
  * `/home`: Contains home directories for regular users.
  * `/etc`: Contains system-wide configuration files.
  * `/var`: Contains variable data, such as logs, spool files, and temporary files.
  * `/bin`: Contains essential command-line binaries.
  * `/usr`: Contains user-related programs and data.
  * `/mnt`: Used for mounting temporary filesystems.
  * `/dev`: Contains device files, representing hardware devices.
  * `/tmp`: Stores temporary files that are deleted on reboot.
  * `/proc`: A virtual filesystem that provides information about processes.
  * `/sys`: A virtual filesystem that provides information about the kernel and hardware.

**Tree-Style ASCII Diagram:**

```ascii
.
├── bin
├── boot
├── dev
├── etc
│   ├── passwd
│   └── group
├── home
│   └── user1
├── lib
├── media
├── mnt
├── proc
├── root
├── run
├── sbin
├── snap
├── srv
├── sys
├── tmp
├── usr
│   ├── bin
│   └── lib
└── var
    ├── log
    └── tmp
```

**Role-Based Color-Coded Chart (Example):**

| Directory | Purpose                       | Color Code |
| :-------- | :---------------------------- | :--------- |
| `/etc`    | Configuration Files           | Red        |
| `/home`   | User Data                     | Green      |
| `/var`    | Variable Data (Logs, etc.)   | Blue       |
| `/bin`    | Essential Binaries            | Yellow     |
| `/dev`    | Device Files                  | Cyan       |

**Path Usage in Real Systems:**

  * Configuration files for Apache web server are typically located in `/etc/apache2/`.
  * User home directories are found in `/home/username/`.
  * System logs are stored in `/var/log/`.

**Source:** [Linux Foundation FHS Guide](https://refspecs.linuxfoundation.org/fhs.shtml)

-----

### 4\. 🔐 File & Directory Permissions

Linux file permissions control who can read, write, and execute files and directories.

**Concepts:**

  * `chmod`: The command used to change file permissions.
  * `umask`: Sets the default permissions for newly created files and directories.
  * `ls -l`: Lists files and directories with detailed information, including permissions.
  * `+x`: Adds execute permission.
  * `-rw-r--r--`: Represents read and write permissions for the owner, read-only for the group and others.

**rwx Permission Table:**

| Permission | Symbol | Numeric Value | Description                                   |
| :--------- | :----- | :------------ | :-------------------------------------------- |
| Read       | `r`    | 4             | Allows viewing the contents of a file.          |
| Write      | `w`    | 2             | Allows modifying the contents of a file.        |
| Execute    | `x`    | 1             | Allows running a file as a program.            |

**chmod Value Breakdown:**

The `chmod` command uses numeric or symbolic modes to set permissions. Numeric mode uses a three-digit number, where each digit represents the permissions for the owner, group, and others.

  * `755`: Owner: read, write, execute; Group: read, execute; Others: read, execute.
  * `644`: Owner: read, write; Group: read; Others: read.
  * `700`: Owner: read, write, execute; Group: no access; Others: no access.

**Before/After Permission Changes:**

```bash
# Before
ls -l myfile.txt
-rw-r--r-- 1 user group 1024 Jun 22 20:00 myfile.txt

# Change to 700 (rwx------)
chmod 700 myfile.txt

# After
ls -l myfile.txt
-rwx------ 1 user group 1024 Jun 22 20:00 myfile.txt
```

**SUID, SGID, Sticky Bit:**

  * **SUID (Set User ID):** When set on an executable, it runs with the permissions of the *owner* of the file, not the user running it.
  * **SGID (Set Group ID):** When set on an executable, it runs with the permissions of the *group* of the file. When set on a directory, new files inherit the group ID of the directory.
  * **Sticky Bit:** When set on a directory, only the owner of a file, the owner of the directory, or the root user can delete or rename files within that directory.

**YouTube Source:** [DorianDotSlash - Linux File Permissions Explained Visually](https://www.youtube.com/watch?v=ZtqBQ68cfJc)

-----

### 5\. 👥 File Ownership: Users & Groups

Linux uses users and groups to manage access control for files, directories, and processes.

**Commands:**

  * `chown`: Changes the owner and/or group of a file or directory.
  * `groups`: Displays the groups a user belongs to.
  * `usermod`: Modifies user account information.
  * `adduser`: Creates a new user.
  * `sudo`: Executes a command with superuser privileges.
  * `id`: Displays user and group IDs.

**Real Output of `ls -l`:**

```bash
ls -l myfile.txt
-rw-r--r-- 1 user group 1024 Jun 22 20:00 myfile.txt
```

In this output, `user` is the owner, and `group` is the group.

**Role of `/etc/passwd` and `/etc/group`:**

  * `/etc/passwd`: Stores basic user account information, such as username, user ID, home directory, and shell.
  * `/etc/group`: Stores group information, such as group name and group ID.

**ASCII Ownership Chart:**

```ascii
+-----------------------+
| File/Directory        |
+-----------------------+
| Owner: user1 (UID: 1000) |
| Group: devgroup (GID: 100) |
+-----------------------+
| Permissions: rw-r--r--  |
+-----------------------+
```

**Trusted Docs:**

  * [Ubuntu User Management Docs](https://ubuntu.com/server/docs/user-management)
  * [Linuxize - Linux Users and Groups](https://linuxize.com/post/how-to-add-remove-users-in-linux/)
