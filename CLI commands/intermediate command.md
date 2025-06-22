
# 🧠 Mastering Intermediate Linux CLI Commands (Module 5)

This module escalates your command-line prowess, focusing on powerful tools for file searching, text processing, archiving, and disk usage monitoring. These are the workhorses for DevOps, system administration, and automation.

-----

## 🔍 1. **File Search & Identification**

Knowing where your files are and what they are is foundational.

| 🛠️ Command    | 🔎 Purpose                           | 📌 Real-World Use                             | Analogy                                                       |
| :------------ | :----------------------------------- | :-------------------------------------------- | :------------------------------------------------------------ |
| `find`        | Recursively search for files/directories based on criteria | Daily operations, security audits, cleanup scripts, locating specific config files | Like a detective methodically searching every room in a house based on clues. |
| `locate`      | Fast indexed search for file names   | Bulk lookup of file locations, quick checks for installed software | Like looking up a phone number in a pre-compiled directory.     |
| `which`       | Find executable location in `PATH`   | Debugging path errors, verifying command versions | Asking "Where is this program actually stored?"                  |
| `whereis`     | Locate binary, source, and man pages | Sysadmin path checks, understanding installed packages | Asking "Where is this tool's program, source code, and instruction manual?" |

-----

### 🧠 Visuals & Real-World Examples

#### `find` vs `locate`: Search Logic Flowchart

```ascii
+----------------+       +-------------------+
|     locate     |       |       find        |
+----------------+       +-------------------+
        |                         |
        | Reads from              | Scans filesystem
        | Database                | (real-time)
        | (updated by updatedb)   |
        v                         v
+-----------------+       +-----------------+
|   Fast Lookup   |       |  Flexible, Deep |
| (Filename only) |       |  Search (Name,  |
|                 |       |  Type, Size,    |
|                 |       |  Permissions,   |
|                 |       |  Time, Owner)   |
+-----------------+       +-----------------+
```

**Comparison:** `locate` is about speed for names you vaguely remember. `find` is about precision and detail, searching every corner.

#### `find` - Deep Dive Example:

**Scenario:** You need to find all shell scripts (`.sh` files) in `/var/www/` that haven't been accessed in the last 7 days.

```bash
# First, let's create a sample structure for demonstration
mkdir -p /tmp/web_scripts/nginx_configs
touch /tmp/web_scripts/deploy.sh
touch /tmp/web_scripts/cleanup.py
touch /tmp/web_scripts/nginx_configs/site_backup.sh
touch /tmp/web_scripts/nginx_configs/old_script.sh # Simulate an old script
sleep 86400 # Wait for 1 day (for -atime to show effect, in real-world you'd wait longer)
touch /tmp/web_scripts/deploy.sh # Access this one today

find /tmp/web_scripts -name "*.sh" -type f -atime +7
# Expected output (after waiting 7+ days for 'old_script.sh'):
# /tmp/web_scripts/nginx_configs/old_script.sh

# More common: find all .sh files and list details
find /tmp/web_scripts -name "*.sh" -type f -print -ls
# Expected output (inode numbers and dates will vary):
# /tmp/web_scripts/deploy.sh
# 1234567 4 -rwxr-xr-x 1 user group 14 Jun 22 21:00 /tmp/web_scripts/deploy.sh
# /tmp/web_scripts/nginx_configs/site_backup.sh
# 1234568 4 -rw-r--r-- 1 user group  0 Jun 22 21:00 /tmp/web_scripts/nginx_configs/site_backup.sh
# /tmp/web_scripts/nginx_configs/old_script.sh
# 1234569 4 -rw-r--r-- 1 user group  0 Jun 15 21:00 /tmp/web_scripts/nginx_configs/old_script.sh
```

