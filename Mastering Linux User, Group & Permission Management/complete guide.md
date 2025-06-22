# 🟤 Mastering Linux User, Group & Permission Management (Module 6)

Managing users, groups, and permissions is fundamental for securing any Linux system. This module will equip you with the essential knowledge and practical skills for robust access control.

-----

## 👤 1. Creating Users & Groups

Linux identifies users and groups internally with unique numerical IDs: User ID (UID) for users and Group ID (GID) for groups.

| 🔧 Command | 🧠 Use                                       | 🔍 Real-World Example                               |
| :--------- | :------------------------------------------- | :-------------------------------------------------- |
| `useradd`  | Creates a new user account                   | `sudo useradd devuser -m -s /bin/bash` (Creates a user 'devuser', creates their home directory, and sets their default shell to bash) |
| `groupadd` | Creates a new group                          | `sudo groupadd developers` (Creates a new group called 'developers') |
| `passwd`   | Sets or changes a user's password            | `sudo passwd devuser` (Prompts to set a password for 'devuser') |
| `usermod`  | Modifies an existing user account            | `sudo usermod -aG developers devuser` (Adds 'devuser' to the 'developers' group) |
| `gpasswd`  | Manages group membership (alternative to `usermod`) | `sudo gpasswd -a devuser developers` (Adds 'devuser' to 'developers' group) |

-----

### 🧱 ASCII User/Group Hierarchy: UID → username → groups

```ascii
+-----------------------+
|  Linux System Users   |
+-----------------------+
           |
           V
+-----------------------+      +-----------------------+
|   User Account (UID)  |----->|   Group Account (GID) |
|  - unique ID          |      |  - unique ID          |
|  - associated with    |      |  - defines access for |
|    a username         |      |    collections of users|
+-----------------------+      +-----------------------+
           |                             |
           V                             V
+-----------------------+      +-----------------------+
|    Primary Group      |      |   Secondary Groups    |
|  (usually same GID    |<-----+ (user can belong to  |
|   as UID by default)  |      |   multiple secondary  |
+-----------------------+      |   groups)             |
                               +-----------------------+
```

#### 🧮 Table: Essential User & Group Configuration Files

These plain text files store crucial information about users and groups.

| File          | Purpose                                        | Format (Colon-Separated Fields)                            | Access        |
| :------------ | :--------------------------------------------- | :--------------------------------------------------------- | :------------ |
| `/etc/passwd` | User account information (except passwords)    | `username:x:UID:GID:GECOS:home_directory:shell`            | World-readable |
| `/etc/shadow` | Encrypted user passwords and password aging    | `username:encrypted_password:last_change:min_days:max_days:warn_days:inactive_days:expire_date:reserved` | Root only    |
| `/etc/group`  | Group information and membership               | `groupname:x:GID:member1,member2,...`                      | World-readable |

**Real Terminal Output:**

```bash
# Add a new user 'devuser' and group 'developers'
sudo useradd -m -s /bin/bash devuser
sudo groupadd developers

# Set password for devuser
sudo passwd devuser
# Output:
# New password:
# Retype new password:
# passwd: password updated successfully

# Add devuser to the 'developers' group
sudo usermod -aG developers devuser

# Verify user and group existence
grep '^devuser:' /etc/passwd
# Output:
# devuser:x:1001:1001:,,,:/home/devuser:/bin/bash

grep '^developers:' /etc/group
# Output:
# developers:x:1002:devuser

# Check devuser's groups
groups devuser
# Output:
# devuser : devuser developers
```

-----

## 🔐 2. Managing File & Directory Permissions

Permissions define who can *read*, *write*, or *execute* files and directories.

| 🔧 Command | 🔍 Purpose                           | 🔒 Use Case                                            |
| :--------- | :----------------------------------- | :----------------------------------------------------- |
| `chmod`    | Change file/directory permissions    | Limit access to sensitive config files, make scripts executable |
| `chown`    | Change file/directory owner          | Transfer ownership to a new user or service account    |
| `chgrp`    | Change file/directory group ownership | Assign a file to a specific group for collaborative access |

