# 🔧 How `printf("Hello World")` Reaches Your Screen

> ✨ “Behind every line of code is a journey through the operating system.”

---

## 📌 What You'll Learn:

* What happens **under the hood** when you run `printf()`
* A full journey from **user code → system call → kernel → hardware**
* **Visual diagrams**, beginner-friendly terms, and Linux internals
* Includes **bonus: Assembly peek**, **Linux syscall table**, and more

---

## 🧠 The Code You Wrote

```c
#include <stdio.h>

int main() {
    printf("Hello World\n");  // Library function call
    return 0;
}
```

You just wrote a simple C program.

Let’s now **zoom into what happens step by step** when you run it:

---

## 🔍 Step-by-Step Journey

### 🔹 Step 1: `printf()` — The Library Call

* `printf()` is **NOT** a system call.
* It's part of the **C Standard Library (glibc)**.
* Its job:

  1. Format the string
  2. Call the **`write()`** function to pass output

```
printf("Hello World\n")
   ↓
write(fd=1, buffer="Hello World\n", count=12)
```

✅ Think of `printf()` as a helper that **prepares your data** and asks `write()` to do the real work.

---

### 🔹 Step 2: `write()` — System Call Wrapper

* `write()` is a **system call wrapper** provided by the C library.
* It prepares a **request to the OS kernel** using a syscall interface.

🎯 It doesn't directly talk to the hardware either — instead, it asks the kernel to handle it.

```c
ssize_t write(int fd, const void *buf, size_t count);
```

* `fd=1` → stdout (your terminal)
* `buf` → pointer to "Hello World\n"
* `count=12` → number of bytes

---

### 🔹 Step 3: `sys_write` — Actual System Call

At this point, a **transition happens**:

* Your code moves from **user mode → kernel mode**
* This is done via **`syscall` instruction** (x86\_64) or **`int 0x80`** (older systems)

```asm
mov eax, 1        ; syscall number for write
mov edi, 1        ; file descriptor (stdout)
mov rsi, msg      ; pointer to string
mov edx, 12       ; length
syscall           ; switch to kernel mode
```

🎯 The kernel has a syscall table.
Each syscall has a unique ID (write = 1 in Linux x86\_64).

---

### 🔹 Step 4: Kernel Handles It

* The kernel runs the **`sys_write()`** function.
* It:

  * Verifies permissions
  * Finds the terminal device
  * Passes the buffer to the **terminal driver**

✅ Now it's in kernel mode and has full access to hardware.

---

### 🔹 Step 5: Device Driver → Hardware

* The **terminal driver** knows how to talk to your display hardware.
* It sends the characters one-by-one to the **graphics output** (or emulator like GNOME Terminal).

---

### ✅ Final Result: Output on Screen

```
Hello World
```

Your monitor shows the text — but only after this deep chain of events!

---

## 🔄 Visualization: Full Journey Flow

```plaintext
🧑 User Space
┌──────────────────────────────────────────┐
│  main()                                  │
│    ↓                                     │
│  printf("Hello World\n")                 │
│    ↓ formats + calls → write()          │
│      ↓ system call wrapper              │
└──────────────────────────────────────────┘

🔧 Kernel Space
┌──────────────────────────────────────────┐
│  syscall(sys_write)                      │
│    ↓                                     │
│  Find terminal driver                    │
│    ↓                                     │
│  Pass buffer to device                   │
└──────────────────────────────────────────┘

💻 Hardware Layer
┌──────────────────────────────────────────┐
│  Display Controller / Terminal Emulator  │
│    ↓                                     │
│  Text appears on your screen             │
└──────────────────────────────────────────┘
```

---

## 📚 Kernel Syscall Table (Linux x86\_64)

| Syscall | Name    | Description            |
| ------- | ------- | ---------------------- |
| 0       | `read`  | Read from file         |
| **1**   | `write` | Write to file/terminal |
| 60      | `exit`  | Terminate process      |

See more in: `/usr/include/asm/unistd_64.h`

---

## ⚙️ Bonus: How `syscall` Works Internally

```asm
; Registers used in syscall on x86_64
rax — syscall number  
rdi — first argument (fd)  
rsi — second argument (buffer)  
rdx — third argument (count)
```

The `syscall` instruction causes a **CPU trap**, moving from **ring 3 → ring 0** (user → kernel).

---

## 🧠 Easy Way to Remember

| Layer       | Action                             | Example                     |
| ----------- | ---------------------------------- | --------------------------- |
| User Space  | You write and run `printf()`       | C code                      |
| Library     | Formats and delegates to `write()` | glibc                       |
| Syscall API | System call wrapper                | `write(fd, buf, count)`     |
| Kernel      | Handles with `sys_write()`         | Protected OS space          |
| Driver      | Knows how to talk to hardware      | Terminal driver             |
| Hardware    | Shows output                       | Monitor / Terminal Emulator |

---

## 📦 Summary Table

| Step        | Layer       | Description                      |
| ----------- | ----------- | -------------------------------- |
| `main()`    | User Space  | Your C code starts execution     |
| `printf()`  | C Library   | Formats and sends to `write()`   |
| `write()`   | Syscall API | Calls `sys_write` in the kernel  |
| `sys_write` | Kernel      | Executes logic to pass to device |
| Driver      | Kernel      | Transfers buffer to terminal     |
| Output      | Hardware    | Text appears on screen           |

---

## 📘 Real-World Tip

Want to **see syscalls** made by your program?

```bash
strace ./your_program
```

You’ll see:

```bash
write(1, "Hello World\n", 12) = 12
```

---

## 📎 Want to Go Deeper?

* [Linux Kernel Source - sys\_write](https://elixir.bootlin.com/linux/latest/source/fs/read_write.c)
* [x86\_64 Syscall ABI](https://wiki.osdev.org/Calling_Conventions#System_V_AMD64_ABI)
* [glibc printf implementation](https://code.woboq.org/userspace/glibc/stdio-common/printf.c.html)

---

## 🏁 Final Thoughts

Even a simple `printf()` command goes through a beautiful, complex journey involving:

* User code
* Libraries
* System calls
* Kernel internals
* Device drivers
* Physical hardware