**Most Used Flags (`find`):**

  * `-name "pattern"`: Search by filename.
  * `-type [f/d/l]`: File (f), directory (d), symbolic link (l).
  * `-size [+-]N[c/k/M/G]`: Size greater/less than/equal to N (bytes/KB/MB/GB).
  * `-atime N`: Accessed N days ago.
  * `-mtime N`: Modified N days ago.
  * `-perm NNN`: Permissions.
  * `-exec command {} \;`: Execute a command on found files.

#### `locate` - Quick Lookup Example:

```bash
# Ensure locate database is up-to-date (run as root/sudo)
sudo updatedb

locate httpd.conf
# Expected output (examples):
# /etc/httpd/conf/httpd.conf
# /usr/share/doc/httpd-2.4.6/httpd.conf.example
```

**Highlight:** `locate` is extremely fast for simple filename searches as it queries a pre-built database, not the live filesystem.

#### `which` & `whereis` - Command Location:

```bash
which python3
# Expected output (example):
# /usr/bin/python3

whereis ls
# Expected output (example):
# ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz
```

**Highlight:** `which` is for executables in your `PATH`. `whereis` is broader, including source and manual pages.

-----

## 🧪 2. **File Processing & Text Parsing**

These commands are the backbone of data analysis, log parsing, and automation.

| 🔧 Command | 🧰 Description                                       | ⚙️ Use Case                                                | Analogy                                                   |
| :--------- | :-------------------------------------------------- | :--------------------------------------------------------- | :-------------------------------------------------------- |
| `grep`     | Searches for patterns in files or standard input    | Log scanning for errors, finding specific lines in config files, security auditing | Like a highlighter finding specific words in a document.  |
| `awk`      | Powerful text processing language, field manipulator | Parse CSV files, generate reports from structured data, extract specific columns based on complex logic | Like a data analyst extracting and transforming specific pieces of information from a spreadsheet. |
| `cut`      | Extracts specific columns (fields) from each line   | Filter fields from log files, extract usernames from `/etc/passwd` | Like using scissors to cut out specific columns from a newspaper. |
| `sort`     | Sorts lines of text alphabetically or numerically   | Process logs, sort output of other commands for readability | Arranging a stack of papers alphabetically or by size.    |
| `wc`       | Counts words, lines, and characters                 | Audit logs (e.g., how many lines in a log), quick file size check | Counting words, lines, or characters in an essay.         |

-----

### 🧠 Real Demo: Text Processing Pipeline

Let's use a common system file, `/etc/passwd`, and `ps aux` output for demonstration.

**Scenario 1: Find Nginx processes and count them.**

```bash
ps aux | grep nginx
# Example output (if nginx is running):
# youruser   1234  0.0  0.0 123456  4567 ?        Sl   Jun20   0:01 nginx: master process /usr/sbin/nginx
# www-data   1235  0.0  0.0 123456  4567 ?        S    Jun20   0:00 nginx: worker process
# www-data   1236  0.0  0.0 123456  4567 ?        S    Jun20   0:00 nginx: worker process
# youruser   5678  0.0  0.0   2640   600 pts/0    S+   21:45   0:00 grep --color=auto nginx

ps aux | grep nginx | wc -l
# Expected output (adjusting for grep process itself):
# 3  (meaning 3 nginx processes, if grep is excluded manually or refined)
# Better: ps aux | grep -c '[n]ginx'  (the brackets prevent grep from matching itself)
```

**Scenario 2: Extract usernames from `/etc/passwd` using `awk` and `cut`, then sort them.**

```bash
# Original /etc/passwd snippet (example):
# root:x:0:0:root:/root:/bin/bash
# daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
# youruser:x:1000:1000:Your User Name,,,:/home/youruser:/bin/bash
# systemd-resolve:x:101:102:systemd Resolver,,,/run/systemd/resolve:/bin/false

# Using cut to get the first field (username) delimited by ':'
cut -d':' -f1 /etc/passwd | head -n 5
# Expected output:
# root
# daemon
# bin
# sys
# sync

# Using awk to get the first field (username) delimited by ':'
awk -F':' '{ print $1 }' /etc/passwd | head -n 5
# Expected output:
# root
# daemon
# bin
# sys
# sync
```