### 🧠 Diagram: Permission bits `rwxr-xr--` Breakdown

Linux file permissions are represented by 9 characters, often seen in `ls -l` output. These 9 characters are divided into three triplets, representing permissions for the **Owner**, **Group**, and **Others**.

```ascii
FILE TYPE     OWNER       GROUP       OTHERS
   |           |           |           |
   -           rwx         r-x         r--
   |           | | |       | | |       | | |
   |           R W X       R - X       R - -
   |           4 2 1       4 0 1       4 0 0
   |           (7)         (5)         (4)

Example: `chmod 754 file.txt`
```

  * **File Type (`-`):** The first character indicates the file type (`-` for regular file, `d` for directory, `l` for symbolic link, etc.).
  * **Owner Permissions (`rwx`):** Permissions for the user who owns the file.
  * **Group Permissions (`r-x`):** Permissions for users who are members of the file's group.
  * **Others Permissions (`r--`):** Permissions for all other users on the system.

#### 🎨 Visual Matrix: Octal to Symbolic Permissions

Permissions are often set using **octal notation**, where each permission (read, write, execute) corresponds to a numerical value.

  * **r** (read) = 4
  * **w** (write) = 2
  * **x** (execute) = 1
  * **-** (no permission) = 0

These values are summed for each triplet (owner, group, others).

| Octal | Binary | Symbolic | Permission Meaning          |
| :---- | :----- | :------- | :-------------------------- |
| **0** | 000    | `---`    | No permissions              |
| **1** | 001    | `--x`    | Execute only                |
| **2** | 010    | `-w-`    | Write only                  |
| **3** | 011    | `-wx`    | Write & Execute             |
| **4** | 100    | `r--`    | Read only                   |
| **5** | 101    | `r-x`    | Read & Execute              |
| **6** | 110    | `rw-`    | Read & Write                |
| **7** | 111    | `rwx`    | Read, Write, & Execute (Full) |

### 🔍 3. Understanding r, w, x for Users, Groups, Others (U/G/O)

The interpretation of `r`, `w`, and `x` differs slightly for files vs. directories.

| Entity | r (Read)                               | w (Write)                                    | x (Execute)                                  |
| :----- | :------------------------------------- | :------------------------------------------- | :------------------------------------------- |
| **File** | ✅ View file contents                  | ✅ Modify/delete file contents               | ✅ Run script/binary                         |
| **Directory** | ✅ List directory contents (`ls`)      | ✅ Create/delete/rename files within directory | ✅ Enter (`cd`) into directory, access its files |

#### 📁 Use Case Examples:

**1. Secure SSH Key Folder:** `chmod 700 ~/.ssh`

```bash
# Create a dummy .ssh directory and a key file for demonstration
mkdir -p ~/.ssh_demo
touch ~/.ssh_demo/id_rsa
ls -ld ~/.ssh_demo # Check initial permissions (might be 775 or 755)
# Output might look like: drwxrwxr-x 2 user user 4096 Jun 22 21:00 /home/user/.ssh_demo/

chmod 700 ~/.ssh_demo
chmod 600 ~/.ssh_demo/id_rsa # Important for private keys

ls -ld ~/.ssh_demo
# Output: drwx------ 2 user user 4096 Jun 22 21:00 /home/user/.ssh_demo/

ls -l ~/.ssh_demo/id_rsa
# Output: -rw------- 1 user user 0 Jun 22 21:00 /home/user/.ssh_demo/id_rsa
```

**Explanation:** This restricts access to the `.ssh_demo` directory and `id_rsa` file to the owner (you) only, providing maximum security for your private keys. SSH daemons will often refuse to use keys if permissions are too liberal.

**2. Collaborative Project Folder for Dev Team:** `chmod 775 project/`