**💡 `cut` vs `awk` Table:**

| Feature            | `cut`                                     | `awk`                                         |
| :----------------- | :---------------------------------------- | :-------------------------------------------- |
| **Primary Use** | Simple column extraction                  | Complex text processing, reporting, pattern action |
| **Flexibility** | Limited to simple delimiters and field numbers | Highly flexible, can use regex, conditions, variables, loops |
| **Delimiter** | Single character (`-d`)                   | Can be multiple characters or regex (`-F`), or whitespace by default |
| **Output Control** | Prints selected fields as-is              | Full control over output format (`print $1, $2`) |
| **Performance** | Generally faster for simple tasks         | Can be slower for simple tasks due to overhead, but excels at complex ones |

**Scenario 3: Pipeline Flow Diagram - Log Analysis Example**

**Task:** Find all lines containing "ERROR" in a log file, sort them, and count the occurrences.

```ascii
+----------------+       +---------------+       +---------------+       +-------------+
|    log.txt     | ----> |     grep      | ----> |     sort      | ----> |     wc      |
| (Raw Log Data) |       |  "ERROR"      |       |  (Alphabetical) |       |  -l (Count) |
+----------------+       +---------------+       +---------------+       +-------------+
        |                        |                       |                       |
        |                        V                       V                       V
        |                (Lines with "ERROR")  (Sorted Error Lines)  (Total Error Count)
        |
        +---------------------------------------------------------------------------------> Terminal Output
```

**Real Command Example:**

```bash
# Create a sample log file
echo "INFO: User logged in" > app.log
echo "ERROR: Database connection failed" >> app.log
echo "WARN: Disk space low" >> app.log
echo "ERROR: Failed to write to file" >> app.log
echo "INFO: Process started" >> app.log

cat app.log | grep "ERROR" | sort | wc -l
# Expected output:
# 2
```

-----

## 🗜️ 3. File Compression & Archiving

Essential for backups, saving disk space, and transferring files efficiently.

| 🧳 Command      | 🎯 Purpose                             | 💼 Use Case                                         | Analogy                                                 |
| :------------- | :------------------------------------- | :-------------------------------------------------- | :------------------------------------------------------ |
| `tar`          | Archive multiple files into a single file | Backup folders, packaging software releases, creating distribution archives | Like putting multiple items into one big box.             |
| `gzip`         | Compress `.tar` files or single files using Lempel-Ziv algorithm | Log rotation, individual file compression             | Shrink-wrapping the box to make it smaller.             |
| `zip` / `unzip` | Compress/decompress files and folders (cross-platform) | File exchange with Windows/macOS users, web downloads | Creating a standard "zip" package for easy sharing.       |
| `gunzip`       | Decompress `.gz` files                 | Restore `.gz` compressed backups, access log files    | Opening the shrink-wrap to get to the box.              |

-----

### 📦 Real Command Examples & Visuals

**Initial State:**

```ascii
my_project/
├── index.html
├── style.css
└── images/
    └── logo.png
```

**1. `tar` - Archiving (Boxing up files)**

```bash
tar -cvf project_archive.tar my_project/
# Expected output:
# my_project/
# my_project/index.html
# my_project/style.css
# my_project/images/
# my_project/images/logo.png
#
# Resulting files:
# project_archive.tar
# my_project/
```

**`tar` Flags:**

  * `-c`: Create archive
  * `-v`: Verbose (show files being added)
  * `-f`: Specify archive file name
  * `-x`: Extract from archive
  * `-z`: Use gzip compression (shorthand for `gzip` after `tar`)
  * `-j`: Use bzip2 compression
  * `-J`: Use xz compression

**2. `gzip` - Compressing the archive (Shrink-wrapping the box)**