```bash
# Create a project directory and assign it to the 'developers' group
mkdir project_dev
sudo chown :developers project_dev
sudo chmod 775 project_dev
ls -ld project_dev
# Output: drwxrwxr-x 2 user developers 4096 Jun 22 21:00 project_dev
```

**Explanation:**

  * **Owner (youruser):** `rwx` (7) - Can read, write, execute (create/delete/modify files).
  * **Group (developers):** `rwx` (7) - All members of `developers` group can read, write, execute (collaborate fully).
  * **Others:** `r-x` (5) - Anyone else can read files and enter the directory but *cannot* create, delete, or modify files.

This is ideal for shared development environments where team members need full access, but external users should only be able to view.

-----

## 🎭 4. Umask & Default Permissions

`umask` (user mask) is a setting that controls the default permissions assigned to newly created files and directories. It's a security mechanism to ensure new files are not created with overly permissive settings.

| Command | 🔍 Use                              |
| :------ | :---------------------------------- |
| `umask` | Sets or displays the default permission mask |

### 📘 Example: `umask 022`

The `umask` value is subtracted from the default maximum permissions.

  * **Default maximum for files:** `666` (rw-rw-rw-)
  * **Default maximum for directories:** `777` (rwxrwxrwx)

**Formula:** `Default maximum - umask = Final permission`

Let's say `umask` is `022`:

  * **For a new file:** `666 - 022 = 644` (rw-r--r--)
  * **For a new directory:** `777 - 022 = 755` (rwxr-xr-x)

**Real Terminal Output:**

```bash
# Check current umask
umask
# Output: 0022 (often displayed with a leading zero)

# Create a file and directory to see default permissions
touch new_file.txt
mkdir new_directory

ls -l new_file.txt
# Output: -rw-r--r-- 1 user group 0 Jun 22 21:00 new_file.txt

ls -ld new_directory
# Output: drwxr-xr-x 2 user group 4096 Jun 22 21:00 new_directory

# Change umask to be more restrictive (e.g., for secure keys)
umask 077
# Now try creating new files/directories
touch restricted_file.txt
mkdir restricted_dir

ls -l restricted_file.txt
# Output: -rw------- 1 user group 0 Jun 22 21:00 restricted_file.txt

ls -ld restricted_dir
# Output: drwx------ 2 user group 4096 Jun 22 21:00 restricted_dir
```

#### 🎨 Visual: `umask` Calculation

```ascii
Default File Perms (666) :  rw-rw-rw-   (110 110 110)
umask (022)              : -  --w-w-    (000 010 010) (NOTE: umask bits are SUBTRACTED/DISABLED)
---------------------------------------------------
Final File Perms (644)   :  rw-r--r--   (110 100 100)

Default Dir Perms (777)  :  rwxrwxrwx   (111 111 111)
umask (022)              : -  --w-w-    (000 010 010) (NOTE: umask bits are SUBTRACTED/DISABLED)
---------------------------------------------------
Final Dir Perms (755)    :  rwxr-xr-x   (111 101 101)
```

#### 🧪 Tips for `umask`:

  * **Persistent `umask`:** To make `umask` persistent for a user, modify `umask` in `~/.bashrc` or `~/.profile`. For system-wide changes, edit `/etc/profile` or a file in `/etc/profile.d/`.
  * **Secure Environments:** Use `umask 077` in highly secure environments (e.g., for root or service accounts handling private keys) to ensure all newly created files and directories are only accessible by the owner.

-----

## 💼 Interview Questions & Answers