```bash
gzip project_archive.tar
# Resulting files:
# project_archive.tar.gz (original .tar file is removed)
# my_project/
```

**Flow: Folder → tar → .tar → gzip → .tar.gz**

```ascii
+----------------+       +--------------+       +--------------+
|    my_project/ |       |     tar      |       |     gzip     |
| (Folder/Files) | ----> |  (Archiving) | ----> | (Compression)|
+----------------+       +--------------+       +--------------+
                                |                      |
                                V                      V
                         project_archive.tar    project_archive.tar.gz
```

**3. `tar -xvzf` - Extracting and Decompressing (Unpacking the box)**

```bash
# Clean up original folder to demonstrate extraction
rm -rf my_project/

tar -xvzf project_archive.tar.gz
# Expected output (files being extracted):
# my_project/
# my_project/index.html
# my_project/style.css
# my_project/images/
# my_project/images/logo.png
#
# Resulting files:
# project_archive.tar.gz
# my_project/ (recreated folder with contents)
```

**Highlight:** `tar -xvzf` is a powerful one-liner for decompressing a `.tar.gz` and extracting its contents.

**Comparison of Compression Rates (Conceptual):**

| Compression Tool | Typical Compression (General) | Notes                                       |
| :--------------- | :---------------------------- | :------------------------------------------ |
| `gzip`           | Good (e.g., 60-80%)           | Faster compression/decompression, widely used |
| `bzip2`          | Better (e.g., 70-85%)         | Slower, but often better compression        |
| `xz`             | Best (e.g., 75-90%)           | Slowest, but often highest compression      |
| `zip`            | Good (similar to gzip)        | Common for cross-platform exchange          |

-----

## 💽 4. Disk Usage Monitoring

Crucial for maintaining system health and preventing disk space issues.

| 💾 Command | 📌 Description                         | 🛠️ Use Case                                         | Analogy                                        |
| :--------- | :------------------------------------- | :-------------------------------------------------- | :--------------------------------------------- |
| `df`       | Reports disk space usage of filesystems | System monitoring, checking partition fullness, determining remaining space for new installs | Checking the fuel gauge of your car.           |
| `du`       | Estimates file space usage              | Identifying large files/directories for cleanup, optimizing storage | Weighing individual bags or boxes to see how heavy they are. |

-----

### 🧠 Real Usage:

**1. `df -h` - Disk Free (Filesystem level)**

```bash
df -h
# Expected output (simplified example - specific numbers will vary):
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda1        20G  8.0G   11G  42% /
# /dev/sdb1        50G   20G   29G  41% /home
# tmpfs           3.9G     0  3.9G   0% /dev/shm
```

**📟 Disk Partition Chart (`df -h` representation):**

```ascii
+-------------------------------------------------+
| Filesystem: /dev/sda1                           |
| Size: 20GB   Used: 8.0GB   Avail: 11GB   Use%: 42%|
| Mounted on: /                                   |
+-------------------------------------------------+
+-------------------------------------------------+
| Filesystem: /dev/sdb1                           |
| Size: 50GB   Used: 20GB   Avail: 29GB   Use%: 41%|
| Mounted on: /home                               |
+-------------------------------------------------+
```

**Highlight:** Shows usage for mounted filesystems (partitions), crucial for overall system health.

**2. `du -sh *` - Disk Usage (Directory/File level)**

**Scenario:** Identify large folders in your home directory (`/home/youruser`).

```bash
cd /home/youruser/
du -sh *
# Expected output (example):
# 1.2G	Downloads
# 4.5M	Documents
# 2.8G	my_project
# 4.0K	public_html
# 8.0K	.bashrc
# ...
```

**📁 Tree Map: `du` folder usage in `/home` (Conceptual Visual)**

Imagine a visual tool that would show:

  * `/home/youruser` (5.0 GB total)
      * `my_project/` (2.8 GB) **[Large Bar]**
      * `Downloads/` (1.2 GB) **[Medium Bar]**
      * `Documents/` (4.5 MB) **[Small Bar]**
      * `.` (Other small files)

**Most Used Flags (`du`):**

  * `-h`: Human-readable format (e.g., K, M, G).
  * `-s`: Summarize (display only a total for each argument).
  * `-c`: Grand total.
  * `--max-depth=N`: Limit depth of recursion.

### ✅ Safe Disk Cleanup Commands:

  * **Clear temporary files:**
    ```bash
    sudo apt clean # For Debian/Ubuntu - clears downloaded package archives
    sudo yum clean all # For CentOS/RHEL
    sudo rm -rf /tmp/* # Use with extreme caution, ensure no critical running services rely on /tmp data
    ```
  * **Remove old logs:**
    ```bash
    # Identify large log files (example for Nginx access logs)
    du -sh /var/log/nginx/* | sort -rh | head -n 10

    # Rotate logs (usually handled by logrotate, but manual example)
    # cp /var/log/nginx/access.log /var/log/nginx/access.log.old
    # echo "" > /var/log/nginx/access.log # Empty the log file
    # rm /var/log/nginx/access.log.old # Delete the old log (after review)
    ```
  * **Find and delete large, old files (use `find` with `-delete` carefully\!):**
    ```bash
    # Find files larger than 100MB in /var/log/ that haven't been accessed in 30 days and list them:
    find /var/log -type f -size +100M -atime +30 -print

    # ONLY IF YOU ARE 100% SURE, add -delete at the end:
    # find /var/log -type f -size +100M -atime +30 -delete
    ```

-----

## 🔐 Bonus: Interview Questions

| ❓ Question                               | ✅ Best Answer                                                                       |
| :---------------------------------------- | :----------------------------------------------------------------------------------- |
| What’s the difference between `find` and `locate`? | `find` performs a real-time, comprehensive search on the live filesystem based on various criteria; `locate` uses a pre-built, fast database index for filename-only searches. |
| When would you use `awk` vs `cut`?        | Use `cut` for simple, single-character delimited column extraction. Use `awk` for more complex parsing, filtering, and reporting with advanced pattern matching and logic. |
| How does `tar` differ from `zip`?        | `tar` is primarily an archiving utility (groups files), while `gzip` (often used with `tar`) is a compression utility. `zip` combines both archiving and compression into one tool, making it cross-platform compatible. |
| What does `df -h` tell you?              | `df -h` displays the total, used, and available disk space for all mounted file systems in a human-readable format, indicating overall partition usage. |
| How can you find large folders in Linux?  | Use `du -sh *` in a directory to get a summarized, human-readable size for each item. For recursive, detailed analysis, use `du -h --max-depth=1 /path/to/check | sort -rh`. |

-----

## 🧠 Tips, Tricks & Real Scenarios

  * **📦 Automate Backup with `tar` + `cron`**: Create daily compressed backups of critical data (`tar -czf /backups/data_$(date +%Y%m%d).tar.gz /var/www/html`) and schedule it with `cron`.
  * **🕵️ Live Log Monitoring**: Combine `grep` with `tail -f` (e.g., `tail -f /var/log/syslog | grep "ERROR"`) to watch for specific issues in real-time.
  * **🔁 Data Analysis Chain**: Use pipes (`|`) to create powerful data processing pipelines: `cat access.log | grep "404" | awk '{print $7}' | sort | uniq -c | sort -nr` (Find most common 404 URLs).
  * **📚 Set Aliases**: Add common complex commands as aliases in your `~/.bashrc` for efficiency, e.g., `alias searchbig='find . -type f -size +100M -print0 | xargs -0 du -h | sort -rh'`.
  * **⚠️ Avoid Compressing Active Logs**: Never directly compress an actively written log file (like `/var/log/messages`) without stopping or signaling the service. Use `logrotate` for proper log management.