| ❓ Question                                 | ✅ Best Answer                                                                                                                                                                                                                                                                                           |
| :------------------------------------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What's the difference between `chmod` and `chown`? | `chmod` (change mode) modifies file or directory permissions (read, write, execute). `chown` (change owner) changes the owner user and/or group of a file or directory. |
| How do `umask` values affect file creation? | `umask` specifies which permission bits are *removed* from the default permissions (666 for files, 777 for directories) when new files or directories are created, enforcing a more secure default. |
| What does `chmod 755` mean?               | `chmod 755` grants the file owner full read, write, and execute permissions (`rwx`), while the group and others are granted read and execute permissions (`r-x`).                                                                                |
| How do you add a user to a group?           | The primary command is `usermod -aG groupname username`. The `-a` flag appends the user to the group without removing them from other groups, and `-G` specifies the supplementary group. Alternatively, `gpasswd -a username groupname` can be used. |
| Why is `chmod 777` dangerous?               | `chmod 777` grants read, write, and execute permissions to *everyone* (owner, group, and others). This poses a significant security risk as any user on the system could view, modify, or execute the file, potentially leading to data corruption, unauthorized access, or privilege escalation. |

-----

## 🚀 Best Practices

  * **🔒 Principle of Least Privilege:** Always grant only the minimum necessary permissions for users and services to perform their tasks.
  * **👥 Use Groups Effectively:** Organize users into logical groups to manage shared access to resources efficiently. Instead of setting individual permissions, set group permissions and add users to the relevant group.
  * **📁 Backup Sensitive Folders:** Regularly back up permission-sensitive directories like `/etc/` to restore system configurations in case of accidental permission changes.
  * **🚫 Avoid `chmod 777`:** Never use `chmod 777` on production systems or any sensitive files. It's a critical security vulnerability.
  * **👁️ Audit Permissions:** Periodically review file and directory permissions, especially for critical system files and web content.
  * **✅ Verify after Changes:** Always use `ls -l` after `chmod`, `chown`, or `chgrp` to verify the changes have been applied correctly.

-----

## 📊 Real-World Use Cases

| 🏢 Use Case             | 🔐 Permissions Strategy                                                                                                                                          |
| :---------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Web Server (Apache/Nginx) Document Root** | Set document root (`/var/www/html`) to be owned by `root:www-data` (or `apache`/`nginx` group). Directories typically `chmod 775` or `755`, files `chmod 664` or `644`. Web server process needs read access. |
| **Dev Team Project Folder (shared)** | Create a dedicated group (e.g., `devs`). Set folder ownership to `user:devs`. Use `chmod 2775` (the '2' sets SGID bit, ensuring new files/dirs inherit the group). This allows all `devs` members to create/modify files. |
| **SSH Key Management** | Ensure `~/.ssh` directory is `chmod 700` and individual private keys (e.g., `id_rsa`) are `chmod 600`. Public keys (`id_rsa.pub`, `authorized_keys`) are typically `chmod 644`.                                      |
| **Shared Media Folder** | Create a `mediausers` group. Set folder ownership to `user:mediausers`. Use `chmod 770` or `775` depending on public access requirements. Members of `mediausers` can share. |

-----

### Flowchart: File Access Check Logic

When a user tries to access a file, the Linux kernel follows a specific logic to determine if access is permitted:

```ascii
+-----------------------+
|  User Attempts Access |
|     (e.g., cat file)  |
+-----------+-----------+
            |
            V
+-----------------------+
|  1. Is User the File  |
|      Owner (UID)?     |
+-----------+-----------+
      | Yes         | No
      V             V
+----------------+  +-----------------------+
|  Check Owner   |  |  2. Is User in File's |
|  Permissions   |  |      Group (GID)?     |
|   (rwx1)       |  +-----------+-----------+
+-------+--------+        | Yes         | No
        |                 V             V
        |          +----------------+  +----------------+
        |          |  Check Group   |  |  Check Others  |
        |          |  Permissions   |  |  Permissions   |
        |          |   (rwx2)       |  |   (rwx3)       |
        |          +-------+--------+  +-------+--------+
        |                  |                   |
        +------------------+-------------------+
                           |
                           V
+---------------------------------------+
|  Is Requested Access (Read/Write/Exec)|
|  Allowed by the Matched Permissions?  |
+---------------------------------------+
            | Yes                  | No
            V                      V
    +---------------+        +-------------------+
    |  Access Granted! |        |  Permission Denied! |
    +---------------+        +-------------------+
